---
layout: post
title: throw，throws
categories: Java
description: throw，throws
keywords: JAVA,throw
---

## 抛出异常

抛出异常有三种形式，一是 throw，一个 throws，还有一种系统自动抛异常。下面它们之间的异同。

## 系统自动抛异常

当程序语句出现一些逻辑错误、主义错误或类型转换错误时，系统会自动抛出异常。如：

``` java

public static void main(String[] args) {
		int a = 5, b =0;
		System.out.println(5/b);
		//function();
}
```

系统会自动抛出ArithmeticException异常。

## throw

throw 是语句抛出一个异常。一般会用于程序出现某种逻辑时程序员主动抛出某种特定类型的异常。如：

``` java
public static void main(String[] args) {
		String s = "abc";
		if(s.equals("abc")) {
			throw new NumberFormatException();
		} else {
			System.out.println(s);
		}
		//function();
}
```

会抛出异常：

``` java
Exception in thread "main" java.lang.NumberFormatException

at test.ExceptionTest.main(ExceptionTest.java:67)
```

## throws

throws是方法*可能*抛出异常的声明。(用在声明方法时，表示该方法可能要抛出异常)
语法：[(修饰符)](返回值类型)(方法名)([参数列表])[throws(异常类)]{......}
      如：      public void function() throws Exception{......}

当某个方法可能会抛出某种异常时用于throws 声明可能抛出的异常，然后交给**上层调用它的方法程序处理**。如：

``` java

public static void function() throws NumberFormatException{
		String s = "abc";
		System.out.println(Double.parseDouble(s));
	}
	
	public static void main(String[] args) {
		try {
			function();
		} catch (NumberFormatException e) {
			System.err.println("非数据类型不能转换。");
			//e.printStackTrace();
		}
}
```

function() 必须用 try catch 处理或者再次在方法中 throws 出去。**但如果 throws 到最顶层调用者（即 JVM）程序就会报错停止。**