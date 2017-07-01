---
layout: post
title:    "��ʹ��Spring����Bean��װ�䣨����"
date:    2017-07-01 23:21:41 +0800
categories:    Spring
comments: true
---

* content
{:toc}

��ʹ��Spring����Bean��װ�䣨����
==

����������������У���������һ�����Ķ�����ɵģ���ͬ�Ķ�����Ĺ���һ����˵�ǲ�һ���ġ��������Ҫʵ��ĳһ�����ܣ�ͨ�����ɶ������Эͬ������ʵ�ֵġ�

����Ͷ���֮���Эͬ����ζ�Ŷ���Ͷ���֮�������������������������һ�����������һ����������ã����£�
```
public class Foo {
	private Boo boo;
}
```
����������棬��Foo������boo��������Boo�������Foo������Boo��

��ô�������ڣ�Foo��γ�ʼ�����Boo�أ�
```
// ʵ��1
public class Foo {
	...
	public Foo() {
		this.boo = new Boo();
	}
	...
}

// ʵ��2
public class Foo {
	...
	public Foo(Boo boo) {
		this.boo = boo;
	}
	...
}

// ʵ��3
public class Foo {
	...
	public void setBoo(Boo boo) {
		this.boo = boo;
	}
	...
}
```

ʵ��һ�������������⣺
* ����Boo������FooӦ�ø���Ĺ��ܡ���Υ���˵�һְ��ԭ��һ����ֻҪרע��ʵ���Լ��Ĺ��ܾͺ��ˣ���Ҫ�������������Foo�Ĳ���������δ���Boo����
* ��������ĳ��ԭ�򣬵���Boo�Ĺ��캯���仯�ˣ���ôFoo����ҲҪ���ű仯�����仯�ĺͲ��仯��Ҫ�ֿ��������Boo�Ĺ��캯���ǿ��ܻ�仯�ģ���Foo��Boo��������ʽ�ǲ���ģ�����Foo�ǲ��ܿ�����Boo��ʵ�ֶ�Boo�����������

ʵ�ֶ���ʵ������������������⡣��ô�鷳�ĵط����ڣ�������������Foo����������setter��������Ȼ������ͨ�������ķ�����Booע�롣�����������Ȼ����Spring��

Ԥ��֪ʶ
==

* Component�������һ���򵥵ĸ��Ŀǰ������Ϊһ���������һ���������������֮���������ϵ�������Ϊ������������װ��һ��
* Autowire���Զ�װ�䣬Spring������Զ�װ������Զ���Component��װ��һ�𡣣����������װ���Լ���������õģ�

ѧϰ���װ��Bean
==

* ��Java��ʽװ��Bean
```
Component
public class Boo {
	...
}

public class Foo {
	...
	@Autowire
	private Boo boo;
}

@Configuration
@ComponentScan
public class JavaConfig {
}
```
