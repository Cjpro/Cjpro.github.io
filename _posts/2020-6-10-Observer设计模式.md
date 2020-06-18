---
layout:     post
title:      Observer设计模式
subtitle:   多线程中的观察者模式强弱智能指针
date:       2020-6-10
author:     Cory
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - C++
    - 设计模式
    - 网络编程
---
# Observer设计模式

### 意图：

定义对象间的一种一对多的关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

### 动机：

将一个系统设计成一系列相互协作的类有一个常见的副作用：需要维护相关对象之间的一致性。

观察者模式定义一种交互，即发布-订阅：

- 一个对象当自身状态发生改变时，会发出通知，但是并不知道谁是他的接收者，但每个接收者都会接收到通知，这些接受者称为观察者。
- 作为对通知的响应，每个观察者都将查询目标状态，然后改变自身的状态以和目标状态进行同步。

### 使用场景：

- 使对象封装为独立的改变和使用；
- 一个对象改变的同事需要改变其他对象，而不知道具体有多少对象需要改变；
- 不希望对象是紧耦合的；

### 结构：

![TIM截图20200610133036](https://i.loli.net/2020/06/10/f1t6p3wgR5JhdQz.jpg)



### 参与者：

Subject：目标，知道它的观察者，提供注册和删除观察者对象的接口

Observer：观察者，为那些在目标发生改变时需获得通知的对象定义一个更新接口

ConcreteSubject：具体目标，存储对象状态，状态改变时，向各个观察者发出通知

ConcreteObserver：具体观察者，维护一个指向ConcreteSubject对象的引用，存储有关状态，实现更新接口update，使自身状态与目标的状态保持一致

### 优缺点：

1 目标和观察者之间松耦合

2 支持广播通信：Subject发送的通知不需要指定它的接受者。通知被自动广播给所有已向该目标对象登记的有关对象。

3 意外的更新：看似无害的操作可能会引起观察者错误的更新。

## 观察者模型的代码实现：

```c++
#include<iostream>

#include<list>

#include<unoreder_map>

#include<stirng>

#include<algorithm>

using namespace std;

//定义监听者基类
class Listener
{
  public:
    //基类ctor
    Listener(string name):_name(name){}
    //监听者处理消息事件的pure virtual接口
    virtual void handleMessage(int msgid)=0;
   protected:
    string _name;
};
//一个具体的监听者类Listener1
class Listener1 : public Listener
{
    Listener1(string name) : Listener(name){}
    //Listener1处理自己感兴趣的事件
    void handleMessage(int msgid)
    {
        cout << "listener:" << _name << "recv:" << msgid
            << "msg, handle it now" << endl;
    }
};
// 一个具体的监听者类Listener2
class Listener2 : public Listener
{
public:
	Listener2(string name) :Listener(name) {}
	// Listener2处理自己感兴趣的事件
	void handleMessage(int msgid)
	{
		cout << "listener:" << _name << " recv:" << msgid
			<< " msg, handle it now!" << endl;
	}
};

//实现观察者
class Observer
{
public:
	/*
	params:
	1. Listener *pListener: 具体的监听者
	2. int msgid： 监听者感兴趣的事件
	该函数接口主要用于监听者向观察者注册感兴趣的事件
	*/
    void registerListener(Listener *pListener, int msgid)
    {
        auto it = listenerMap.find(msgid);
        if (it == listenerMap.end())
        {
            //没人对msgid事件感兴趣过，第一次注册
            listenerMap[msgid].push_front(pListener);
        }
        else
        {
            // 直接把当前pListener添加到对msgid事件感兴趣的list列表中
			it->second.push_front(pListener);
        }
    }
    /*
	params:
	1. int msgid：观察到发生的事件id
	该函数接口主要用于观察者观察到事件发生，并转发到对该事件感兴趣
	的监听者
	*/
	void dispatchMessage(int msgid)
	{
		auto it = listenerMap.find(msgid);
		if (it != listenerMap.end())
		{
			for_each(it->second.begin(),it->second.end(),
                     [&msgid](Listener *pListener)->void    //lambda表达式
			{
				// 观察者派生事件到感兴趣的监听者，监听者通过handleMessage接口负责事件的具体处理操作
				pListener->handleMessage(msgid);
			});
		}
	}
private:
    // 存储监听者注册的感兴趣的事件
    unordered_map<int,list<Listener*>> listenerMap;
};

int main()
{
    Listener *p1 = new Listener1("张三");
	Listener *p2 = new Listener2("李四");

	Observer obser;
	// 监听者p1注册1，2，3号事件
	obser.registerListener(p1, 1);
	obser.registerListener(p1, 2);
	obser.registerListener(p1, 3);
	// 监听者p2注册1，3号事件
	obser.registerListener(p2, 1);
	obser.registerListener(p2, 3);

	// 模拟事件的发生
	int msgid;
	for (;;)
	{
		cout << "输入事件id:";
		cin >> msgid;
		if (-1 == msgid)
			break;
		// 通过用户手动输入msgid模拟事件发生，此处观察者派发事件
		obser.dispatchMessage(msgid);
	}

	return 0;
}
```



## 考虑观察者运行在多线程中

考虑这样一个问题，如果观察者Observer独立的运行在一个线程环境中，那么当它观察到事件发生时，它是这样通知监听者处理事件的，代码如下:

```c++
for_each(it->second.begin(),
				it->second.end(),
				[&msgid](Listener *pListener)->void
			{
				// 观察者派生事件到感兴趣的监听者，监听者通过handleMessage接口负责事件的具体处理操作
				pListener->handleMessage(msgid);
			});

```

遍历一个list<Listener*>的列表，然后回调每一个监听者的handleMessage方法进行事件派发，那么这就涉及到在多线程环境中，共享对象的线程安全问题（解决方法就是使用智能指针）

上面代码通过访问Listener*，也就是指向监听者的指针，在调用handleMessage时，其实在多线程环境中，肯定不明确此时监听者对象是否还存活，或是已经在其它线程中被析构了，此时再去通知这样的监听者，肯定是有问题的，也就是说，当Observer观察者运行在独立的线程中时，在通知监听者处理该事件时，应该先判断监听者对象是否存活，如果监听者对象已经析构，那么不用通知，并且需要从map表中删除这样的监听者对象。
那**当然是使用shared_ptr和weak_ptr智能指针**了，上面**使用Listener\*裸指针在多线程环境中肯定存在多线程访问共享对象的线程安全问题**，代码修改如下：

```cpp
// 实现观察者用强弱智能指针，解决多线程访问共享对象的线程安全问题
class Observer
{
public:
	/*
	params:
	1. Listener *pListener: 具体的监听者
	2. int msgid： 监听者感兴趣的事件
	该函数接口主要用于监听者向观察者注册感兴趣的事件
	*/
	void registerListener(weak_ptr<Listener> pwListener, int msgid)
	{
		auto it = listenerMap.find(msgid);
		if (it == listenerMap.end())
		{
			// 没人对msgid事件感兴趣过，第一次注册
			listenerMap[msgid].push_front(pwListener);
		}
		else
		{
			// 直接把当前pListener添加到对msgid事件感兴趣的list列表中
			it->second.push_front(pwListener);
		}
	}
	/*
	params:
	1. int msgid：观察到发生的事件id
	该函数接口主要用于观察者观察到事件发生，并转发到对该事件感兴趣
	的监听者
	*/
	void dispatchMessage(int msgid)
	{
		auto it = listenerMap.find(msgid);
		if (it != listenerMap.end())
		{
			for (auto it1 = it->second.begin();
				it1 != it->second.end();
				++it1)
			{
				// 智能指针的提升操作，用来判断监听者对象是否存活
				shared_ptr<Listener> ps = it1->lock();
				// 监听者对象如果存活，才通知处理事件
				if (ps != nullptr)
				{
					ps->handleMessage(msgid);
				}
				else
				{
					// 监听者对象已经析构，从map中删除这样的监听者对象
					it1 = it->second.erase(it1);
				}
			}
		}
	}
private:
	// 存储监听者注册的感兴趣的事件
	unordered_map<int, list<weak_ptr<Listener>>> listenerMap;
};
```

### 参考目录：

链接：https://blog.csdn.net/QIANGWEIYUAN/article/details/88745835

连接：https://www.cnblogs.com/suzhou/p/dp16obsvr.html
