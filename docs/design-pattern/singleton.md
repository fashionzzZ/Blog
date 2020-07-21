---
title: 设计模式-Singleton模式
author: Freedom
date: 2020-01-19
tags: [设计模式]
categories: 
- 设计模式
- 创建型
description: 创建型模式由两个主导思想构成。一是将系统使用的具体类封装起来，二是隐藏这些具体类的实例创建和结合的方式。
---

`创建型模式由两个主导思想构成。一是将系统使用的具体类封装起来，二是隐藏这些具体类的实例创建和结合的方式。`

### 单例模式（Singleton Pattern）

#### 定义

确保一个类只有一个实例，并提供该实例的全局访问点。

#### 类图

![单例模式类图](https://www.ultroncode.com/source/0a17dffafacec24a0dcfa15da758c487.png)

#### 说明

Singleton类称为单例类，通过使用private的构造函数确保了在一个应用中只产生一个实例，并且是自行实例化的。

#### 实现

##### 懒汉式-线程不安全

只有调用getInstance()方法才生成实例，如果多个线程能够同时进入 `if (instance == null)` ，并且此时 instance 为 null，那么会有多个线程执行 instance = new Singleton();` 语句，这将导致实例化多次 instance。

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

##### 饿汉式-线程安全

线程不安全问题主要是由于 instance 被实例化多次，采取直接实例化 instance 的方式就不会产生线程不安全问题。

但是直接实例化的方式也丢失了延迟实例化带来的节约资源的好处。

```java
public class Singleton {

    private static Singleton instance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

##### 双重校验锁-线程安全

instance 只需要被实例化一次，之后就可以直接使用了。加锁操作只需要对实例化那部分的代码进行，只有当 instance 没有被实例化时，才需要进行加锁。

双重校验锁先判断 instance 是否已经被实例化，如果没有被实例化，那么才对实例化语句进行加锁。

```java
public class Singleton {

    private volatile static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

> instance 采用 volatile 关键字修饰也是很有必要的，`instance = new Singleton();` 这段代码其实是分为三步执行：
>
> 1. 为 instance 分配内存空间
> 2. 初始化 instance
> 3. 将 instance 指向分配的内存地址
>
> 但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1>3>2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getInstance() 后发现 instance 不为空，因此返回 instance，但此时 instance 还未被初始化。
>
> 使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

#### 参考资料

- [单例](https://github.com/CyC2018/CS-Notes/blob/master/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20%20-%20%E5%8D%95%E4%BE%8B.md)