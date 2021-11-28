---
title: "C#中的 explicit 和 implicit 关键字"
date: 2020-01-21T12:50:02+08:00
draft: false
tags: ["C#", "Code", ".NET"]
categories: ["C#代码"]
series: [""]
---

## C#中的 explicit 和 implicit 关键字

简单记录一下。

#### explicit

explicit 用于声明必须使用强制转换来调用的用户定义的类型转换运算符。

例如如下类型：

```c#
public class StringCombine
{
    private readonly string _firstString;
    private readonly string _secondString;

    public StringCombine(string firstString, string secondString)
    {
        this._firstString = firstString;
        this._secondString = secondString;
    }

    public string Combine()
    {
        return $"{_firstString} {_secondString}";
    }
}
```

可以看到StringCombine类中有一个Combine()方法，用于将该类中的两个私有成有组合后输出：

```c#
StringCombine stringCombine = new StringCombine("Hello", "world!");
string newString = stringCombine.Combine();
Console.WriteLine(newString);
```

如上代码将会输出`Hello world!`。

现在我们在StringCombine类中添加一段代码：

```c#
public static explicit operator string(StringCombine stringCombine)
{
    return stringCombine.Combine();
}
```

这时我们稍稍修改一下上面的代码：
```c#
StringCombine stringCombine = new StringCombine("Hello", "world!");
string newString = (string)stringCombine;
Console.WriteLine(newString);
```

依然可以获得`Hello world!`，这里就将StringCombine类显式转换成了string类型。

#### implicit

implicit 作用与 explicit相似，只是 implicit 实现的是隐式转换。我们将上述代码中的 explicit 关键字 替换为 implicit 后，我们就可以将显式转换`(string)`去掉了：

```c#
StringCombine stringCombine = new StringCombine("Hello", "world!");
string newString = stringCombine;
Console.WriteLine(newString);
```

#### 最后

explicit 和 implicit 都可以起到转换类型的作用，区别仅仅是显式或隐式。

需要注意的是，因为隐式转换容易让人忽略，所以隐式转换运算符应当从不引发异常并且从不丢失信息，否则应当使用 explicit 来显式转换。