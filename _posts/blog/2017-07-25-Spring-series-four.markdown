---
layout: post
title:    "Spring系列（四）"
description: Bean的高级装配
date:    2017-07-25 22:11:24 +0800
categories:    Spring | blog
comments: true
---

预备知识
==
学习过Bean的基本装配知识即可。

问题的提出
==

上一篇博客的代码能够完全运行正确 : )。有一个地方不知道你们有没注意到，那就是 Wheel 的接口的实现只有一个 RubberWheel，所以不论是 Java注解利用自动扫描，还是用 Java代码方式实现装配，程序都能够正确判断要装配哪一个 Wheel，那么问题：如果 Wheel 有多个实现，那么 Spring是怎么判断的？见如下例子：

{% highlight java %}
@Component
public class RubberWheel implements Wheel {
	void role() {
		// do something
	}
}
@Component
public class SquareWheel implements Wheel {
	void role() {
		// do something
	}
}
@Component
public class Bench implements Car {
	/**
	* :( 自动扫描的时候，发现 SquareWheel 和 RubberWheel 都
	* 实现了 Wheel 这个接口，我要装配哪一个呢？？
	*/
  @Autowire
	private Wheel wheel;
	void drive() {
		wheel.role();
	}
}
{% endhighlight %}


如果程序运行起来，会抛出`org.springframework.beans.factory.NoUniqueBeanDefinitionException`异常，因为 Wheel 不知道要装配 `RubberWheel` 还是 `SquareWheel`。

如果用 Java代码方式也是存在这种问题的。比如：

{% highlight java %}
/**
* 这里生成了两个 Wheel，Bench 在装配的时候要选哪一个？
*/
@Configurable
public class JavaConfig {
    @Bean
    public Wheel getRubberWheel() {
        return new RubberWheel();
    }

    public Wheel getSquareWheel() {
		return new SquareWheel();
    }
}
{% endhighlight %}

Bean的条件化装配
==

用一个简单的 if - else 说明：
{% highlight ruby %}
if (路面很平坦) {
    选用 RubberWheel 来装配
} else if (路面不平) {
    选用 SquareWheel 来装配
}

// 根据不同的情况来装配不同的 Bean，接下来就要用代码来模拟不同的情况
// 让 Spring 根据情况来装配。
{% endhighlight %}

* **Spring 条件化装配**
这种方式是利用 `@Conditional` 注解，实现根据不同的条件来装配 Bean的功能。
{% highlight ruby %}
@Configurable
public class JavaConfig {
    /**
    * @Conditional 内的参数，必须要实现 Condition 这个接口
    * 这个接口会返回一个 bool 值，于是
    * bool 值为true：装配时选择这个 Bean
    * bool 值为true：装配时候不选择这个 Bean
    */
    @Bean
    @Conditional(RoadFlatConditional.class)
    public Wheel getRubberWheel() {
        return new RubberWheel();
    }

    // 其他的省略
}

// 看看是怎么实现的
public class RoadFlatConditional implements Condition {
    /**
    * 最重要的是传入的两个参数。你可以通过这两个参数程序运行的两大信息：
    * 1、系统的环境变量
    * 2、检查带有 @Bean 注解的方法上还有说明其他注解
    */
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // do something
    }
}
{% endhighlight %}
matches 方法的实现完全看个人，这里给一个样例：
{% highlight ruby %}
// 如果系统带有 flat 变量，那么方法返回 true，否则 false
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
	Environment environment = context.getEnvironment();
    return environment.containsProperty("flat");
}
{% endhighlight %}
这个环境变量有几种设置方法：

> 作为 DispatcherServlet 的初始化参数
> 作为 Web 引用的上下文参数
> 作为 JNDI 条目 （这个我不懂）
> 作为环境变量
> 作为 JVM 系统属性

我用的一个方法是在虚拟机启动的时候把 `flat` 设置成了环境变量。

* **利用Profile**
把 Profile 理解成一个环境，这个环境包含了一套 Bean。你现在有很多个 Profile，在运行时你来制定用哪一个Profile。

{% highlight ruby %}
/**
* 程序运行的时候，可以通过查看当前运行时的参数，确定
* 当前的 Profile 是 flat 还是 uneven，如果是 flat profile，
* 就返回 RubberWheel；如果是 uneven profile，则返回 SquareWheel。
*/
@Configurable
public class JavaConfig4 {
    @Bean
    @Profile("flat")
    public Wheel getRubberWheel() {
        return new RubberWheel()
    }

    @Bean
    @Profile("uneven")
    public Wheel getSquareWheel()) {
        return new SquareWheel();
    }
{% endhighlight %}
额外提一些。注释中说程序运行时查看的参数，具体是如下两个参数：

> spring.profiles.active
> spring.profiles.default

运行时查看 `spring.profiles.active` 的值，根据这个值设置 profile，如果这个值没有设置，就查看 `spring.profiles.default`来设置值。

至于如何设置的嘛，和上文一样，简单不解释。

* **使用限定符注解**
这样理解：有好多个实现相同接口的 Bean，我们要给这些 Bean 添加上一些不同的特征，到时候用的时候，可以根据我需要的特征挑选 Bean。

而所谓的添加特征，就是在 Bean 上添加 `@Qualifier`。添加上一个 `@Qualifier`注解，就代表给 Bean 添加一个特征。

这次我们用 `@Component` 的形式做例子

{% highlight ruby %}
@Component
@Qualifier("sweet")
public class Apple implements Fruit { // ... }

@Component
@Qualifier("water")
public class Watermelon implements Fruit { // ... }

/**
* 由于 '苹果' 和 '西瓜' 都实现了 Fruit 接口，
* 但是这里添加了 @Qualifier("sweet") 这个'甜味'特征，
* 所以 Apple 会被装配到这个地方。
*/
@Autowire
@Qualifier("sweet")
public void setFruit(Fruit fruit) {
	// do something
}
{% endhighlight %}

Bean 的作用域
==
你用`@Bean`创建了一个 Bean 是吧？这里值得注意的一个小问题就是 Bean 的作用域，就是 Bean 起作用的范围。以它默认的作用域`singleton`来说明，`singleton`指 Bean 的作用域是 Spring 应用的所有位置，这个 Bean 是单例的，应用中对这个 Bean 的引用都是同一个 Bean。

{% highlight ruby %}
    /**
    * 看这里，默认情况下 Wheel 这个 Bean 的作用域是 singleton，
    * 意味着应用中对这个 wheel 的引用都是同一个 wheel、整个应用中
    * 这存在这一个 wheel。
    *
    * 你可能会疑问这里明明采用了 new 关键字，应该是每次调用这个方法
    * 时都会创建一个 Wheel 的实例才对？
    *
    * 实际情况是 Spring 会对这个方法的每次调用进行拦截。第一次拦截时，
    * 会 new 出这个 Wheel，以后再次拦截的时候就发现已经创建这个 Bean
    * 了，所以会把之前创建的 Wheel 返回。从而实现了 singleton。
    */
    @Bean
    public Wheel getSquareWheel()) {
        return new SquareWheel();
    }

{% endhighlight %}

另外三个作用域是

* Prototype 每次需要使用 `@Bean`注解时，都会创建一个 Bean
* Session 在 Web 应用中，为每次会话创建一个 Bean
* Request 在 Web 应用中，为每次请求创建一个 Bean

解释一下 Web 应用中的 Session 作用域这例子：比如用户上网购物行为属于一个会话，购买的物品都要放入购物车（shopcart），如果要用一个 Bean 表示这个 shopcart ，那么这个 Bean 就必须要声明为 Session 作用域。

其他的作用域都有各自的类似场景。

声明 Bean 的作用域的方法如下
{% highlight ruby %}
    /**
    * 加上 @Scope 注解用来改变 Bean 的作用域，
    * ConfigurableBeanFactory.SCOPE_PROTOTYPE 参数代表
    * Prototype 作用域
    */
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public Wheel getSquareWheel()) {
        return new SquareWheel();
    }

    // 或者在 XML 文件里面声明
    <bean id="rubberWheel" class="com.stormphoenix.RubberWheel"
          scope="prototype"/>
{% endhighlight %}

关于 `@Scope` 的 `proxyMode` 属性需要说明一下。如下

{% highlight ruby %}
@Component
public class Bench implements Car {
    @Autowire
    Wheel wheel;
}

public class JavaConfig {
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION,
           proxyMode = ScopedProxyMode.INTERFACES)
    public Wheel getWheel() {
        return new RubberWheel();
    }
}
{% endhighlight %}

解释上述代码：
首先 Bench 的作用域是 `singleton`，Wheel 的作用域是 `session`，所以看出问题来吧了 : )

Bench 是一个，而 Wheel 有好多个（每发生一次会话就会创建一个），那么到底要装配哪一个 Wheel 到 Bench 里面呢？

首先要确定几点：
1. 被 `@Autowire` 注解的属性肯定只会装配一次。
2. Wheel 的作用域是 `session` ，肯定有很多个 Wheel 对象。
3. Bench 作用域是 `singleton` ，肯定只有一个 Bench 对象。
4. 一个解决方案是：给 Bench 注入一个对象，这个对象可以根据运行时不同的环境调用不同的 Wheel 对象的功能。

要解决这个问题就是利用 `@Scope` 的 `proxyMode` 参数，这个参数的值是 INTERFACES，表明当 Wheel 被装配的时候不装配 Wheel 对象本身，而装配`一个实现了 Wheel 对象功能的代理`。所以注入到 Bench 的 Wheel 根本就不是我们创建了 Wheel，而是一个`代理`，这个代理可以根据不同的会话环境，来调用对应的 Wheel 对象的功能。

其实懂了代理这个含义，上述代码就很好理解。请搜索一下设计模式中的代理模式！然后学习下 Java 中的代理！

最后 `proxyMode` 参数有除了 INTERFACES 之外的几个值，了解就好。

完毕
==
其他的就自己探索吧！还有一个 `@Primary`注解，被这个注解上的 Bean 会被有限装配。

完毕！
