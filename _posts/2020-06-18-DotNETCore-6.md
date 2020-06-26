---
layout: post
title: '.NET Core前后端分离(5) - IOC(控制反转)与DI(依赖注入)'
date: 2020-06-20
author: HANABI
color: rgb(91,47,107)
tags: [.NET Core]
---

> 准备好了所有的层，如何在各个*Service*层中调用*Repository*层的接口，并控制调用的*Repository*层的生命周期呢，之前写在各构造函数中的参数是如何获得的呢，让我们来了解IoC(Inversion of Control)以及DI(Dependence Injection)这两个概念

## IoC(控制反转)思想

> 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递(注入)给它。

### 为什么需要控制反转

**先从工厂模式开始**

我们这里来写一段代码，定义一个小猫接口类，用几种不同的小猫类来实现这个接口，并且在调用程序用，动态的根据一个字符串的值，决定要用定义哪个小猫类，并且调用其发出叫声的函数

首先定义小猫接口`ICat`，其包含一个猫叫函数：

```c#
public interface ICat
{
    void Meow();
}
```

接着定义`Cats`类，里面包含三种不同类型的猫，并且都实现了`ICat`接口：

```c#
using System;

public class Cats
{
    public class BlackCat : ICat
    {
        public void Meow()
        {
            Console.WriteLine($"I'm {nameof(BlackCat)}");
        }
    }

    public class WhiteCat : ICat
    {
        public void Meow()
        {
            Console.WriteLine($"I'm {nameof(WhiteCat)}");
        }
    }

    public class PinkCat : ICat
    {
        public void Meow()
        {
            Console.WriteLine($"I'm {nameof(PinkCat)}");
        }
    }
}
```

最后，我们对其进行调用，调用的代码是这样写的

```c#
class Program
{
    static void Main(string[] args)
    {
        var strCatName = "BlackCat";

        ICat cat = strCatName switch
        {
            "BlackCat" => new Cats.BlackCat(),
            "WhiteCat" => new Cats.WhiteCat(),
            "PinkCat" => new Cats.PinkCat(),
            _ => null
        };

        cat.Meow();
    }
}
```

可以看到，此时采用的是在调用的地方来对具体对象进行初始化，这样做的缺点是什么呢，显而易见，这里的代码不符合*对修改关闭*的设计原则，虽然使用了接口，但是还是在实现业务功能的代码中初始化了好几种具体的类，如果需要增加或删除任何猫咪的种类，将不得不对这一段代码做出调整，并且如果其它地方需要使用的话，不具有可复用性

所以，我们创建一个新的类，将这个根据字符串获取具体类的方法放进去

```c#
public static class CatFactory
{
    public static ICat CreateCat(string strCatName)
    {
        ICat cat = strCatName switch
        {
            "BlackCat" => new Cats.BlackCat(),
            "WhiteCat" => new Cats.WhiteCat(),
            "PinkCat" => new Cats.PinkCat(),
            _ => null
        };

        return cat;
    }
}
```

之后，再对其进行调用

```c#
class Program
{
    static void Main(string[] args)
    {
        var strCatName = "BlackCat";
        ICat cat = CatFactory.CreateCat(strCatName); 
        cat.Meow();
    }
}
```

在这里，*工厂*的概念就产生了，可以看到，我们无论在哪里需要根据字符串获取到具体的小猫类时，都可以使用这个静态的工厂类，(注意，这里实现的只是一个简单的工厂类，不是完整的工厂模式，不过可以从中体会到工厂模式的思想)

**体会从最开始的模式到工厂模式的变化**

对于调用者来说，最开始的模式中，为了得到一个具体的小猫类，我们要需要自己描述小猫的外观，自己定义产生小猫的方法，最后才得到一个心仪的小猫类。而到了工厂模式，对于调用者来说，则只需要描述小猫的外观，由工厂那边代为处理，最后就能得到自己想要的结果，获取小猫的具体过程，不需要自己操心。

在前后两种模式的变化中，可以看到我们获取小猫的过程的**控制权已经发生了转移**，可以理解为交给了一个专业的工厂去处理这个过程，即使我们仍然需要通过调用工厂类，来达到我们的目的，但是这样做，已经让代码耦合性降低了，至于工厂类的出现让代码的职责划分得更加明确，加强了通用性就更加不必说了。

**如何达到真正的控制反转呢**

来看一段项目中使用了依赖注入中的构造函数注入的代码：

```c#
private readonly SqlSugarClient _db;

public BaseRepository(IUnitOfWork unitOfWork)
{
    _db = unitOfWork.GetDbClient();
}

/// <summary>
/// 写入实体数据
/// </summary>
/// <param name="model">实体类</param>
/// <returns></returns>
public async Task<int> Add(TEntity model)
{
    var insert = _db.Insertable(model);
    return await insert.ExecuteReturnIdentityAsync();
}
```

在使用这个*BaseRepository*类的时候，我们不需要手动去初始化*unitOfWork*，而是在代码走到其构造函数之前，已经得到了*unitOfWork*，进而得到*_db*来与数据库交互

是不是非常不可思议，就好像是小说里面的人物，心里想着一把兵器，兵器就自己出现在他手上。在类的内部函数中，没有任何一个对于要使用的类的实例化过程，兵器不是自己造的，也不是在要用到的时候委托别人去造的，更像是有人知道他需要这把兵器，就直接把这把兵器给他了。兵器的存在早于使用者自身的存在，这才是真正的，达到了控制反转

## DI(依赖注入)

通过上文我们也知道了，依赖注入是实现控制反转的一种方式，上文也展示了其使用的效果，但是描述得非常玄幻，那么实际上，依赖注入是怎么实现的呢

其实，在介绍控制反转时开头引用的那段概念，已经可以很好的说明依赖注入的实现方式了，即：
*通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递(注入)给它。*

**在我们.NET Core Web框架中，是如何实现这一点的呢**，这里我们介绍*.NET Core*中两种依赖注入的方式

1.	通过调用原生框架自带的依赖注入方法

上面提到了，依赖注入是由一个调控系统内所有对象的外界实体来完成的，这个实体在*.NET Core*框架中以容器的形式被提供，首先，查看*Program.cs*中的*CreateHostBuilder*函数

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

这里调用了*UseStartup*，然后我们看到*Startup.cs*中的*ConfigureServices*函数，这个函数的官方介绍是*This method gets called by the runtime. Use this method to add services to the container.*，可以看出，这个方法是在运行时被调用，用来往容器中添加服务

没错，这里提到的容器，就是用来实现我们依赖注入的容器，这里要做的，就是在构建这个容器的时候，往里面添加我们需要用到的被注入的类就行了，我们直接用一段代码来说明

2.	使用*Autofac*实现依赖注入
