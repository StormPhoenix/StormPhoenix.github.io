---
layout: post
title:    "Spring系列（一）"
description: 初识Spring
date:    2017-06-24 23:26:34 +0800
categories: blog | Spring
---

作为一个Java框架，Spring的根本使命就是简化Java开发。主要是让Java开发变得更加简单，更加具有可测试性，更加的松耦合。然而，Spring实现以上功能的仅仅只是基于少许的基本理念，比如说：松耦合、可重用等等。为了实现简化Java开发的使命，Spring采取了几种基本策略，如下。


Spring采用的几种基本策略
==
* 基于对象的轻量级和最小侵入性
非侵入性编程，通俗点说就是某一个类在使用了框架和不使用框架的情况下都能够发挥相同的作用。用Struts2框架举个例子。
{% highlight ruby %}
public class HelloAction extends ActionSupport {
	public String execute() {
		return "Hello world!";
	}
}
{% endhighlight %}
在这个例子中如果想让`HelloAction`能够正常的工作，必须要继承`ActionSupport`，这就意味着`HelloAction`和Struts2框架死死的绑定在了一起。这就是侵入式，从字面意思理解就是Struts2“侵入了“`HelloAction`这个对象。这样带来了两个问题：
	1. 如果`HelloAction`的功能需要被另一个模块重用，那么目标模块就不得不导入Struts框架，否则`HelloAction`就无法工作，这样就加大了项目的代码冗余。
	2. 倘若决定不采用Struts2框架，那么这个`HelloAction`将无法发挥它的正常功能，内部的逻辑代码必须要重写。所以，以上两点就降低了代码的可重用度。

* 通过依赖注入（Dependency injection）和面向接口实现松耦合
程序中的对象和对象之间必然是有关联的。如果一个对象A需要使用B的某些功能，就可以称A对B产生了依赖。见如下代码

{% highlight ruby %}
public class Kid {
	private Ball ball;

	public Kid() {
		this.ball = new Ball();
	}

	public void playBall() {
		ball.play();
	}
}

public class Ball {
	public void play() {
		// do something
	}
}
{% endhighlight %}

这里的 Kid 依赖于 Ball ，所以在Kid的构造方法里面创建了Ball接口的一个实现——Basketball。那么问题来了：构造 Ball 是 Kid 应该负责的事情吗？如果 Basketball 的构造方法改变了，那么 Kid 的代码是不是也需要改变呢？

上面描述的就是两个类之间过度**耦合**的现象，它导致的问题就是：一个类如果修改了将导致其他的类变化，从而可能引入了“打地鼠”式的bug。

在看看下面两个修改过后的代码。

{% highlight ruby %}
public class Kid {
        private Ball ball;

	public Kid(Ball ball) {
		this.ball = ball;
	}
	// do something
}

public interface Ball {
	void play();
}
{% endhighlight %}

{% highlight ruby %}
public class Kid {
	private Ball ball;

	public void setBall(Ball ball) {
		this.ball = ball;
	}
    // do something
}

public interface Ball {
	void play();
}

{% endhighlight %}

Kid 不需要知道 Ball 是如何具体实现的，所以通过构造器或者setter方法注入 Ball —— **依赖注入（Dependency Injection , DI)**。

Kid 不需要知道 Ball 是什么类型的，不需要知道到底是篮球还是足球，只需要知道 Ball 可以用来 play，所以 Ball 用接口进行替代—— **面向接口编程**。

好处就是，不管 Ball 的实现如何变化，都不会影响到 Kid 的代码。（推荐一本书： Dhanji R. Prasanna 的 Dependency Injection，据说很不错）

问题在于，这个构造器或者setter方法的中的 Ball 又是如何注入的？在 Spring 中，你不需要手动调用构造器或者setter方法，Spring会替你做的。关于这一点以后会讲到。

* 基于切面（aspect-oriented programming）和惯例进行声明式编程
  在一个应用中可能有许多地方都使用了相同的功能，这样导致的问题就是——代码冗余。同一个功能的代码重复的分布在系统的各个地方，如果需要对功能进行修改，就不得不在许多个功能调用点对代码进行修改。而AOP的作用，就是将这些地方的功能抽取出来形成一个独立的**组件**，在需要的地方对组件进行调用。形象化的说，调用**组件**的地方称之为**切点**，所谓组件的调用就是将某个组件插入到某个**切点**之中。不过令人惊喜的是，用Spring实现AOP甚至都不需要在切点编写调用方法！只要在配置文件声明切点的位置和相应的组件就行了。

* 通过切面和模板减少样板式代码
我见过最多的样板代码就是JDBC操作。Spring 提供了许多这样的模板，让开发人员只需要关注应用的逻辑。

Bean和Bean容器
==

一个应用之中是有很多的对象的，对象和对象之间有着各种相互依赖的关系。在 Spring 中，每个对象被称之为 **Bean**，所有的 Bean 都被容纳在 Spring 的一个容器（Container）里面，这个容器就是 **Bean 容器**，而 Bean 和 Bean 之间的关系是 Bean 容器利用 DI 进行处理的。

容器里面的对象是有生命周期的，Bean 容器管理着 Bean 的生命周期。Bean 容器会在每个 Bean 的不同生命周期阶段调用不同的生命周期方法（和Android中的Activity一模一样），下面举一个例子：

{% highlight ruby %}
public class MyBean implements BeanNameAware {
	/**
	* 如果MyBean处于设置BeanName的生命周期阶段，并且MyBean实现了
	* BeanNameAware接口，那么对应的setBeanName方法就会被调用，
	*/
	@Override
	public void setBeanName(String beanName) {
		// do something
	}
}
{% endhighlight %}

Bean 容器是通过 bean 工厂 / 应用上下文（ApplicationContext）来创建所有的 bean 对象的，bean 工厂了解的不多，主要讲解应用上下文。
* AnnotationConfigApplicationContext
* ClassPathXmlApplicationContext
* 其他 ... ...

上面列出了两个常用的应用上下文类。如果你在main方法中创建了相应的 Application Context，它就会自动加载相关的配置文件，从配置文件中读取它需要创建哪些 bean，并初始化它们。不同的 ApplicationContext 的区别在于加载的配置文件的不同。其中 AnnotationConfigApplicationContext 通过读取 Java 注解来创建 Bean ，而 ClassPathXmlApplicationContext 则是通过读取 xml 文件来读取。

Spring还可以做什么
==

基于 Spring 的核心容器上，Spring 还有着许多功能模块。

* SpringMVC —— 网站开发
* 数据集成和访问 —— 数据库操作
* 其他许多 ... ...
