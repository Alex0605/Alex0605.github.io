---
title: JDK源码阅读系列(三)--AbstractStringBuilder类
date: 2017-10-20 21:50:11
tags: [Java, 源码阅读]
---

​	因为看StringBuffer 和 StringBuilder 的源码时发现两者都继承了AbstractStringBuilder，并且很多方法都是直接super的父类AbstractStringBuilder的方法，所以还是决定先看AbstractStringBuilder的源码，然后再看StringBuffer 和 StringBuilder.

#### 1.成员变量

 	AbstractStringBuilder和String一样，在其内部都是以字符数组的形式实现的。也就是String,StringBuffer以及StringBuilder在其内部都是以字符数组的形式实现的。

```java
char value[];
int count;
```

#### 2.构造函数

​	AbstractStringBuilder的构造函数中传入的capacity是指容量，实际长度是以leng中的count表示的，注意区分容量和实际长度

```java
AbstractStringBuilder() {
 }
 
 AbstractStringBuilder(int capacity) {
     value = new char[capacity];
 }
```

#### 3.容量和长度

```java
public int length() {
	return count;
}
public int capacity() {
	return value.length;
}
```

#### 4.AbstractStringBuilder的扩容

```java
public void ensureCapacity(int minimumCapacity) {
	if (minimumCapacity > value.length) {//如果传入的容量大于原来的容量就扩容
	    expandCapacity(minimumCapacity);
	}
    }
 
 void expandCapacity(int minimumCapacity) {
	int newCapacity = (value.length + 1) * 2;//首先默认扩容为  （原容量+1）*2
        if (newCapacity < 0) {
            newCapacity = Integer.MAX_VALUE;
        } else if (minimumCapacity > newCapacity) {//如果传入的容量大于 默认扩容量，则传入容量为新容量
	    newCapacity = minimumCapacity;
	}
        value = Arrays.copyOf(value, newCapacity);
    }

```

#### 5.字符串减少存储空间

如果实际长度小于容量，为了减少存储空间，就把容量缩小为刚好满足字符串长度。

```java
public void trimToSize() {
        if (count < value.length) {
            value = Arrays.copyOf(value, count);
        }
    }
```

#### 6.setLength(int newLength)

```java
public void setLength(int newLength) {
	if (newLength < 0)
	    throw new StringIndexOutOfBoundsException(newLength);
	if (newLength > value.length)//设置长度大于容量时先扩容
	    expandCapacity(newLength);
if (count < newLength) {//设置长度大于原来长度时，后面部分补以空白
    for (; count < newLength; count++)
	value[count] = '\0';
} else {
        count = newLength;
    }
}
```
#### 7.char charAt(int index)

```java
public char charAt(int index) {
	if ((index < 0) || (index >= count))
	    throw new StringIndexOutOfBoundsException(index);
	return value[index];
    }
```

#### 8.void getChars(int srcBegin, int srcEnd, char dst[],  int dstBegin)

getChars() 方法将字符从字符串复制到目标字符数组。

参数

- srcBegin -- 字符串中要复制的第一个字符的索引。
- srcEnd -- 字符串中要复制的最后一个字符之后的索引。
- dst -- 目标数组。
- dstBegin -- 目标数组中的起始偏移量。

返回值

没有返回值，但会抛出 IndexOutOfBoundsException 异常。

一定要注意参数情况的考虑，自己编程时一定要养成好习惯。主要是调用了System.arraycopy()的方法，以后具体分析。

源码：

```java
 public void getChars(int srcBegin, int srcEnd, char dst[],
                                      int dstBegin)
    {
	if (srcBegin < 0)
	    throw new StringIndexOutOfBoundsException(srcBegin);
	if ((srcEnd < 0) || (srcEnd > count))
	    throw new StringIndexOutOfBoundsException(srcEnd);
        if (srcBegin > srcEnd)
            throw new StringIndexOutOfBoundsException("srcBegin > srcEnd");
	System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
    }
```

实例：

```java
public class Test {
    public static void main(String args[]) {
        String Str1 = new String("www.baidu.com");
        char[] Str2 = new char[5];

        try {
            Str1.getChars(4, 9, Str2, 0);
            System.out.print("拷贝的字符串为：" );
            System.out.println(Str2 );
        } catch( Exception ex) {
            System.out.println("触发异常...");
        }
    }
}
```

以上程序执行结果为：

拷贝的字符串为：runoob

#### 9.void setCharAt(int index, char ch)

方法的源码很简单，是直接替换该index的值，而不是后移之类的，因为是数组，不可变长度。

```java
 public void setCharAt(int index, char ch) {
	if ((index < 0) || (index >= count))
	    throw new StringIndexOutOfBoundsException(index);
	value[index] = ch;
    }
```

#### 10.append(String str)

对于StringBuffer和StringBuilder重要的append()方法的源码闪亮登场咯~~

```java
public AbstractStringBuilder append(String str) {
	if (str == null) str = "null";//若传入为null,则会在后面加上“null”
        int len = str.length();
	if (len == 0) return this;//传入长度为0，则返回本身
	int newCount = count + len;
	if (newCount > value.length)//传入后的长度大于容量就扩容
	    expandCapacity(newCount);
	str.getChars(0, len, value, count);//用的是getChars()为value传值
	count = newCount;
	return this;
    }
```

#### 11.append(CharSequence s, int start, int end)

```java
 public AbstractStringBuilder append(CharSequence s, int start, int end) {
        if (s == null)
            s = "null";
	if ((start < 0) || (end < 0) || (start > end) || (end > s.length()))//一定要注意情况考虑
	    throw new IndexOutOfBoundsException(
                "start " + start + ", end " + end + ", s.length() " 
                + s.length());
	int len = end - start;
	if (len == 0)
            return this;
	int newCount = count + len;
	if (newCount > value.length)
	    expandCapacity(newCount);
        for (int i=start; i<end; i++)//用的是charAt()为value循环赋值
            value[count++] = s.charAt(i);//!!
        count = newCount;
	return this;
    }
```

#### 12.append(char str[])

若是字符数组，用的是System.arraycopy（）方法。

#### 12.append(char str[])

若是字符数组，用的是System.arraycopy（）方法。

```java
 public AbstractStringBuilder append(char str[]) { 
	int newCount = count + str.length;
	if (newCount > value.length)
	    expandCapacity(newCount);
        System.arraycopy(str, 0, value, count, str.length);
        count = newCount;
        return this;
    }
```

#### 13.append(boolean b)

这个源码很简单

```java
 public AbstractStringBuilder append(boolean b) {
        if (b) {
            int newCount = count + 4;
            if (newCount > value.length)
                expandCapacity(newCount);
            value[count++] = 't';
            value[count++] = 'r';
            value[count++] = 'u';
            value[count++] = 'e';
        } else {
            int newCount = count + 5;
            if (newCount > value.length)
                expandCapacity(newCount);
            value[count++] = 'f';
            value[count++] = 'a';
            value[count++] = 'l';
            value[count++] = 's';
            value[count++] = 'e';
        }
	return this;
    }
```

#### 14.append(int i)

```java
public AbstractStringBuilder append(int i) {
        if (i == Integer.MIN_VALUE) {
            append("-2147483648");
            return this;
        }
        int appendedLength = (i < 0) ? stringSizeOfInt(-i) + 1 : stringSizeOfInt(i);//!!!
        int spaceNeeded = count + appendedLength;
        if (spaceNeeded > value.length)
            expandCapacity(spaceNeeded);
	Integer.getChars(i, spaceNeeded, value);
        count = spaceNeeded;
        return this;
    }
    final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                     99999999, 999999999, Integer.MAX_VALUE };
// Requires positive x
static int stringSizeOfInt(int x) {
    for (int i=0; ; i++)
        if (x <= sizeTable[i])
            return i+1;
}
```
####  15.delete(int start, int end)

主要还是用的System.arraycopy()

```
public AbstractStringBuilder delete(int start, int end) {
	if (start < 0)
	    throw new StringIndexOutOfBoundsException(start);
	if (end > count)
	    end = count;
	if (start > end)
	    throw new StringIndexOutOfBoundsException();
        int len = end - start;
        if (len > 0) {
            System.arraycopy(value, start+len, value, start, count-end);
            count -= len;
        }
        return this;
    }
```

#### 16.reverse()

这个是StringBuffer和StringBuilder常用到的方法，而String并没有这个牛逼的功能~~

```java
public AbstractStringBuilder reverse() {
	boolean hasSurrogate = false;
	int n = count - 1;
	for (int j = (n-1) >> 1; j >= 0; --j) {
	    char temp = value[j];
	    char temp2 = value[n - j];
	    if (!hasSurrogate) {
		hasSurrogate = (temp >= Character.MIN_SURROGATE && temp <= Character.MAX_SURROGATE)
		    || (temp2 >= Character.MIN_SURROGATE && temp2 <= Character.MAX_SURROGATE);
	    }
	    value[j] = temp2;
	    value[n - j] = temp;
	}
	if (hasSurrogate) {
	    // Reverse back all valid surrogate pairs
	    for (int i = 0; i < count - 1; i++) {
		char c2 = value[i];
		if (Character.isLowSurrogate(c2)) {
		    char c1 = value[i + 1];
		    if (Character.isHighSurrogate(c1)) {
			value[i++] = c1;
			value[i] = c2;
		    }
		}
	    }
	}
	return this;
    }
```

#### 17.String toString()

这个只是一个抽象的方法，没有方法体。

```java
public abstract String toString();
```
