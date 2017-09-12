---
title: DesignPattern-Adapter
date: 2016-01-01 08:27:02
tags:
- Adapter
categories:
- DesignPattern
---

# 适配器模式 Adapter

## 适配器模式种类
> 1. 对象适配器（组合）
> 2. 类适配器（多重继承）

## 定义
> 将一个类的接口，转换成客户期望的另一个接口。
适配器让原本接口不兼容的类可以合作无间。

## 目的
> 在不改变原有设计的基础上（或者说在无法改变原有设计的时候），为适应新的需求而作出的一种方案。

## 应用场景
> Android中的ListView、RecyclerView的Adapter，通过相同的Adapter接口方法，使用不同的Adapter算法，展现出不同的布局效果。

## 例子
1. 插座和插头
2. Java中的枚举器
> 1. 旧API<Interface>Enumeration，
hasMoreElements()、nextElement()；
> 2. 新API<Interface>Iterator，
hasNext()、next()、remove()；
> 3. 设计适配器兼容新旧API（继承Interator，组合Enumeration）
<Interface>EnumerationInterator，
hasNext()、next()、remove()；
> 4. 类图
![设计适配器兼容新旧API类图](/images/DesignPattern/Adapter_Pattern_Implement_Enumerator.png)

## 优点
> 可复用，系统需要使用现有的类，而此类的接口不符合系统的需要。那么通过适配器模式就可以让这些功能得到更好的复用。
可扩展，在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。

## 缺点
> 过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

## 注意
适配器的工作量，和目标接口的大小成正比。

## 类图
![适配器模式类图](/images/DesignPattern/Adapter_Pattern_Class_Diagram.png)

## 代码示例

### 目标接口
```
package com.designpattern.adapter;

/**
 * 目标接口
 * @author atopom
 *
 */
public interface Duck {
	void quack();
	void fly();
}
```

### 目标接口实现
```
package com.designpattern.adapter;

/**
 * 目标接口实现类
 * @author atopom
 *
 */
public class MallardDuck implements Duck {

	@Override
	public void quack() {
		System.out.println("Quack");
	}

	@Override
	public void fly() {
		System.out.println("I'm flying");
	}

}
```

### 被适配者Adaptee
```
/**
 * 被适配者Adaptee：Turkey
 * @author atopom
 *
 */
public interface Turkey {
	void gobble();
	void fly();
}
```

### Adaptee实现类
```
package com.designpattern.adapter;

/**
 * 被适配者实现类
 * @author atopom
 *
 */
public class WildTurkey implements Turkey {

	@Override
	public void gobble() {
		System.out.println("Gobble gobble");
	}

	@Override
	public void fly() {
		System.out.println("I'm flying a short distance");
	}
}
```

### 适配者Adapter
```
package com.designpattern.adapter;

/**
 * 适配器Adapter
 * @author atopom
 *
 */
public class TurkeyAdapter implements Duck {

	Turkey turkey;

	public TurkeyAdapter(Turkey turkey) {
		this.turkey = turkey;
	}

	@Override
	public void quack() {
		turkey.gobble();
	}

	@Override
	public void fly() {
		for(int i = 0; i < 5; i++) {
			turkey.fly();
		}
	}
}
```

### 测试类
```
package com.designpattern;

import com.designpattern.adapter.*;

/*
 * 适配器模式
 *
 * 将一个类的接口，转换成客户期望的另一个接口。
 * 适配器让原本接口不兼容的类可以合作无间。
 *
 * 适配器的工作量，和目标接口的大小成正比。
 *
 * 目的：
 * 在不改变原有设计的基础上（或者说，在无法改变原有设计的时候），
 * 为适应新的需求而做出的一个方案。
 */

/**
 * 适配器模式测试类
 * @author atopom
 *
 */
public class AdapterTestDrive {

	public static void main(String[] args) {
		MallardDuck duck = new MallardDuck();

		WildTurkey turkey = new WildTurkey();
		Duck turkeyAdapter = new TurkeyAdapter(turkey);

		System.out.println("The Turkey says...");
		turkey.gobble();
		turkey.fly();

		System.out.println("\nThe Duck says...");
		testDuck(duck);

		System.out.println("\nThe TurkeyAdapter says...");
		testDuck(turkeyAdapter);
	}

	static void testDuck(Duck duck) {
		duck.quack();
		duck.fly();
	}
}
```

### 结果
```
The Turkey says...
Gobble gobble
I'm flying a short distance

The Duck says...
Quack
I'm flying

The TurkeyAdapter says...
Gobble gobble
I'm flying a short distance
I'm flying a short distance
I'm flying a short distance
I'm flying a short distance
I'm flying a short distance
```

# 代码地址
> https://github.com/atopom/java_familiar_strange/tree/master/Code/AdapterPattern
