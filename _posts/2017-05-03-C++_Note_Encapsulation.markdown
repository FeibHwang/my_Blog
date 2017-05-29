---
layout:     post
title:      "C++进阶：封装"
subtitle:   "C++笔记"
date:       2017-05-3 18:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
    - 学习笔记
---

# 类：Class

类：某一个概念的数据与方法的集合。通过类，我们定义了一个物体的自身信息与它可以做什么。

封装：把实现细节隐藏起来，选择性地暴露某些特性

# 对象实例化

实例化：将类作为模板制造出多个实体，计算机有两种实例化方式：从栈中实例化和从堆中实例化。

{% highlight c++ %}

class TV
{
public:
	char name[20];
	int type;

	void changeVol();
	void power();
};


int main(void)
{
	//栈中实例化

	TV tv;
	TV tv[20];

	//堆中实例化
	TV *p = new TV();
	TV *q = new TV[20];

	delete p;
	delete []q;

	return 0;
}

{% endhighlight %}

栈中实例化的对象会自动回收，而堆中实例化的对象不会，因而需要手动删除对象

# 对象成员访问

以下代码分别定义了两种对象访问方法：

{% highlight c++ %}

int main(void)
{
	TV tv;
	tv.type = 0;
	tv.changeVol();
	return 0;
}

int main(void)
{
	TV *p = new TV();
	p->type = 0;
	p->changeVol();
	delete p;
	p = NULL;
	return 0;
}

{% endhighlight %}

现在看一个Coordinate对象的实例化例子：

{% highlight c++ %}

#include "stdafx.h"
#include <iostream>
#include <stdlib.h>

using namespace std;

class Coordinate
{
public:
	int x;
	int y;

	void printX()
	{
		cout << x << endl;
	}

	void printY()
	{
		cout << y << endl;
	}
};



int main()
{
	//通过栈实例化
	Coordinate coor;
	coor.x = 10;
	coor.y = 20;
	coor.printX();
	coor.printY();

	//通过堆实例化
	Coordinate *p = new Coordinate();

	if (p == NULL)
	{
		cout << "Memory apply failed" << endl;
		return 0;
	}
	p->x = 100;
	p->y = 200;
	p->printX();
	p->printY();

	delete p;
	p = NULL;

	system("pause");
    return 0;
}

{% endhighlight %}

# 初始封装

在之前的例子中，我们通过直接对public成员进行赋值初始化，这是可行的，但违背了面向对象的思想。

面向对象基本思想：`谁`做`什么`，数据操作通过函数完成。

{% highlight c++ %}

class Student
{
public:
	void setAge(int _Age){age = _Age;}
	int getAge(){return age;}
private:
	string name;
	int age;
	......
}

{% endhighlight %}

封装的优点：

* 数据有效性检查
* 保护只读变量

一个使用封装访问的Student类例子

{% highlight c++ %}

#include "stdafx.h"
#include <iostream>
#include <stdlib.h>
#include <string>

using namespace std;


class Student
{
public:
	void setName(string _name) { m_strName = _name; }

	string getName() { return m_strName; }

	void setGender(string _gender) { m_strGender = _gender; }

	string getGender() { return m_strGender; }

	int getScore() { return m_iScore; }

	void initScore() { m_iScore = 0; }

	void study(int _score) { m_iScore += _score; }

private:
	string m_strName;
	string m_strGender;
	int m_iScore;
};



int main()
{
	Student stu;
	stu.initScore();
	stu.setName("zhangsan");
	stu.setGender("female");
	stu.study(5);
	stu.study(3);

	cout << stu.getName() << " " << stu.getGender() << " " << stu.getScore() << endl;
	system("pause");
    return 0;
}

{% endhighlight %}

## 关于内联函数

普通函数调用流程：

* 发现函数调用请求
* 找到函数调用入口
* 运行函数
* 返回原进程

内联函数在编译时会替换原函数调用所在位置，从而省去了调用跳转

## 关于类内定义与类外定义

之前的例子中类函数的实现是在类内进行的，对于编译器，类内函数会被作为内联函数优先编译，对于比较复杂的函数，推荐使用类外定义

类外定义分为同文件与分文件，分文件定义有诸多好处，以后再说。。。

