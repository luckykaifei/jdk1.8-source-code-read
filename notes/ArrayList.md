---
layout: post
title: "ArrayList源码总结和扩容机制"
date: 2019-01-28 13:26
comments: true
tags:
     - jdk1.8源码
---
## ArrayList源码总结

1：三个不同的构造方法。<font color=red size=2 face="微软雅黑">**无参构造方法构造的ArrayList的容量默认为10**</font>，带有Collection参数的构造方法，将Collection转化为数组赋给ArrayList的实现数组elementData。 

2：<font color=red size=2 face="微软雅黑">**扩容的时候，ArrayList在每次增加一个元素的时候，都要调用ensureCapacity来确保足够的容量，当容量不够时，设置新的容量为原来容量的1.5倍（一般来说到10.5倍我们遇到的扩容都是足够的），然后用Arrays.copy()方法将元素拷贝到新的数组，但是这样耗时，所以在事先确定了元素的数量的情况下，用ArrayList，否则用LinkedList**</font>

3：Arrays.copyof()和System.copyof()
```   
public static <T> T[] copyOf(T[] original, int newLength) {
        //调用两一个copyof()方法
        return (T[]) copyOf(original, newLength, original.getClass());
    }
```
```
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }

```
<font color=red size=2 face="微软雅黑">**该方法实际上是在其内部又创建了一个长度为newlength的数组，调用System.arraycopy()方法，将原来数组中的元素复制到了新的数组中。**</font>

System.arraycopy()方法。该方法被标记了native，调用了系统的C/C++代码，在JDK中是看不到的，但在openJDK中可以看到其源码。该函数实际上最终调用了C语言的memmove()函数，因此它可以保证同一个数组内元素的正确复制和移动，比一般的复制方法的实现效率要高很多，很适合用来批量处理数组。<font color=red size=2 face="微软雅黑">**Java强烈推荐在复制大量数组元素时用该方法，以取得更高的效率。**</font>

## ArrayList扩容机制
1:调用```add```方法
```
 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

2：调用 ```ensureCapacityInternal``` 方法
```
private void ensureCapacityInternal(int minCapacity) {
        // 判空
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
```

3：调用 ```ensureExplicitCapacity``` 方法
```
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 当数组存不下新元素时，对数组进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

4：调用 ```grow``` 方法
```
 private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 数组容量被扩大为原来的 1.5 倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // MAX_ARRAY_SIZE = 2<sup>31</sup>-1-8
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

5：调用 ```Arrays.copyOf``` 方法
```
 public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```