---
title: "Java Synthetic"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Synthetic

```java
/**
 * @Author mubi
 * @Date 2020/11/22 13:11
 */
public class SyntheticTest {

	/*
	1. private 只能本类访问
	2. 但是内部类是可以被外部类访问的(与1有冲突)
	 */
	class SyntheticTestInner{
		private int i;

		// 私有构造方法
		private SyntheticTestInner() {
		}

		private SyntheticTestInner(int i) {
			this.i = i;
		}
	}

	public void test(){
		SyntheticTestInner syntheticTestInner = new SyntheticTestInner(10);
		System.out.println(syntheticTestInner.i);
	}

	public static void main(String[] args) {
		new SyntheticTest().test();
		// 如下可以看到 SyntheticTestInner 多了一个 access$000 方法，这种方法就是合成方法
		for(Method m : SyntheticTestInner.class.getDeclaredMethods()){
			System.out.println(m);
		}
	}
}
```
