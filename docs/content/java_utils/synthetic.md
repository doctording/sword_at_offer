---
title: "Java Synthetic"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Synthetic

```bash
英[sɪnˈθetɪk]
美[sɪnˈθetɪk]
adj. (人工)合成的; 人造的; 综合(型)的;
n. 合成物; 合成纤维(织物); 合成剂;
```

有synthetic标记的field和method是class内部使用的；非static的inner class里面都会有一个`this$0`的字段保存它的父对象

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
