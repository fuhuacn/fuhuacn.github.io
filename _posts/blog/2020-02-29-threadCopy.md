---
layout: post
title: 多线程复制代码记录
categories: Blogs
description: 多线程复制代码记录
keywords: java,多线程复制
---

``` java
package threadCopy;

import java.io.File;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadCopy {
    static CountDownLatch countDownLatch;
    static int threadNum = 300;
    static String fileName = "/Users/fuhua/Documents/自动监测错误系统/代码要用的内容/2.csv";
    static String destFileName = "/Users/fuhua/Desktop/copy.csv";

    public static void main(String[] args) throws InterruptedException {
        long startTime = System.currentTimeMillis();
        File file = new File(fileName);
        long length = file.length();
        if(length/threadNum>Integer.MAX_VALUE){
            System.out.println("线程数量太小，超限度");
            return;
        }
        int eachLength = (int)length/threadNum;
        int res = (int)length%threadNum;
        if (res>0){
            countDownLatch = new CountDownLatch(threadNum+1);
        }else {
            countDownLatch = new CountDownLatch(threadNum);
        }
        ExecutorService es = Executors.newFixedThreadPool(threadNum);
        for (int i=0;i<threadNum;i++){
            es.execute(new CopyByThread(fileName,destFileName,i*eachLength,(i+1)*eachLength));
        }
        es.shutdown();
        CopyByThread copyByThread = new CopyByThread(fileName,destFileName,threadNum*eachLength,threadNum*eachLength+res);
        new Thread(copyByThread).start();
        countDownLatch.await();
        System.out.println("耗时 "+(System.currentTimeMillis()-startTime));

    }
    public static class CopyByThread implements Runnable{

        long startPosition;
        long endPosition;
        String srcFileName;
        String destFileName;

        public CopyByThread(String srcFileName, String destFileName, long startPosition,long endPosition){
            this.startPosition = startPosition;
            this.endPosition = endPosition;
            this.srcFileName = srcFileName;
            this.destFileName = destFileName;
        }

        public void run(){
            try{
                RandomAccessFile src = new RandomAccessFile(srcFileName,"rw");
                RandomAccessFile dest = new RandomAccessFile(destFileName,"rw");
                FileChannel srcC = src.getChannel(); // 0-4 4-8 8-12每次读4个字节这样读
                FileChannel destC = dest.getChannel();
                ByteBuffer byteBuffer = ByteBuffer.allocateDirect((int)(endPosition-startPosition));
                srcC.position(startPosition);
                destC.position(startPosition);
                srcC.read(byteBuffer);
                byteBuffer.flip();
                destC.write(byteBuffer);
                countDownLatch.countDown();
            }catch (Exception e){

            }
            System.out.println(Thread.currentThread()+" 已完成 "+startPosition+" "+endPosition);
        }
    }
}
```

453.9MB csv 文件，当 threadNum 设为 1 的情况下（需要简单修改 countDownLatch 配置）速度大概 1400ms。

当 threadNum 设为 10 的情况下（需要简单修改 countDownLatch 配置）速度大概 1100ms。

当 threadNum 设为 300 的情况下（需要简单修改 countDownLatch 配置）速度大概 450ms。