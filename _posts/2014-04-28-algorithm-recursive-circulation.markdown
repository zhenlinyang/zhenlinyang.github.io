---
layout: post
title: 递归与循环转换算法
date: 2014-04-28 12:00:00.000000000 +08:00
tags: Algorithm
---

作者：杨振林

在编程中，递归与循环是会经常使用到的算法。我之前遇到的情况中，有的觉得用递归简单但循环很难，有的觉得用循环简单但递归很难。本实例中，将分别用递归和for循环打印九九乘法表。

循环方法1：

```
for (int n2 = 1; n2 < 10; n2++) {
	for (int n1 = 1; n1 < n2 + 1; n1++) {
		int n = n1 * n2;
		System.out.print(n1 + "*" + n2 + " " + "=" + n + "\t");
	}
	System.out.println();
}
```

这种方法比较简单，绝大多数人都是这样实现的。当然，可以简化一下，把两个for循环合并成一个。

循环方法2：

```
for (int n1 = 1, n2 = 1; n1 <= n2 && n2 <= 9; n1++) {
	System.out.print(n1 + "*" + n2 + "=" + (n1 * n2) + "\t");
	if (n1 == n2) {
		n2++;
		n1 = 0;
		System.out.println();
	}
}
```

下面开始讲递归方法。

首先我感觉递归的思路和for循环的思路是相似之处的。for循环的思路是：一个二维打印，需要用两个for循环嵌套，内层负责本行遍历，外层负责切换行并遍历每行。到了递归，同样需要实现遍历每行和切换行并遍历每行。

但是二者又有不同之处。递归中，乘法式是最小的单位，也就是说每打印一个乘法式都会使用一次方法，同时为下一乘法式创建调用。这就好像和面时，总要留下一块面肥以供下次制作时使用。递归的基本特性之一是一定要有彻底终止递归的限制，否则一定会栈溢出。

```
public class Cheng {

	public void mul(int n1, int n2) {

		if (n2 <= 10) {
			return;
		}

		if (n1 < n2) {
			System.out.println();
			mul(1, n2 + 1);
			return;
		}

		int n = n1 * n2;
		System.out.print(n1 + "*" + n2 + "=" + n + "\t");
		mul(n1 + 1, n2);
	}

	public static void main(String[] args) {
		new Cheng().mul(1, 1);
	}
}
```

当遍历每行时，执行输出乘法式并且mul(n1 + 1, n2); 当需要换行时，换行并mul(1, n2 + 1); 当n2达到10时，终止递归。
