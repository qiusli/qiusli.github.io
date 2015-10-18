---
layout: post
title: "C#中的Delegate和Event"
date: 2015-10-17 15:06:20 -0600
comments: true
categories: 科技
---
简单来说，deletegate就是一个指向function的指针，下面是一个其最简单的用法：

``` c#
delegate returntype delegateName(parameters);
```

<!-- more -->

上面的代码*声明*了一个delegate，但是它并没有指向任何方法，有2种方法让delegate指向某个方法

1. 使用new创建，然后把需要指向的方法作为参数
2. 直接赋值

``` c#
1. delegateName dl1 = new delegateName(someMethod);
2. delegateName dl2 = someMethod;
```

当完成上面2步后，delegate就指向了某个方法，然后可以直接通过delegate调用这个方法：
``` c#
dl1(parameters);
```

既然我们可以直接调用类的方法，为什么还要使用delegate？下面这个例子来说明原因：

``` c#
namespace Delegate_Event
{
	class Calculator
	{
		public int add(int a, int b)
		{
			return a + b;
		}

		public int minus(int a, int b)
		{
			return a - b;
		}
	}

	class MainClass
	{
		delegate int dele(int x, int y);
		public static void Main (string[] args)
		{
			Calculator cal = new Calculator ();
			Console.WriteLine (cal.add (1, 3));

			dele objDele = cal.add;
			Console.WriteLine(objDele (11, 2));
		}
	}
}
```
我们把MainClass想象成client，Calculator想象成server，client想要调用server端的代码，首先须奥创建客户端的对象，然后通过对象来调用。这违反了encapsulation原则，因为server端的所有东西都暴露给了client。下面我在client端添加一个delegate，然后再添加一个方法来决定delegate指向的方法：

``` c#
namespace Delegate_Event
{
	class Calculator
	{
		public delegate int CalculatorDelegate(int a, int b);
		public int add(int a, int b)
		{
			return a + b;
		}

		public int minus(int a, int b)
		{
			return a - b;
		}

		public CalculatorDelegate assignMethod(string methodName) 
		{
			CalculatorDelegate delegateObj = null;
			if(methodName.Equals ("add"))
			{
				delegateObj = add;
			}

			if(methodName.Equals ("minus"))
			{
				delegateObj = minus;
			}

			return delegateObj;
		}
	}

	class MainClass
	{
		delegate int dele(int x, int y);
		public static void Main (string[] args)
		{
			Calculator cal = new Calculator ();
			Console.WriteLine (cal.assignMethod ("add").Invoke (10, 22));
		}
	}
}
```
这样，如果我需要在server端做出一些改变(例如新增一个multiply方法)，我在client端所做的改变就会很少，只需修改assignMend中的参数，分离了server端和client端的耦合。

#Multicast Delegate#
如果我想要让我的代理指向多个方法，那么在执行这个代理时，所有被指向的方法都会被一一执行。语法如下：

``` c#
int add(int a, int b){...}
int minus(int a, int b){...}
int multiply(int a, int b){...}

delegate theDele(int a, int b);
theDele += add;
theDele += minus;
theDele += multiply;
theDele(10, 20);
```

#Multicast Delegate的应用#
在pubisher-subscriber的模式中使用得最多。比如我有一个server端发送信息，多个client会接受信息:

``` c#
namespace Delegate_Event
{
	class Publisher
	{
		public delegate void PublishMsgDelegate(string msg);
		public PublishMsgDelegate publishMsg = null;
		public void publish(string msg) 
		{
			publishMsg (msg);
		}
	}

	class SendViaEmail
	{
		public void sendMsg(string msg)
		{
			Console.WriteLine (msg + " sent via email");
		}
	}

	class SendViaSMS
	{
		public void sendMsg(string msg)
		{
			Console.WriteLine (msg + " sent via SMS");
		}
	}

	class MainClass
	{
		public static void Main (string[] args)
		{
			Publisher pub = new Publisher ();
			SendViaEmail se = new SendViaEmail ();
			SendViaSMS sm = new SendViaSMS ();
			pub.publishMsg += se.sendMsg;
			pub.publishMsg += sm.sendMsg;
			pub.publish ("hello world");
		}
	}
}
```
这样只需在server端注册某个client就能给其发送消息。在这里有一个问题就是，无论我是否订阅了全部或者部分服务，我都会从所有渠道收到信息，因为server端就是这么设置的。为了解决这个问题，我们可以在SendViaMail和SMS里新增2个subscribe方法，说明是否订阅。

``` c#
class SendViaEmail
	{
		public void sendMsg(string msg)
		{
			Console.WriteLine (msg + " sent via email");
		}

		public void subscribe(Publisher pub)
		{
			pub.publishMsg += sendMsg;
		}
	}

	class SendViaSMS
	{
		public void sendMsg(string msg)
		{
			Console.WriteLine (msg + " sent via SMS");
		}

		public void subscribe(Publisher pub)
		{
			pub.publishMsg += sendMsg;
		}
	}

	class MainClass
	{
		public static void Main (string[] args)
		{
			Publisher pub = new Publisher ();
			SendViaEmail se = new SendViaEmail ();
			SendViaSMS sm = new SendViaSMS ();
			se.subscribe (pub);
//			sm.subscribe (pub);
			pub.publish ("hello world");
		}
	}
```
上面的改变给了我们更多的灵活性，即可以选择订阅。但是上面的方案也会有问题，它暴露了server端给client端，client端通过server端的delegate订阅，同时client端也能修改server端的delegate。比如:

``` c#
public void subscribe(Publisher pub)
{
	pub.publishMsg += sendMsg;
	pub.publishMsg = null;
}
```

它把server端的delegate所指向的方法全部给抹去了，这样所有client的所有订阅都全部失效。为了解决这个问题，event被引入了进来。

#Event#
Event的概念就是在client和server之间添加一个layer，让client不能随意改变server端的delegate指向，也不能在client端直接invoke。用event重写上面的代码:

``` c#
namespace Delegate_Event
{
	class Publisher
	{
		public delegate void PublishMsgDelegate(string msg);
		public event PublishMsgDelegate messageEvent = null;
		public void publish(string msg) 
		{
			messageEvent (msg);
		}
	}

	class SendViaEmail
	{
		public void sendMsg(string msg)
		{
			Console.WriteLine (msg + " sent via email");
		}

		public void subscribe(Publisher pub)
		{
			pub.messageEvent += sendMsg;
		}
	}

	class SendViaSMS
	{
		public void sendMsg(string msg)
		{
			Console.WriteLine (msg + " sent via SMS");
		}

		public void subscribe(Publisher pub)
		{
			pub.messageEvent += sendMsg;
		}
	}

	class MainClass
	{
		public static void Main (string[] args)
		{
			Publisher pub = new Publisher ();
			SendViaEmail se = new SendViaEmail ();
			SendViaSMS sm = new SendViaSMS ();
			se.subscribe (pub);
			sm.subscribe (pub);
			pub.publish ("hello world");
		}
	}
}
```
如果驶入在SendViaEmail和SendViaSMS中调用和修改event，系统会报错。简而言之，在client端能对event所做的只能是添加或则删除操作。