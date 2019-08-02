---
title: JDK源码阅读系列(五)--StringBuilder类
date: 2017-10-22 12:23:39
tags: [Java, 源码阅读]
---

1、概况

​	在 Java 中处理字符串时经常会使用 String 类，实际上 String 对象的值是一个常量，一旦创建后不能被改变。正是因为其不可变，所以也无法进行修改操作，只有不断地 new 出新的 String 对象。为此 Java 引入了可变字符串变量 StringBuilder 类，它不是线程安全的，只用在单线程场景下。类关系如下：

![class_sb_1](D:/Study/Blog/Hexo/source/_posts/JDK%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E7%B3%BB%E5%88%97-%E4%BA%94-StringBuilder%E7%B1%BB/class_sb_1.png)

2、继承关系及类定义

```java
--java.lang.Object
  --java.lang.AbstractStringBuilder
    --java.lang.StringBuilder
```

```java
public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, CharSequence
```

​	StringBuilder 类被声明为 final，说明它不能再被继承。同时它继承了 AbstractStringBuilder 类，并实现了 Serializable 和 CharSequence 两个接口。

其中 Serializable 接口表明其可以序列化。

CharSequence 接口用来实现获取字符序列的相关信息，接口定义如下： 

- length()获取字符序列长度。 
- charAt(int index)获取某个索引对应字符。 
- subSequence(int start, int end)获取指定范围子字符串。 
- toString()转成字符串对象。 
- chars()用于获取字符序列的字符的 int 类型值的流，该接口提供了默认的实现。 
- codePoints()用于获取字符序列的代码点的 int 类型的值的流，提供了默认的实现。

```java
public interface CharSequence {

    int length();

    char charAt(int index);

    CharSequence subSequence(int start, int end);

    public String toString();

    public default IntStream chars() {
        省略代码。。
    }

    public default IntStream codePoints() {
        省略代码。。
    }
}
```

