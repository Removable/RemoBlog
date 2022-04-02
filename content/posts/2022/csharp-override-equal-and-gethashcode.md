---
title: "C# 重写（override）Equals 和 GetHashCode"
date: 2022-04-02T10:39:11+08:00
draft: false
tags: [".Net", "C#", "Code"]
categories: ["C#代码"]
series: [""]
---

## Equals 和 GetHashCode

Equals 每个实现都必须遵循以下约定:

- **自反性(Reflexive):** x.Equals(x) 必须返回 true
- **对称性(Symmetric):** x.Equals(y) 为 true 时，y.Equals(x) 也为 true
- **传递性(Transitive):** 对于任何非 null 的应用值 x, y 和 z，如果 x.Equals(y) 返回 true，并且 y.Equals(z) 也返回true，那么 x.Equals(z) 必须返回 true
- **一致性(Consistence):** 如果多次将对象与另一个对象比较，结果始终相同。只要未修改 x 和 y 的应用对象，x.Equals(y) 连续调用 x.Equals(y) 返回相同的值
- **非null(Non-null):** 如果 x 不是 null，y 为 null，则 x.Equals(y) 必须为 false

GetHashCode:

- 两个相等对象根据 equals 方法比较时相等，那么这两个对象中任意一个对象的 GetHashCode 方法都必须产生同样的整数
- 在我们未对对象进行修改时,多次调用 GetHashCode 使用返回同一个整数。在同一个应用程序中多次执行，每次执行返回的整数可以不一致
- 如果两个对象根据 Equals 方法比较不相等时，那么调用这两个对象中任意一个对象的 GetHashCode 方法，不一同的整数。但不同的对象，产生不同整数，有可能提高散列表的性能

> 以上内容摘自@冯辉 的博客： https://www.cnblogs.com/yyfh/p/12245916.html

## 默认情况

我们定义一个如下的 Class：

```c#
public class Example
{
	public string? Str { get; set; }
	public int Number { get; set; }
}
```

在默认情况下进行比较，我们会得到如下结果：

```c#
var e1 = new Example { Str = "Str", Number = 0 };
var e2 = new Example { Str = "Str", Number = 0 };
var e3 = e2;
Console.WriteLine(e1.Equals(e1)); //True
Console.WriteLine(e1.Equals(e2)); //False
Console.WriteLine(e3.Equals(e2)); //True

var hashSet = new HashSet<Example>();
Console.WriteLine(hashSet.Add(e1)); //True
Console.WriteLine(hashSet.Add(e1)); //False
Console.WriteLine(hashSet.Add(e2)); //True
Console.WriteLine(hashSet.Add(e3)); //False
```

## 重写 Equals 方法

如果我们需要让 Str 字段和 Number 字段相等时，就判断两个 Example 类相等，就要对 Equals 方法进行重写（override）:

```c#
public class Example
{
    public string? Str { get; set; }
    public int Number { get; set; }

    public override bool Equals(object? obj)
    {
        if (obj is not Example example) return false;
        return this.Number.Equals(example.Number) && string.Equals(this.Str, example.Str);
    }
}
```

这时再运行上面的比较：

```c#
var e1 = new Example { Str = "Str", Number = 0 };
var e2 = new Example { Str = "Str", Number = 0 };
var e3 = e2;
Console.WriteLine(e1.Equals(e1)); //True
Console.WriteLine(e1.Equals(e2)); //True
Console.WriteLine(e3.Equals(e2)); //True

var hashSet = new HashSet<Example>();
Console.WriteLine(hashSet.Add(e1)); //True
Console.WriteLine(hashSet.Add(e1)); //False
Console.WriteLine(hashSet.Add(e2)); //True
Console.WriteLine(hashSet.Add(e3)); //False
```

## 重写 GetHashCode 方法

现在`e1.Equals(e2)`的结果已经是`True`了，但是 HashSet 在添加了 e1 之后，依然可以添加 e2；并且你会发现编译器发出了一个警告：`'Example' overrides Object.Equals(object o) but does not override Object.GetHashCode()`。所以我们还需要重写 GetHashCode 方法。

在 Example 类中添加一个 GetHashCode 重写：

```c#
public override int GetHashCode()
{
    if (this.Str == null)
        return 0;
    return Str.GetHashCode() * Number.GetHashCode();
}
```

输出就会变成：

```c#
var hashSet = new HashSet<Example>();
Console.WriteLine(hashSet.Add(e1)); //True
Console.WriteLine(hashSet.Add(e1)); //False
Console.WriteLine(hashSet.Add(e2)); //False
Console.WriteLine(hashSet.Add(e3)); //False
```

> 有需要的话还可以重载（overloading）== 操作符，实现 `e1 == e2` 为 `True`

