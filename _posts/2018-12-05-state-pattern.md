---
layout: post
title: 状态模式 (State Pattern) - 处理对象的多种状态及其相互转换
description: 允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。其别名为状态对象(Objects for States)，状态模式是一种对象行为型模式。
author: jinzhg
date: 2018-12-05 13:37 +0800
categories:
- Tutorial
- DesignPattern
tags:
- design pattern
media_subpath: "/assets/img"
---

## 结构图
![](state-pattern.png)

- Context（环境类）：环境类又称为上下文类，它是拥有多种状态的对象。由于环境类的状态存在多样性且在不同状态下对象的行为有所不同，因此将状态独立出去形成单独的状态类。在环境类中维护一个抽象状态类State的实例，这个实例定义当前状态，在具体实现时，它是一个State子类的对象。
- State（抽象状态类）：它用于定义一个接口以封装与环境类的一个特定状态相关的行为，在抽象状态类中声明了各种不同状态对应的方法，而在其子类中实现类这些方法，由于不同状态下对象的行为可能不同，因此在不同子类中方法的实现可能存在不同，相同的方法可以写在抽象状态类中。
- ConcreteState（具体状态类）：它是抽象状态类的子类，每一个子类实现一个与环境类的一个状态相关的行为，每一个具体状态类对应环境的一个具体状态，不同的具体状态类其行为有所不同。

## 示例
{% highlight c# %}
using System;

namespace DesignPatterns.State
{
    // The Context defines the interface of interest to clients. It also
    // maintains a reference to an instance of a State subclass, which
    // represents the current state of the Context.
    class Context
    {
        // A reference to the current state of the Context.
        private State _state = null;

        public Context(State state)
        {
            this.TransitionTo(state);
        }

        // The Context allows changing the State object at runtime.
        public void TransitionTo(State state)
        {
            Console.WriteLine($"Context: Transition to {state.GetType().Name}.");
            this._state = state;
            this._state.SetContext(this);
        }

        // The Context delegates part of its behavior to the current State
        // object.
        public void Request1()
        {
            this._state.Handle1();
        }

        public void Request2()
        {
            this._state.Handle2();
        }
    }
    
    // The base State class declares methods that all Concrete State should
    // implement and also provides a backreference to the Context object,
    // associated with the State. This backreference can be used by States to
    // transition the Context to another State.
    abstract class State
    {
        protected Context _context;

        public void SetContext(Context context)
        {
            this._context = context;
        }

        public abstract void Handle1();

        public abstract void Handle2();
    }

    // Concrete States implement various behaviors, associated with a state of
    // the Context.
    class ConcreteStateA : State
    {
        public override void Handle1()
        {
            Console.WriteLine("ConcreteStateA handles request1.");
            Console.WriteLine("ConcreteStateA wants to change the state of the context.");
            this._context.TransitionTo(new ConcreteStateB());
        }

        public override void Handle2()
        {
            Console.WriteLine("ConcreteStateA handles request2.");
        }
    }

    class ConcreteStateB : State
    {
        public override void Handle1()
        {
            Console.Write("ConcreteStateB handles request1.");
        }

        public override void Handle2()
        {
            Console.WriteLine("ConcreteStateB handles request2.");
            Console.WriteLine("ConcreteStateB wants to change the state of the context.");
            this._context.TransitionTo(new ConcreteStateA());
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            var context = new Context(new ConcreteStateA());
            context.Request1();
            context.Request2();
        }
    }
}
{% endhighlight %}

### 运行结果
```
Context: Transition to ConcreteStateA.
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB.
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA.
```

## 总结
状态模式将一个对象在不同状态下的不同行为封装在一个个状态类中，通过设置不同的状态对象可以让环境对象拥有不同的行为，而状态转换的细节对于客户端而言是透明的，方便了客户端的使用。在实际开发中，状态模式具有较高的使用频率，在工作流和游戏开发中状态模式都得到了广泛的应用，例如公文状态的转换、游戏中角色的升级等。

### 优点
1. **封装了状态的转换规则**，在状态模式中可以将状态的转换代码封装在环境类或者具体状态类中，可以对状态转换代码进行集中管理，而不是分散在一个个业务方法中。
2. **将所有与某个状态有关的行为放到一个类中**，只需要注入一个不同的状态对象即可使环境对象拥有不同的行为。
3. **允许状态转换逻辑与状态对象合成一体，而不是提供一个巨大的条件语句块**，状态模式可以让我们避免使用庞大的条件语句来将业务方法和状态转换代码交织在一起。
4. 可以**让多个环境对象共享一个状态对象**，从而减少系统中对象的个数。

### 缺点
1. 状态模式的使用**必然会增加系统中类和对象的个数，导致系统运行开销增大**。
2. 状态模式的结构与实现都较为复杂，**如果使用不当将导致程序结构和代码的混乱，增加系统设计的难度**。
3. 状态模式**对“开闭原则”的支持并不太好**，增加新的状态类需要修改那些负责状态转换的源代码，否则无法转换到新增状态；而且修改某个状态类的行为也需修改对应类的源代码。

### 适用场景
1. 对象的行为依赖于它的状态（如某些属性值），状态的改变将导致行为的变化。
2. 在代码中包含大量与对象状态有关的条件语句，这些条件语句的出现，会导致代码的可维护性和灵活性变差，不能方便地增加和删除状态，并且导致客户类与类库之间的耦合增强。