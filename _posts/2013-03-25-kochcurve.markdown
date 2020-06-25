---
layout: post
title: 科赫曲线
date: 2013-03-25 12:00:00.000000000 +08:00
tags: Java
---

作者：杨振林 yangzhenlin.com

下面我分享一下我用递归绘制科赫曲线的基本过程。

绘制科赫曲线，我首先想到了从简到难的基本思路。最开始的时候，只有一条直线。然后中间三分之一的部分突起成为三角状。当我想到这里的时候，我在想如何把中间三分之一的横线擦除。但又想了一想，我发现一边画一边擦并不是非常好的方法。如果可以规划好各个折点，最后一起连接起来，应该比较方便。

因为这不是单纯的一条链的循环，而是一变四、四变十六的形式，所以我想到了递归的方法。因为在递归中，可以用递归的形式， KochPaint(x1, y1, x3, y3, level);KochPaint(x3, y3, x5, y5, level);KochPaint(x5, y5, x4, y4, level);KochPaint(x4, y4, x2, y2, level); 一步步地深入下去。


我将整个项目分为两个文件。一个是 Koch.java 主要用来实现窗体等。另一个是 KochListener.java 主要用来实现监听器。

在 Koch.java 中，我实现了窗体、设置了拉杆（用来调节层数）、获取了画布。并将拉杆和画布传入到了 KochListener.java 中的鼠标监听器中去。

在 KochListener.java 中，用 KochListener 类继承了 java.awt.event.MouseListener 接口，并在下面实现了全部方法。用 mouseClicked 点击获取到了起始的画点，并在类最后用 KochPaint(x1, y1, x2, y2, level); 使用自定义的递归方法。

关于递归方法，在一次使用的过程中，有五个点是最重要的，即层数是2的时候的五个折点（左点（1）、左三分之一点（3）、中间凸起点（5），右三分之一点（4），右点（2））。然后将五个点按顺序两两传到下一层中。这样就基本实现了递归方法绘制科赫曲线的功能。这时出现了一个问题，是关于中间凸起点的。我在绘制的过程中，有时这个点是凸起的，有时竟然凹陷下去了。经过分析我发现，1号点和2号点是有方向的，有时可能出现1号点在2号点的右侧，所以如果方法不得当，定义5号点的时候反向，就可能出现凹陷的情况。只要保证5号点在以1号点为圆心，1、2号点为半径的逆时针旋转小于90度的范围内就是正确的。所以，根据1、2号点的位置情况，用if-else写出了不同的定义5号点的方法。根据分析，1、2号点只可能出现六种不同的位置，具体见下方代码。

递归方法自身调用容易让人有一些疑惑。事实上，递归到最后一定要停下来的。所以，在每深入一层使用递归的时候，要有一些不同从而判断是否应该停止递归。用计数器是一个比较方便的方法。每深入一层，计数器减一，直到减到特定数值为止。

最后，实现了用递归绘制科赫曲线。

需要更深入学习的地方：

* 优化算法

* 绘制更复杂的科赫曲线

* 进一步学习拉杆JSlider的相关知识

代码如下

Koch.java

```
import java.awt.Color;
import java.awt.FlowLayout;
import java.awt.Graphics;
import java.awt.event.MouseListener;
import javax.swing.JFrame;
import javax.swing.JSlider;

/**
 * 
 * @author yangzhenlin
 * 
 */
public class Koch extends JFrame {
	/**
	 * 主函数
	 * 
	 * @param args
	 */
	public static void main(String[] args) {
		Koch koch = new Koch();
		koch.initUI();
	}

	/**
	 * initUI
	 */
	public void initUI() {
		this.setTitle("科赫曲线");
		this.setSize(500, 500);
		this.setDefaultCloseOperation(3);
		this.setLocationRelativeTo(null);
		this.setResizable(true);
		this.setLayout(new FlowLayout());

		/**
		 * 设置拉杆
		 */
		JSlider js = new JSlider(1, 7);
		this.add(js);

		this.setVisible(true);

		/**
		 * 获取画布
		 */
		Graphics g = this.getGraphics();
		/**
		 * 设置背景颜色
		 */
		this.getContentPane().setBackground(Color.BLACK);
		MouseListener kl = new KochListener(g, js);
		this.addMouseListener(kl);
	}

}
```

KochListener.java

```
import java.awt.Color;
import java.awt.Graphics;
import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;
import javax.swing.JSlider;

/**
 * 
 * @author yangzhenlin
 *
 */

public class KochListener implements MouseListener {

	double x1, x2, y1, y2, x3, y3, x4, y4, x5, y5;
	Graphics g;
	JSlider js;
	private int level;

	public KochListener(Graphics g, JSlider js) {
		this.g = g;
		this.js = js;

		/**
		 * 设置颜色
		 */
		g.setColor(Color.GREEN);
	}

	/**
	 * 点击
	 */
	public void mouseClicked(MouseEvent e) {

		/**
		 * 设置层数
		 */
		level = this.js.getValue();


		x1 = e.getX();
		y1 = e.getY();
		x2 = x1 + 400;
		y2 = y1;

		KochPaint(x1, y1, x2, y2, level);

	}

	public void KochPaint(double x1, double y1, double x2, double y2, int level) {

		if (level == 1) {

			g.drawLine((int) x1, (int) y1, (int) x2, (int) y2);

		} else if (level > 1) {
			level--;
			double x3 = (2 * x1 + x2) / 3;
			double y3 = (2 * y1 + y2) / 3;
			double x4 = (2 * x2 + x1) / 3;
			double y4 = (2 * y2 + y1) / 3;
			double x5 = 0, y5 = 0;


			/**
			 * 水平线中间凸起点位置是(x5,y5)
			 */
			
			/**
			 * 0度
			 */
			if ((x3<x4)&&(y3==y4)) {
				x5 = (x3 + x4) / 2;
				y5 = y3 - (x4 - x3) * Math.sqrt(3) / 2;
			}

			/**
			 * 60度
			 */
			else if ((x3<x4)&&(y3>y4)) {
				x5 = x1;
				y5 = y4;
			}
			
			/**
			 * 120度
			 */
			else if ((x3>x4)&&(y3>y4)) {
				x5 = x2;
				y5 = y3;
			}
			
			/**
			 * 180度
			 */
			else if ((x3>x4)&&(y3==y4)) {
				x5 = (x3 + x4) / 2;
				y5 = y3 + (x3 - x4) * Math.sqrt(3) / 2;
			}
			
			/**
			 * 240度
			 */
			else if ((x3>x4)&&(y3<y4)) {
				x5 = x1;
				y5 = y4;
			}

			/**
			 * 300度
			 */
			else if ((x3<x4)&&(y3<y4)) {
				x5 = x2;
				y5 = y3;
			}

			KochPaint(x1, y1, x3, y3, level);
			KochPaint(x3, y3, x5, y5, level);
			KochPaint(x5, y5, x4, y4, level);
			KochPaint(x4, y4, x2, y2, level);

		}

	}

	public void mousePressed(MouseEvent e) {
	}

	public void mouseReleased(MouseEvent e) {
	}

	public void mouseEntered(MouseEvent e) {
	}

	public void mouseExited(MouseEvent e) {
	}

}
```

运行结果如下

![](//source.yangzhenlin.com/kochcurve/001.png)

附jar包

[koch.jar](//source.yangzhenlin.com/kochcurve/koch.jar)
