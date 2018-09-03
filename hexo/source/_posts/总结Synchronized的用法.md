---
title: 总结Synchronized的用法
date: 2018-05-22 11:16:16
tags: [Java, synchronized]
---

synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种： 

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 

2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 

3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 

4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

本文从使用技巧来总结synchronized的常用方法

<!-- more -->

### 实现支持高并发且线程安全的单例

```java
public class MySingleton {
    private volatile static MySingleton instance = null;

    private MySingleton(){}

    public static MySingleton getInstance() {
        if (instance == null) {
            synchronized (MySingleton.class) {
                if (instance == null) {
                    instance = new MySingleton();
                }
            }
        }
        return instance;
    }
}
```

这里有一个很关键的修饰词volatile，如果没有volatile，这个单例在多线程会如何表现，我们可以来测试一下：

```
public class MyThread extends Thread{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ":" + MySingleton.getInstance().hashCode());
    }

    public static void main(String[] args) {

        MyThread[] mts = new MyThread[10];
        for(int i = 0 ; i < mts.length ; i++){
            mts[i] = new MyThread();
        }

        for (int j = 0; j < mts.length; j++) {
            mts[j].start();
        }
    }
}
```

得到的结果是

```
Thread-8:760429199
Thread-6:760429199
Thread-7:760429199
Thread-5:760429199
Thread-9:760429199
Thread-4:760429199
Thread-0:760429199
Thread-3:760429199
Thread-2:760429199
Thread-1:760429199
```

从运行结果来看，该中方法保证了多线程并发下的线程安全性。

这里在声明变量时使用了volatile关键字来保证其线程间的可见性；在同步代码块中使用二次检查，以保证其不被重复实例化。集合其二者，这种实现方式既保证了其高效性，也保证了其线程安全性。

