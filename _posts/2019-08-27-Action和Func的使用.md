---
title: 委托 delegate
tags:委托 delegate Action Func lambda
key: 20190827
---

委托 delegate
===============
委托 delegate
------
委托是c#中类型安全的,可以订阅一个或多个具有相同签名方法的函数指针
声明委托的方式：delegate 返回值类型 委托类型名(参数)

Action
------
Action委托有两种方式：无参数无返回值的委托，有至少一个最多16个的参数无返回值的泛型委托。

{% highlight ruby linenos %}

	// 摘要: 
     //     封装一个方法，该方法不具有参数并且不返回值。
     [TypeForwardedFrom("System.Core, Version=3.5.0.0, Culture=Neutral, PublicKeyToken=b77a5c561934e089")]
     public delegate void Action();

{% endhighlight ruby %}


Func
------

如果要用有输入参数，有返回值的委托，那么Func委托将满足你的要求。
Func泛型委托，可以没有输入参数，但必须有返回值。根据输入参数的多少有17个重载


匿名函数 -（C# 编程指南） lambda
------------------------

匿名函数是一个“内联”语句或表达式，可在需要委托类型的任何地方使用。 可以使用匿名函数来初始化命名委托，或传递命名委托（而不是命名委托类型）作为方法参数。
可以使用 lambda 表达式或匿名方法来创建匿名函数。 建议使用 lambda 表达式，因为它们提供了更简洁和富有表现力的方式来编写内联代码。 与匿名方法不同，某些类型的 lambda 表达式可以转换为表达式树类型。
C# 中委托的演变

在 C# 1.0 中，通过使用在代码中其他位置定义的方法显式初始化委托来创建委托的实例。 C# 2.0 引入了匿名方法的概念，作为一种编写可在委托调用中执行的未命名内联语句块的方式。 C# 3.0 引入了 lambda 表达式，这种表达式与匿名方法的概念类似，但更具表现力并且更简练。 这两个功能统称为匿名函数。 通常，面向 .NET Framework 3.5 及更高版本的应用程序应使用 lambda 表达式。
下面的示例演示从 C# 1.0 到 C# 3.0 委托创建过程的发展：

{% highlight ruby linenos %}
class Test
{
    delegate void TestDelegate(string s);
    static void M(string s)
    {
        Console.WriteLine(s);
    }

    static void Main(string[] args)
    {
        // Original delegate syntax required 
        // initialization with a named method.
        TestDelegate testDelA = new TestDelegate(M);

        // C# 2.0: A delegate can be initialized with
        // inline code, called an "anonymous method." This
        // method takes a string as an input parameter.
        TestDelegate testDelB = delegate(string s) { Console.WriteLine(s); };

        // C# 3.0. A delegate can be initialized with
        // a lambda expression. The lambda also takes a string
        // as an input parameter (x). The type of x is inferred by the compiler.
        TestDelegate testDelC = (x) => { Console.WriteLine(x); };

        // Invoke the delegates.
        testDelA("Hello. My name is M and I write lines.");
        testDelB("That's nothing. I'm anonymous and ");
        testDelC("I'm a famous author.");

        // Keep console window open in debug mode.
        Console.WriteLine("Press any key to exit.");
        Console.ReadKey();
    }
}
/* Output:
    Hello. My name is M and I write lines.
    That's nothing. I'm anonymous and
    I'm a famous author.
    Press any key to exit.
 */
 
 {% endhighlight ruby %}