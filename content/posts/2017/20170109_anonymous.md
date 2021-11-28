---
title: 【转】C# 之 匿名方法
date: 2017-01-09 23:50:01
tags:
  - C#
  - .Net
  - Code
categories:
  - C#代码
---
> 转自http://www.cnblogs.com/daxnet/archive/2008/11/12/1687011.html

匿名方法是C# 2.0的语言新特性。首先看个最简单的例子：

<!--more-->


```c#
class Program   
{   
    static void Main(string[] args)   
    {   
        List<string> names = new List<string>();   
        names.Add("Sunny Chen");   
        names.Add("Kitty Wang");   
        names.Add("Sunny Crystal");   
           
        List<string> found = names.FindAll(   
            new Predicate<string>(NameMatches));   
  
        if (found != null)   
        {   
            foreach (string str in found)   
                Console.WriteLine(str);   
        }   
    }   
  
    static bool NameMatches(string name)   
    {   
        return name.StartsWith("sunny",    
            StringComparison.OrdinalIgnoreCase);   
    }   
}   
```


这段代码在开始的时候初始化了一个字符串列表（string list），然后通过列表的FindAll方法来查找以“sunny”起始的字符串，最后将所查找到的所有结果输出。

我们需要着重介绍List<T>的FindAll方法。这个方法是一个参数为Predicate<T>类型、返回值为List<T>类型的函数。注意，Predicate<T>是一个泛型委托，它指代这样一些函数：这些函数仅有一个T类型的参数，并且返回值是布尔类型。通过reflector等工具，我们可以看到Predicate<T>的定义如下：

public delegate bool Predicate<T>(T obj);   
至此我们也多少能够猜到FindAll方法的具体实现。即针对List<T>中的每个元素，调用Predicate<T>所指代的函数，如果函数返回为true，则将其加入新建的列表中。遍历完所有的元素后，将新建的列表返回给调用者。如下：

```c#
public List<T> FindAll<T>(Predicate<T> match)   
{   
    List<T> ret = new List<T>();   
    foreach (T elem in items)   
    {   
        if (match(elem))   
            ret.Add(elem);   
    }   
    return ret;   
}   
```

因此，针对上面的例子，要调用FindAll方法，我们必须先定义一个参数为string类型，返回值为布尔类型的函数，在这个函数中，对参数进行条件判断，如果符合条件（也就是以“sunny”作为起始字符串），那么就返回true，否则返回false。最后再将这个函数作为参数传递给FindAll。于是也就得到了最上面的代码。

在上面的例子中，为了调用FindAll方法，我们不得不新定义一个函数，其实这个函数除了FindAll方法要用外，别的地方都几乎很少使用到它，你还不得不给它起个名字。如果程序中有多处需要调用FindAll方法，或者类似的情况，那么整个程序也就会出现一大批“只有一个地方使用”的函数，使得代码难于阅读和维护。

由于存在这样的问题，C# 2.0引入了匿名方法。开发人员在实现方法的时候，只需要给出方法的参数列表（甚至也可以不给）以及方法具体实现，而不需要关心方法的返回值，更不必给方法起名字。最关键的是，只在需要的地方定义匿名方法，保证了代码的简洁。

匿名方法只在需要的地方定义，定义的时候，使用delegate关键字，后接参数列表，然后跟上用一对花括号包括起来的函数体即可。上面的代码可以重构成下面的形式：

```c#
class Program   
{   
    static void Main(string[] args)   
    {   
        List<string> names = new List<string>();   
        names.Add("Sunny Chen");   
        names.Add("Kitty Wang");   
        names.Add("Sunny Crystal");   
  
        List<string> found = names.FindAll(   
            delegate(string name)   
            {   
                return name.StartsWith("sunny",   
                    StringComparison.OrdinalIgnoreCase);   
            });   
  
        if (found != null)   
        {   
            foreach (string str in found)   
                Console.WriteLine(str);   
        }   
    }   
  
    //static bool NameMatches(string name)   
    //{   
    //    return name.StartsWith("sunny",    
    //        StringComparison.OrdinalIgnoreCase);   
    //}   
}   
```

此时，我们完全不需要NameMatches方法了，直接将匿名方法作为参数传递给FindAll方法。其实匿名方法本身还是有名字的，只是我们并不关心它究竟该取什么名字，因而.NET帮我们随便取了个名字罢了。

匿名方法在C#中应用十分广泛，因为委托作为函数参数是件非常平常的事情。在定义简单的事件处理过程时，我们同样可以使用匿名方法。比如：

```c#
ServiceHost host = new ServiceHost(typeof(FileTransferImpl));   
host.Opened += delegate(object sender, EventArgs e)   
{   
    Console.WriteLine("Service Opened.");   
}; 
```


匿名方法可以很方便地使用本地变量，这与单独定义的命名方法相比，能够简化编程。比如上文的例子中，假如Main函数里面定义了一个整型本地变量（局部变量）number，那么可以在delegate (string name)这一匿名方法定义中使用number变量。

上文提到，在定义匿名方法的时候，连参数列表都可以省略。因为编译器可以根据委托的签名来确定函数的签名，然后只要再给函数起个名字就可以了。下面的代码演示了这种使用方式：

```c#
delegate void IntDelegate(int x);   
```

```c#
// 带参数的定义方式   
IntDelegate d2 = delegate(int p) { Console.WriteLine(p); };   
// 不带参数的定义方式（当然也没带返回值）   
IntDelegate d3 = delegate { Console.WriteLine("Hello."); };   
```

在使用不带参数和返回值的匿名方法定义时，需要注意以下两点：

如果在你的匿名方法中需要对参数进行处理，那么你不能使用不定义参数列表的声明方式。也就是在定义匿名方法的时候，需要给出参数列表
不带参数和返回值的匿名方法，可以被具有任何形式签名的委托所指代
上述第一点显而易见，因为你没有定义参数列表，也就没有办法使用参数；要说明第二点，我们可以看下面的代码：

```c#
class Program   
{   
    delegate void IntDelegate(int x);   
    delegate void StringDelegate(string y);   
  
    static void Output(IntDelegate id)   
    {   
    }   
  
    static void Output(StringDelegate sd)   
    {   
    }   
  
    static void Main(string[] args)   
    {   
        /*  
         * ERROR: The call is ambiguous between  
         *        Output(IntDelegate)  
         *              and  
         *        Output(StringDelegate)  
         */  
        Output(delegate { });   
    }   
}   
```

上面的代码没法编译通过，因为编译器不知道应该将delegate { }这一匿名方法还原为由IntDelegate指代的函数，还是还原为由StringDelegate指代的函数。此时只能显式给定参数列表，以便让编译器知道，我们究竟是想调用哪个Output函数。