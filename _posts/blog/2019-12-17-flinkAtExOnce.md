---
layout: post
title: Flink 消费 Kafka 的 exactly-once
categories: Blogs
description: Flink 消费 Kafka 的 exactly-once
keywords: kafka,flink
---

由于 ETL 任务需要，也就是迁移过程中不能多一条或者少一条，必须要实现消费 Kafka 时的精准一次消费，网上找了很多资料，没有一个写的非常详细的，在这里分享一下整体的流程。

# Exactly-once VS At-least-once

算子做快照时，如果等所有输入端的 barrier 都到了才开始做快照，那么就可以保证算子的 exactly-once；如果为了降低延时而跳过对其，从而继续处理数据，那么等 barrier 都到齐后做快照就是 at-least-once 了，因为这次的快照掺杂了下一次快照的数据，当作业失败恢复的时候，这些数据会重复作用系统，就好像这些数据被消费了两遍。

*注：对齐只会发生在算子的上端是 join 操作以及上游存在 partition 或者 shuffle 的情况，对于直连操作类似 map、flatMap、filter 等还是会保证 exactly-once 的语义。*

# 原理

原理这里就简述一下，详细的介绍可以参考[这里](http://www.54tianzhisheng.cn/2019/06/20/flink-kafka-Exactly-Once/)。

## Flink Checkpoint

想要实现精准一次必须开启 Flink 的 CheckPoint，CheckPoint 是 Flink 的全局快照，当系统崩溃时，下一次在启动系统时会从此快照恢复。放到 Flink + Kafka 中可以理解为每个 CheckPoint 恢复到上一次已经消费的 Offset。

## Flink 中的两阶段提交

目的是为方便分布式中的 Exactly-Once 等实现。

通过定义四个接口方法可以完成不同要求等级的操作。

对于 Exactly-Once 大致的思想就是所有的 Message 都会先预提交一次，直到确认所有都正常预提交后，在正式提交。基本流程（这里直接拿了TwoPhaseCommitSinkFunction 的接口方法）：

+ invoke - 在事务中提交一条数据。
+ beginTransaction - 在事务开始前，我们在目标文件系统的临时目录中创建一个临时文件。随后，我们可以在处理数据时将数据写入此文件。
+ preCommit - 老的事务到此阶段，老得事务需要被 jobManager 检测是否提交。在预提交阶段，我们刷新之前的文件到存储，关闭文件，不再重新写入。我们还将为属于下一个（也就是这个新的） checkpoint 的任何后续文件写入启动一个新的事务（所以说他处在老事务准备提交时，新的事物开启之前）。一般做 flush。
+ commit - 在提交阶段，我们将预提交阶段的文件原子地移动到真正的目标目录。需要注意的是，这会增加输出数据可见性的延迟。
+ abort - 在中止阶段，我们删除临时文件。

*注意：以上是通用的一个模版，但我们知道 Kafka 是具有事务机制的，利用 Kafka 的事务机制可以大大减少这一步骤，所以对于 FlinkKafkaProducer 其实是这样做的（实现了 TwoPhaseCommitSinkFunction）：*

+ invoke：发送一条为提交数据
  ``` java
  @Override
	public void invoke(FlinkKafkaProducer.KafkaTransactionState transaction, IN next, Context context) throws FlinkKafkaException {
		checkErroneous();

		ProducerRecord<byte[], byte[]> record;
		if (keyedSchema != null) {
			byte[] serializedKey = keyedSchema.serializeKey(next);
			byte[] serializedValue = keyedSchema.serializeValue(next);
			String targetTopic = keyedSchema.getTargetTopic(next);
			if (targetTopic == null) {
				targetTopic = defaultTopicId;
			}

			Long timestamp = null;
			if (this.writeTimestampToKafka) {
				timestamp = context.timestamp();
			}

			int[] partitions = topicPartitionsMap.get(targetTopic);
			if (null == partitions) {
				partitions = getPartitionsByTopic(targetTopic, transaction.producer);
				topicPartitionsMap.put(targetTopic, partitions);
			}
			if (flinkKafkaPartitioner != null) {
				record = new ProducerRecord<>(
						targetTopic,
						flinkKafkaPartitioner.partition(next, serializedKey, serializedValue, targetTopic, partitions),
						timestamp,
						serializedKey,
						serializedValue);
			} else {
				record = new ProducerRecord<>(targetTopic, null, timestamp, serializedKey, serializedValue);
			}
		} else if (kafkaSchema != null) {
			if (kafkaSchema instanceof KafkaContextAware) {
				@SuppressWarnings("unchecked")
				KafkaContextAware<IN> contextAwareSchema =
						(KafkaContextAware<IN>) kafkaSchema;

				String targetTopic = contextAwareSchema.getTargetTopic(next);
				if (targetTopic == null) {
					targetTopic = defaultTopicId;
				}
				int[] partitions = topicPartitionsMap.get(targetTopic);

				if (null == partitions) {
					partitions = getPartitionsByTopic(targetTopic, transaction.producer);
					topicPartitionsMap.put(targetTopic, partitions);
				}

				contextAwareSchema.setPartitions(partitions);
			}
			record = kafkaSchema.serialize(next, context.timestamp());
		} else {
			throw new RuntimeException(
					"We have neither KafkaSerializationSchema nor KeyedSerializationSchema, this" +
							"is a bug.");
		}

		pendingRecords.incrementAndGet();
		transaction.producer.send(record, callback);
	}
  ```
+ beginTransaction：开启 Kafka 事务
  ``` java
  case EXACTLY_ONCE:
				FlinkKafkaInternalProducer<byte[], byte[]> producer = createTransactionalProducer();
				producer.beginTransaction();
				return new FlinkKafkaProducer.KafkaTransactionState(producer.getTransactionalId(), 
  ```
+ preCommit：因为借助了事务，没有操作，但可以看见 AT_LEAST_ONCE 做了 flush，这样至少都能之前那次都能提交了。
  ``` java
  switch (semantic) {
			case EXACTLY_ONCE:
			case AT_LEAST_ONCE:
				flush(transaction);
				break;
			case NONE:
				break;
			default:
				throw new UnsupportedOperationException("Not implemented semantic");
  ```
+ commit：真正的提交了 Message。
  ``` java
  protected void commit(FlinkKafkaProducer.KafkaTransactionState transaction) {
		if (transaction.isTransactional()) {
			try {
				transaction.producer.commitTransaction();
			} finally {
				recycleTransactionalProducer(transaction.producer);
			}
		}
	}
  ```
+ abort：舍弃了事务，如果一个出现问题，做 abort 这样就相当于回滚了。
  ``` java
  public void abortTransaction() throws ProducerFencedException {
        this.throwIfNoTransactionManager();
        TransactionalRequestResult result = this.transactionManager.beginAbort();
        this.sender.wakeup();
        result.await();
    }
  ```

## Kafka 事务和 CheckPoint 的配合逻辑

Kafka 是有事务的，也都分析了。具体调用的时候是这样的：

[barrier](https://juejin.im/post/5bf93517f265da611510760d) 是用来分隔每个 Checkpoint 阶段的标记。比如说 Checkpoint 设定的间隔是 1 分钟，则每分钟会往流中发送 barrier 以标记数据的不同 Checkpoint 的段以区分进入当前快照和下一个快照。

Checkpoint 由 JobManager 中的 CheckpointCoordinator 负责管理，注入 barrier 后，barrier 每经过一个 operator 会往 backend 做快照存储（证明这个 checkpoints 的全部已经在此 operator 做完了），如下：

![存储](/images/posts/blog/flinkAtExactlyOnce/2019-07-07-094742.jpg)

当通过 sink 后会做直接做预提交，这样之后的数据都会往下一次的 checkpoint 中写。

当检查 checkpoint 完成后，就可以通知所有各机器 sink operator 提交了。

![完毕](/images/posts/blog/flinkAtExactlyOnce/2019-07-07-094810.jpg)

## 代码

``` java
package org.flinkETL;

import org.apache.avro.Schema;
import org.apache.avro.generic.GenericRecord;
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.formats.avro.registry.confluent.ConfluentRegistryAvroDeserializationSchema;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.AllWindowFunction;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;
import org.apache.flink.streaming.connectors.kafka.KafkaSerializationSchema;
import org.apache.flink.util.Collector;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.flinkETL.bean.People;
import java.util.Properties;

public class StreamingJob {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(5);
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        env.enableCheckpointing(60000);
//        env.getCheckpointConfig().setCheckpointInterval(60000);

        env.setStateBackend(new FsStateBackend("hdfs://ambari-namenode.com:8020/flinkBackEnd/"));
        Properties props = new Properties();
        props.put("bootstrap.servers", "192.168.34.60:9092,192.168.34.62:9092,192.168.34.57:9092,192.168.34.79:9092,192.168.34.80:9092");
        props.put("group.id", "flinkTest");
        props.put("auto.offset.reset", "latest");

        String asvc = "{\n" +
        "  \"name\": \"Message\",\n" +
        "  \"namespace\": \"com.free4lab.databus.collector.model\",\n" +
        "  \"type\": \"record\",\n" +
        "  \"fields\": [\n" +
        "    {\n" +
        "      \"name\": \"id\",\n" +
        "      \"type\": \"int\"\n" +
        "    },\n" +
        "    {\n" +
        "      \"name\": \"name\",\n" +
        "      \"type\": [\"string\", \"null\"]\n" +
        "    },\n" +
        "    {\n" +
        "      \"name\": \"identifyNumber\",\n" +
        "      \"type\": [\"string\", \"null\"]\n" +
        "    }\n" +
        "  ]\n" +
        "}";
        FlinkKafkaConsumer<GenericRecord> myConsumer = new FlinkKafkaConsumer(
                "flink-identify",
                ConfluentRegistryAvroDeserializationSchema.forGeneric(new Schema.Parser().parse(asvc), "http://192.168.34.62:8081"),
                props);
        myConsumer.setCommitOffsetsOnCheckpoints(true);
        DataStream<GenericRecord> stream = env.addSource(myConsumer).setParallelism(5);
        DataStream<String> res = stream.map((MapFunction<GenericRecord, People>) genericRecord -> {
            People people = new People();
            people.setId((Integer) genericRecord.get("id"));
            people.setName(String.valueOf(genericRecord.get("name")));
            people.setIdentifyNumber(String.valueOf(genericRecord.get("identifyNumber")));
            return people;
        }).filter(new FilterFunction<People>() {
            @Override
            public boolean filter(People people) throws Exception {
                if (people.getIdentifyNumber().length()!=18) return false;
                else return true;
            }
        }).timeWindowAll(Time.seconds(20)).apply(new AllWindowFunction<People, String, TimeWindow>() {
            @Override
            public void apply(TimeWindow window, Iterable<People> values, Collector<String> out) throws Exception {
                values.forEach(l->{
                    out.collect(l.getIdentifyNumber());
                });
            }
        });
        res.print();
        Properties sendProp = new Properties();
        sendProp.put("bootstrap.servers","192.168.34.60:9092,192.168.34.62:9092,192.168.34.57:9092");
        sendProp.put("transaction.timeout.ms","600000");//详见问题 1

        res.addSink(new FlinkKafkaProducer<String>("suibianfade", (KafkaSerializationSchema<String>) (element, timestamp) -> new ProducerRecord<byte[], byte[]>("suibianfade",timestamp.toString().getBytes(),element.getBytes()), sendProp, FlinkKafkaProducer.Semantic.EXACTLY_ONCE));

        env.execute("Flink Streaming Java API Skeleton");
    }
}

//    FlinkKafkaConsumer<GenericRecord> myConsumer = new FlinkKafkaConsumer(
//            "flink-identify",
//            new KafkaGenericAvroDeserializationSchema("http://192.168.34.62:8081","flink-identify"),
//            props);
```

## 遇到的问题

+ Kafka 报事务超时

  Kafka broker 将 transaction.max.timeout.ms 设定为 15 分钟，而 Flink 设定为 1 小时。一般事务超时时间肯定也不能像 15 分钟这么小，所以得改一下。

+ 消费事务要设定只查看提交内容

  isolation.level=read_committed，这个坑了我好久，默认是没提交的也能看到的（read_uncommited）。如果使用控制台的 consumer，注意 property 会被覆盖。

## 效果

``` text
发送完在 20 秒窗口结束直接能看到：

1>  23:44:40 CST 2019
5>  23:44:40 CST 2019
4>  23:44:40 CST 2019

但在 consumer 端没有，直到控制台提示 checkpoint 1 分钟更新：


23:45:38,191 INFO  org.apache.flink.streaming.api.functions.sink.TwoPhaseCommitSinkFunction  - FlinkKafkaProducer 1/5 - checkpoint 68 complete, committing transaction TransactionHolder{handle=KafkaTransactionState [transactionalId=Sink: Unnamed-ab2068ebf6ea6694da77a7abef5d7955-5, producerId=43006, epoch=55], transactionStartTime=1576597465168} from checkpoint 68
23:45:38,191 INFO  org.apache.flink.streaming.api.functions.sink.TwoPhaseCommitSinkFunction  - FlinkKafkaProducer 0/5 - checkpoint 68 complete, committing transaction TransactionHolder{handle=KafkaTransactionState [transactionalId=Sink: Unnamed-ab2068ebf6ea6694da77a7abef5d7955-4, producerId=42001, epoch=53], transactionStartTime=1576597465337} from checkpoint 68
23:45:38,191 INFO  org.apache.flink.streaming.api.functions.sink.TwoPhaseCommitSinkFunction  - FlinkKafkaProducer 2/5 - checkpoint 68 complete, committing transaction TransactionHolder{handle=KafkaTransactionState [transactionalId=Sink: Unnamed-ab2068ebf6ea6694da77a7abef5d7955-10, producerId=44024, epoch=51], transactionStartTime=1576597460589} from checkpoint 68
23:45:38,191 INFO  org.apache.flink.streaming.api.functions.sink.TwoPhaseCommitSinkFunction  - FlinkKafkaProducer 4/5 - checkpoint 68 complete, committing transaction TransactionHolder{handle=KafkaTransactionState [transactionalId=Sink: Unnamed-ab2068ebf6ea6694da77a7abef5d7955-20, producerId=47001, epoch=55], transactionStartTime=1576597465170} from checkpoint 68
23:45:38,191 INFO  org.apache.flink.streaming.api.functions.sink.TwoPhaseCommitSinkFunction  - FlinkKafkaProducer 3/5 - checkpoint 68 complete, committing transaction TransactionHolder{handle=KafkaTransactionState [transactionalId=Sink: Unnamed-ab2068ebf6ea6694da77a7abef5d7955-17, producerId=45015, epoch=54], transactionStartTime=1576597460589} from checkpoint 68

之后在 consumer 可以看到：

 23:44:40 CST 2019
 23:44:40 CST 2019
 23:44:40 CST 2019
```
