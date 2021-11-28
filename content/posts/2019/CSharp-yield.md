---
title: "【转】C#基础小知识之 yield 关键字"
date: 2019-08-16T12:50:02+08:00
draft: false
tags: ["C#", "Code", ".NET"]
categories: ["C#代码"]
series: [""]
---

> 转自博客：[一天两天三天](http://www.cnblogs.com/santian/p/4389675.html "一天两天三天")

对于yield关键字我们首先看一下 msdn 的解释：

如果你在语句中使用 yield 关键字，则意味着它在其中出现的方法、运算符或 get 访问器是迭代器。 通过使用 yield 定义迭代器，可在实现自定义集合类型的 [IEnumerable](https://msdn.microsoft.com/zh-cn/library/system.collections.ienumerable.aspx) 和 [IEnumerator](https://msdn.microsoft.com/zh-cn/library/system.collections.ienumerator.aspx) 模式时无需其他显式类（保留枚举状态的类，有关示例，请参阅 [IEnumerator](https://msdn.microsoft.com/zh-cn/library/78dfe2yb.aspx)）。

### yield是一个语法糖

看 msdn 的解释总是让人感觉生硬难懂。其实 yield 关键字很好理解。首先我们对于性质有个了解。yield 是一个语法糖。既然 yield 是在C#中的一个语法糖，那么就说明yield是对一种复杂行为的简化，就是将一段代码简化为一种简单的形式，方便我们程序员使用。

那么yield到底是对什么行为的简化。我们首先来看一下yield的使用场景。

还是来看msdn上的例子。

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApplication2
{
    class Program
    {
        static void Main(string[] args)
        {
         
            foreach (int i in Power(2, 8, ""))
            {
                Console.Write("{0} ", i);
            }
            Console.ReadKey();
        }


        public static IEnumerable<int> Power(int number, int exponent, string s)
        {
            int result = 1;

            for (int i = 0; i < exponent; i++)
            {
                result = result * number;
                yield return result;
            }
            yield return 3;
            yield return 4;
            yield return 5;
        }

    }
}
```

这是msdn上yield的一种使用场景。

我们首先看一下下面的Power方法。该静态方法返回一个IEnumerablel<int>类型的参数。按照我们平常的做法。应该对数据执行一定操作，然后return一个IEnumerablel<int>类型的参数。我们把Power方法改造如下：

```C#
 public static IEnumerable<int> Power(int number, int exponent, string s)
        {
            int result = 1;
            //接口不能实例化，我们这儿new一个实现了IEnumerable接口的List
            IEnumerable<int> example = new List<int>();
            for (int i = 0; i < exponent; i++)
            {
                result = result * number;
                (example as List<int>).Add(result);
            }
            return example;
        }
```

这是我们平常的思路。但是这样做就有个问题。这儿要new一个List,或者任何实现了IEnumerable接口的类型。这样也太麻烦了吧。要知道IEnumerable是一个常用的返回类型。每次使用都要new一个LIst,或者其他实现了该接口的类型。与其使用其他类型，不如我们自己定制一个实现了IEnumerable接口专门用来返回IEnumerable类型的类型。我们自己定制也很麻烦。所以微软帮我们定制好了。这个类是什么，那就是yield关键字这个语法糖。

### 语法糖的实现（实现IEnumerable<T>接口的类）

我们来看一下yield的反编译代码。

```c#
namespace ConsoleApplication2
{
    using System;
    using System.Collections;
    using System.Collections.Generic;
    using System.Diagnostics;
    using System.Runtime.CompilerServices;

    internal class Program
    {
        private static void Main(string[] args)
        {
            IEnumerable<int> enumerable = Power(2, 8);
            Console.WriteLine("Begin to iterate the collection.");
            foreach (int num in Power(2, 8))
            {
                Console.Write("{0} ", num);
            }
            Console.ReadKey();
        }

        public static IEnumerable<int> Power(int number, int exponent)
        {
            <Power>d__0 d__ = new <Power>d__0(-2);
            d__.<>3__number = number;
            d__.<>3__exponent = exponent;
            return d__;
        }

        [CompilerGenerated]
        private sealed class <Power>d__0 : IEnumerable<int>, IEnumerable, IEnumerator<int>, IEnumerator, IDisposable
        {
            private int <>1__state;
            private int <>2__current;
            public int <>3__exponent;
            public int <>3__number;
            private int <>l__initialThreadId;
            public int <result>5__1;
            public int exponent;
            public int number;

            [DebuggerHidden]
            public <Power>d__0(int <>1__state)
            {
                this.<>1__state = <>1__state;
                this.<>l__initialThreadId = Environment.CurrentManagedThreadId;
            }

            private bool MoveNext()
            {
                switch (this.<>1__state)
                {
                    case 0:
                        this.<>1__state = -1;
                        this.<result>5__1 = 1;
                        Console.WriteLine("Begin to invoke GetItems() method");
                        this.<>2__current = 3;
                        this.<>1__state = 1;
                        return true;

                    case 1:
                        this.<>1__state = -1;
                        this.<>2__current = 4;
                        this.<>1__state = 2;
                        return true;

                    case 2:
                        this.<>1__state = -1;
                        this.<>2__current = 5;
                        this.<>1__state = 3;
                        return true;

                    case 3:
                        this.<>1__state = -1;
                        break;
                }
                return false;
            }

            [DebuggerHidden]
            IEnumerator<int> IEnumerable<int>.GetEnumerator()
            {
                Program.<Power>d__0 d__;
                if ((Environment.CurrentManagedThreadId == this.<>l__initialThreadId) && (this.<>1__state == -2))
                {
                    this.<>1__state = 0;
                    d__ = this;
                }
                else
                {
                    d__ = new Program.<Power>d__0(0);
                }
                d__.number = this.<>3__number;
                d__.exponent = this.<>3__exponent;
                return d__;
            }

            [DebuggerHidden]
            IEnumerator IEnumerable.GetEnumerator()
            {
                return this.System.Collections.Generic.IEnumerable<System.Int32>.GetEnumerator();
            }

            [DebuggerHidden]
            void IEnumerator.Reset()
            {
                throw new NotSupportedException();
            }

            void IDisposable.Dispose()
            {
            }

            int IEnumerator<int>.Current
            {
                [DebuggerHidden]
                get
                {
                    return this.<>2__current;
                }
            }

            object IEnumerator.Current
            {
                [DebuggerHidden]
                get
                {
                    return this.<>2__current;
                }
            }
        }
    }
}
```

反编译代码有三部分，其中程序的入口点   `private static void Main(string[] args) Power` 方法  `public static IEnumerable<int> Power(int number, int exponent)` 和我们自己写的代码一样，但是反编译代码中还多了一个密封类

  `private sealed class <Power>d__0 : IEnumerable<int>, IEnumerable, IEnumerator<int>, IEnumerator, IDisposable`

现在情况已经明了了。yield这个语法糖实现了一个实现 IEnumerable<int>接口的类来返回我们需要到 IEnumerable<int>类型的数据。

我们再看一下反编译后的Power方法

```c#
 public static IEnumerable<int> Power(int number, int exponent)
        {
            <Power>d__0 d__ = new <Power>d__0(-2);
            d__.<>3__number = number;
            d__.<>3__exponent = exponent;
            return d__;
        }
```

此时就确认，的确是使用了实现枚举接口的类来返回我们需要的数据类型。

每次yield return <expression>;就会像该类的实例中添加 一条数据。当yield break;的时候停止添加。

至此yield的用法就很清楚了。当我们需要返回IEnumerable类型的时候，直接yield返回数据就可以了。也不用new一个list,或其他类型。所以yield是一个典型的语法糖。

### yield使用中的特殊情况

我们看到编译器将我们yield的数据添加到了一个集合中。Power方法在编译器中实例化了一个实现枚举接口的类型。但是我们在Power方法中写一些方法，编译器会如何处理

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
namespace ConsoleApplication2
{
    class Program
    {
        static void Main(string[] args)
        {
            //这儿调用了方法。
            var test = Power(2, 8, "");
            Console.WriteLine("Begin to iterate the collection.");
            //Display powers of 2 up to the exponent of 8:
            foreach (int i in Power(2, 8, ""))
            {
                Console.Write("{0} ", i);
            }
            Console.ReadKey();
        }
        public static IEnumerable<int> Power(int number, int exponent, string s)
        {
            int result = 1;
            if (string.IsNullOrEmpty(s))
            {
                //throw new Exception("这是一个异常");
                Console.WriteLine("Begin to invoke GetItems() method");
            }

            for (int i = 0; i < exponent; i++)
            {
                result = result * number;
                yield return result;
            }
            yield return 3;
            yield return 4;
            yield return 5;
        }
    }
}
```

按照我们的理解当我们 `var test = Power(2, 8, "");` 的时候确实调用了Power方法。此时应该程序打印 `Console.WriteLine("Begin to invoke GetItems() method");` 然后继续执行 `Console.WriteLine("Begin to iterate the collection.");` 方法。所以打印顺序应该是

Begin to invoke GetItems() method

Begin to iterate the collection.

但是我们运行的时候却发现

![img](https://i.loli.net/2019/08/06/kRnJjQ7UHCW698i.png)

打印顺序和我们想象的不同。此时还是去看反编译代码。

```c#
namespace ConsoleApplication2
{
    using System;
    using System.Collections;
    using System.Collections.Generic;
    using System.Diagnostics;
    using System.Runtime.CompilerServices;

    internal class Program
    {
        private static void Main(string[] args)
        {
            IEnumerable<int> enumerable = Power(2, 8, "");
            Console.WriteLine("Begin to iterate the collection.");
            foreach (int num in Power(2, 8, ""))
            {
                Console.Write("{0} ", num);
            }
            Console.ReadKey();
        }

        public static IEnumerable<int> Power(int number, int exponent, string s)
        {
            <Power>d__0 d__ = new <Power>d__0(-2);
            d__.<>3__number = number;
            d__.<>3__exponent = exponent;
            d__.<>3__s = s;
            return d__;
        }

        [CompilerGenerated]
        private sealed class <Power>d__0 : IEnumerable<int>, IEnumerable, IEnumerator<int>, IEnumerator, IDisposable
        {
            private int <>1__state;
            private int <>2__current;
            public int <>3__exponent;
            public int <>3__number;
            public string <>3__s;
            private int <>l__initialThreadId;
            public int <i>5__2;
            public int <result>5__1;
            public int exponent;
            public int number;
            public string s;

            [DebuggerHidden]
            public <Power>d__0(int <>1__state)
            {
                this.<>1__state = <>1__state;
                this.<>l__initialThreadId = Environment.CurrentManagedThreadId;
            }

            private bool MoveNext()
            {
                switch (this.<>1__state)
                {
                    case 0:
                        this.<>1__state = -1;
                        this.<result>5__1 = 1;
                        if (string.IsNullOrEmpty(this.s))
                        {
                            Console.WriteLine("Begin to invoke GetItems() method");
                        }
                        this.<i>5__2 = 0;
                        while (this.<i>5__2 < this.exponent)
                        {
                            this.<result>5__1 *= this.number;
                            this.<>2__current = this.<result>5__1;
                            this.<>1__state = 1;
                            return true;
                        Label_009D:
                            this.<>1__state = -1;
                            this.<i>5__2++;
                        }
                        this.<>2__current = 3;
                        this.<>1__state = 2;
                        return true;

                    case 1:
                        goto Label_009D;

                    case 2:
                        this.<>1__state = -1;
                        this.<>2__current = 4;
                        this.<>1__state = 3;
                        return true;

                    case 3:
                        this.<>1__state = -1;
                        this.<>2__current = 5;
                        this.<>1__state = 4;
                        return true;

                    case 4:
                        this.<>1__state = -1;
                        break;
                }
                return false;
            }

            [DebuggerHidden]
            IEnumerator<int> IEnumerable<int>.GetEnumerator()
            {
                Program.<Power>d__0 d__;
                if ((Environment.CurrentManagedThreadId == this.<>l__initialThreadId) && (this.<>1__state == -2))
                {
                    this.<>1__state = 0;
                    d__ = this;
                }
                else
                {
                    d__ = new Program.<Power>d__0(0);
                }
                d__.number = this.<>3__number;
                d__.exponent = this.<>3__exponent;
                d__.s = this.<>3__s;
                return d__;
            }

            [DebuggerHidden]
            IEnumerator IEnumerable.GetEnumerator()
            {
                return this.System.Collections.Generic.IEnumerable<System.Int32>.GetEnumerator();
            }

            [DebuggerHidden]
            void IEnumerator.Reset()
            {
                throw new NotSupportedException();
            }

            void IDisposable.Dispose()
            {
            }

            int IEnumerator<int>.Current
            {
                [DebuggerHidden]
                get
                {
                    return this.<>2__current;
                }
            }

            object IEnumerator.Current
            {
                [DebuggerHidden]
                get
                {
                    return this.<>2__current;
                }
            }
        }
    }
}
```

我们看到Power方法

```c#
 public static IEnumerable<int> Power(int number, int exponent, string s)
        {
            <Power>d__0 d__ = new <Power>d__0(-2);
            d__.<>3__number = number;
            d__.<>3__exponent = exponent;
            d__.<>3__s = s;
            return d__;
        }
```

还是还我们没有加打印方法之前一样。我们的打印方法并没有出现在Power方法中，而是被封装进了实现枚举接口的类方法  private bool MoveNext()中。所以方法不会立即被执行，而是在我们使用数据的时候被执行。如果对此机制不了解，就容易出现另外一些意想不到的问题。例如在Power方法中添加一些验证程序，如果不符合条件就抛出一个异常。这样的异常检查不会被执行。只有我们使用数据的时候才会执行。这样就失去了检查数据的意义。

具体的例子可以看[Artech博主的文章](http://www.cnblogs.com/artech/archive/2013/04/12/yield-in-wcf.html)

另外使用yield还有一些注意事项：

你不能在具有以下特点的方法中包含 yield return 或 yield break 语句：

- 匿名方法。 有关详细信息，请参阅[匿名方法（C# 编程指南）](https://msdn.microsoft.com/zh-cn/library/0yw3tz5k.aspx)。
- 包含不安全的块的方法。 有关详细信息，请参阅[unsafe（C# 参考）](https://msdn.microsoft.com/zh-cn/library/chfa2zb8.aspx)。

异常处理

不能将 yield return 语句置于 try-catch 块中。 可将 yield return 语句置于 try-finally 语句的 try 块中。

yield break 语句可以位于 try 块或 catch 块，但不能位于 finally 块。

如果 foreach 主体（在迭代器方法之外）引发异常，则将执行迭代器方法中的 finally 块。