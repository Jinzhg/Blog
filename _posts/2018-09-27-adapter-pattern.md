---
layout: post
title: 适配器模式 (Adapter Pattern) - 协调不兼容的结构
description: 将一个接口转换成另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。
author: jinzhg
date: 2018-09-27 09:31 +0800
categories:
- Tutorial
- DesignPattern
tags:
- design pattern
media_subpath: "/assets/img"
---

在对象适配器模式中，适配器与适配者之间是关联关系；在类适配器模式中，适配器与适配者之间是继承（或实现）关系。

## 对象适配器

### 结构图
![](object-adapter.png)

- Target（目标抽象类）：目标抽象类定义客户所需接口，可以是一个抽象类或接口，也可以是具体类。
- Adapter（适配器类）：适配器可以调用另一个接口，作为一个转换器，对Adaptee和Target进行适配，适配器类是适配器模式的核心，在对象适配器中，它通过继承Target并关联一个Adaptee对象使二者产生联系。
- Adaptee（适配者类）：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法，在某些情况下可能没有适配者类的源代码。

### 示例
{% highlight c# %}
class Adapter : ITarget {
	private Adaptee adaptee; //维持一个对适配者对象的引用
	
	public Adapter(Adaptee adaptee) {
		this.adaptee=adaptee;
	}
	
	public void request() {
		adaptee.specificRequest(); //转发调用
	}
}
{% endhighlight %}

## 类适配器

### 结构图
![](class-adapter.png)

### 示例
{% highlight c# %}
class Adapter : Adaptee, ITarget {
	public void request() {
		this.specificRequest();
	}
}
{% endhighlight %}

## 双向适配器

### 结构图
![](bidirectional-adapter.png)

### 示例
{% highlight c# %}
class Adapter : ITarget, IAdaptee {
    //同时维持对抽象目标类和适配者的引用
	private ITarget target;
	private IAdaptee adaptee;
	
	public Adapter(ITarget target) {
		this.target = target;
	}
	
	public Adapter(IAdaptee adaptee) {
		this.adaptee = adaptee;
	}
	
	public void request() {
		adaptee.specificRequest();
	}
	
	public void specificRequest() {
		target.request();
	}
}
{% endhighlight %}

## 总结
适配器模式将现有接口转化为客户类所期望的接口，实现了对现有类的复用。

### 优点
1. **将目标类和适配者类解耦**，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构。
2. **增加了类的透明性和复用性**，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。
3. **灵活性和扩展性都非常好**，通过使用配置文件，可以很方便地更换适配器，也可以在不修改原有代码的基础上增加新的适配器类，完全符合“开闭原则”。

#### 类适配器模式优点
1. 由于适配器类是适配者类的子类，因此可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强。

#### 对象适配器模式优点
1. 一个对象适配器可以把多个不同的适配者适配到同一个目标。
2. 可以适配一个适配者的子类，由于适配器和适配者之间是关联关系，根据“里氏代换原则”，适配者的子类也可通过该适配器进行适配。

### 缺点

#### 类适配器模式缺点
1. 对于Java、C#等不支持多重类继承的语言，一次最多只能适配一个适配者类，不能同时适配多个适配者；
2. 适配者类不能为最终类，如在Java中不能为final类，C#中不能为sealed类；
3. 在Java、C#等语言中，类适配器模式中的目标抽象类只能为接口，不能为类，其使用有一定的局限性。

#### 对象适配器模式缺点
与类适配器模式相比，要在适配器中置换适配者类的某些方法比较麻烦。如果一定要置换掉适配者类的一个或多个方法，可以先做一个适配者类的子类，将适配者类的方法置换掉，然后再把适配者类的子类当做真正的适配者进行适配，实现过程较为复杂。

### 适用场景
1. 系统需要使用一些现有的类，而这些类的接口（如方法名）不符合系统的需要，甚至没有这些类的源代码。
2. 想创建一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作。