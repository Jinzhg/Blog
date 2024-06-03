---
layout: post
title: 单例模式 (Singleton Pattern)
description: 确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。
author: jinzhg
date: 2018-09-06 14:16 +0800
categories:
- Tutorial
- DesignPattern
tags:
- design pattern
media_subpath: "/assets/img"
---

## 饿汉式单例类
在定义静态变量的时候实例化单例类，因此在类加载的时候就已经创建了单例对象。

### 结构图
![](eager-singleton.png)

### 示例
{% highlight c# %}
class EagerSingleton 
{ 
    private static EagerSingleton instance = new EagerSingleton(); 
 
    private EagerSingleton() { } 
 
    public static EagerSingleton GetInstance() 
    {
        return instance; 
    }
}
{% endhighlight %}

## 懒汉式单例类与线程锁定
线程锁（synchronized或lock）。双重检查锁定。

### 结构图
![](lazy-singleton.png)

### 示例
{% highlight c# %}
class LazySingleton 
{ 
    private static LazySingleton instance = null; 
    //程序运行时创建一个静态只读的辅助对象
    private static readonly object syncRoot = new object();
 
    private LazySingleton() { } 
 
    public static LazySingleton GetInstance() 
    { 
        //第一重判断，先判断实例是否存在，不存在再加锁处理
        if (instance == null) 
        {
            //加锁的程序在某一时刻只允许一个线程访问
            lock(syncRoot)
            {
                //第二重判断
                if(instance==null)
                {
                    instance = new LazySingleton();  //创建单例实例
                }
            }
        }
        return instance; 
    }
}
{% endhighlight %}

## 静态内部类
既可以实现延迟加载，又可以保证线程安全，不影响系统性能

### 示例
{% highlight c# %}
class SingleMode
{
    private static class InterClass
    {
        internal static SingleMode instance = new SingleMode();
    }

    public static SingleMode GetInstance()
    {
        return InterClass.instance;
    }
}
{% endhighlight %}

## 总结
单例模式作为一种目标明确、结构简单、理解容易的设计模式，在软件开发中使用频率相当高，在很多应用软件和框架中都得以广泛应用。

### 优点
1. 单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它。
2. 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象单例模式无疑可以提高系统的性能。
3. 允许可变数目的实例。基于单例模式我们可以进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例，既节省系统资源，又解决了单例单例对象共享过多有损性能的问题。

### 缺点
1. 由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
2. 单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起。
3. 现在很多面向对象语言(如Java、C#)的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失。

### 适用场景
1. 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器，或者需要考虑资源消耗太大而只允许创建一个对象。
2. 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。