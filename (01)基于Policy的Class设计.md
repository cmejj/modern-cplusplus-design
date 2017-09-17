# 基于Policy的Class设计

这一章将介绍所谓 policies 和 policy classes，它们是一种重要的 classes 设计技术，能够增加程序库的弹性并提高复用性，这正是LOk的目标所在。

> 简言之，具备复杂功能的 policy-based class 由许多小型 classes （称为 policies ）组成，每一个这样的小型 class 都只负责单纯如行为或结构的某一方面（ behavioral or structural aspect ）。一如名称所示，一个 policy 会针对特定主题建立一个接口，在`遵循 policy 接口`的前提下，你可以采用任何适当的方法来实作 policies 。

## 1.1 软件设计的多样性

所谓软件设计就是解域空间（ solution space ）中的一道选择题

> 软件工程，也许比其他工程展现出更丰富的多样性。你可以采用多种正确做法完成一件事，而对与错之间存在无尽的细微差别。每一个新选择都会开启一个新世界。一旦你选择了一个解决方案，便会有一大堆变化随之涌现，而且会在每个阶段中持续不停地涌现，大至系统架构，小至程序片段.

编程（ programming ）才华可能在年轻时就展现出来，而软件设计（ software design ）才华却常需要更多的时间才能成熟。

## 1.2 全功能型（ Do-It-All. ）接口的失败

在一个“无所不包”的大型接口上厉行如此的 constraints 是很困难的。

> 一般来说，一旦你选择了某一组 design constraints，大型接口内就只有某些子集能够维持其有效语义，程序库（ library ）的运用向来在“语法有效”和“语义有效”之间存在着缝隙，程序员可能写出愈来愈多的概念，它们虽然“语法有效”，却“语义无效”。

如果程序库将不同的设计实作为各个小型 classes，每个 class 代表一个特定的罐装解法，如何？

> 这种做法的问题是会产生大量设计组合，面对这么多的潜在组合（近乎指数爬升），千万别企图使用暴力枚举法。

设计是为了厉行 constraints（约束条件、规范）。因此，以设计为目标的程序库必须帮助使用者精巧完成设计，以实现使用者自己的 constraints，而不是实现预先定义好的 constraints。

## 1.3 多重继承(Multiple Inheritance )是救世主?

这样天真的设计方式其实无法运作。

问题

- 1．关于技术（Mechanics）。

- 2．关于型别信息（Type information）。

- 3．关于状态处理（State manipulation）。

虽然本质上是组合（combinatorial）但多重继承无法单独解决设计吋的多样性选择。

## 1.4 Templates 带来曙光

templates 是一种很适合“组合各种行为”的机制，主要因为它们是“依赖使用者提供的型别信息”,并且“在编译期才产生”的代码。

问题

- 1.你无法特化结构。

- 2.成员函数的特化并不能“依理扩张”。

- 3.程序库撰写者不能够提供多笔缺省值。

现在让我们比较一下多重继承和 templates 之间的缺点，有趣的是两者互补，多重继承欠缺技术（Mechanics） templates 有丰富的技术。多重继承缺乏型别信息，而那东西在 templates 里大量存在。Templates 的特化无法扩张（scales），多重继承却很容易扩张。你只能为 templates 成员函数写一份缺省版本，但你可以写数量无限的 base classes。

根据以上分析，如果我们将 templates 和多重继承组合起来，将会产生非常具弹性的设备，
（device），应该很适合用来产生程序库中的“设计元素”（design elements)。

## 1.5 Policies 和 Policy Classes

Policies 和 Policy Classes 有助于我们设计出安全、有效率且具高度弹性的“设计元素”。所谓 policy ，乃用来定义一个 class 或 class templates 的接口，该接口由下列项目之一或全部组成：内隐型别定义（inner type definition）、成员函数和成员变量。

Policies 也被其他人用于 traits （ Alexandrescu 2000a ），不同的是后者比较重视行为而非型别。

Policies 也让人联想到设计模式 Strategy （Gamma et al. 1995），只不过 policies 吃紧于编译期所谓（compile-time bound)。

``` C++
template <class T>
struct OpNewCreator
{
    static T* Create()
    {
        return new T;
    }
}

template <class T>
struct MallocCreator
{
    static T* Create()
    {
        void* buf = std::mallOC(sizeof(T));
        if(!buf) return 0;
        return new(buf) T;
    }
}

template <class T>
struct PrototypeCreator
{
    PrototypeCreator(T* pObj = 0)
        : pPrototype_(pObj)
    {}

    T* Create()
    {
        return pPrototype_ ? pPrototype_->Clone() : 0;
    }

    T* GetPrototype() { return pPrototype_; }
    void SetPrototype(T* pObj) { pPrototype_ = pObj; }

private:
    T* pPrototype_;
}
```

这里有一个重要观念： **policies 接口和般传统的 classes 接口（纯虚函数集）不同，它比较松散，因为 policies 是语法导向（syntax oriented）而非标记导向（signature oriented）**。

``` c++
// Library code
template <class CreationPolicy>
class WidgetManager : public CreationPolicy
{ ... };

//Application code
typedef WidgetManager< OpNewCreator<Widget> > MyWidgetMgr;
```

### 1.5.1 运用 Template Template 参数实作 Policy Classes

``` c++
// Library code
template <template <class Created> class CreationPolicy>
class WidgetManager : public CreationPolicy<Widget>
{ ... };

// 译注： Created 是 CreationPolicy 的参数， CreationPolicy 则是 WidgetManager 的参数
// Widget 已经写入上述程序中 所以使用时不需要再传一次给 Policy。
```

尽管露了脸，上述 Created 也并未对 WidgetManager 有任何贡献。你不能在 WidgetManager 中使用 Created，它只是 CreationPolicy（而非 WidgetManager ）的形式引数（formal argument），因此可以省略。

应用端现在只需在使用 WidgetManager 时提供 template 名称即可：

``` c++
//Application code
typedef WidgetManager<OpNewCreator> MyWidgetMgr;
```

搭配 policy class 使用“ template template 参数”，并不单纯只为了方便．有时候这种用法不可或缺，以便 host class 可藉由 tempaes 产生不同型别的对象。举个例子，假设 WidgetManager 想要以相同的生成策略产生一个 Gadget 对象，代码如下：

``` c++
// Library code
template <template <class> class CreationPolicy>
class WidgetManager : public CreationPolicy<Widget>
{
    ...
    void DoSomeThing()
    {
        Gadget* pW = CreationPolicy<Gadget>().Create();
    }
};
```

注意，policies 和虚函数有很大不同，虽然虚函数也提供类似效果 class 作者以基本的（primtive）虚函数来建立高端功能，并允许使用者改写这些基本虚函数的行为，然而如前所示， policies 因为有丰富的型别信息及静态连接等特性，所以是建立“设计元素”时的本质性东西，不正是“设计”指定了“执行前型别如何互相作用、你能够做什么、不能够做什么”的完整规则吗？

Policies 可以让你在型别安全（typesafe）的前提下藉由组合各个简单的需求来产出你的设计。此外，由
于编译期才将 host class 和其 policies 结合在一起，所以和手工打造的程序比较起来更加牢固并且更有效率。

当然，也由于 policies 的特质，它们不适用于动态连结和二进位接口，所以本质上 policies 和传统接口并不互相竞争。

### 1.5.2 运用 Template 成员函数实作 Policy Classes

另外一种使用“template template 参数”的情况是把 template 成员函数用来连接所需的简单类。也就是说，将 policy 实作为一般 class（“一般”是相对于 class template 而言）,但有一个或数个 templated members。

例如，我们可以重新定义先前的 Creator policy 成为一个non-template class，其内提供—个名为 Create＜T> 的 template 函数。如此一来， policy class 看起来像下面这个样子：

``` c++
struct OpNewCreator
{
    template <class T>
    static T* Create()
    {
        return new T;
    }
};
```

这种方式所定义并实作出来的 policy 对于旧式编译器有较佳兼容性。但从另一方面来说，这
样的 policy 难以讨论、定义、实作和运用。

# 1.6 更丰富的 policies

Creator policy 只指定了一个 Create 成员函数。然而 PrototypeCreator 却多定义了两个函数．分别为 GetPrototype 和 SetPrototype 。让我们来分析一下。

由于 WidgetManager 继承了 policy class，而且 GetPrototype 和 SetPrototype 是 public 成员，所以这两个函数便被加至 WidgetManager·并且可以直接被使用者取用。

然而 WidgetManager 只要求 Create；那是 WidgetManager 一切所需，也是用来保证它自己机能的唯一要求。不过使用者可以开发出更丰富的接口。

Prototype-based Creator policy class 的使用者可以写出下列代码：

``` c++
typedef WidgetManager<PrototypeCreator> MywidgetManager;
...
Widget* pPrototype = ...;
MywidgetManager mgs;
mgr.SetPrototype(pPrototype);
... use mgr ...
```

如果此后使用者决定采用―个不支持 prototypes 的生成策略,那么编译器会指出问题：prototype专属接口已经被用上了。这正是我们希望获得的坚固设计。

决定“哪个 policy 被使用”的是使用者而非程序库自身，和一般多重接口不同的是， policy 给予使用者一种能力，在型别安全（ typesafe ）的前提下扩增 host class 的功能。

# 1.7 Policy Classes 的析构函数

有一个关于建立 policy class 的重要细节。大部分情况下 host class 会以“ public 继承”方式从某些 policies 派生而来。因此，使用者可以将—个 host class 自动转为一个 policy class （译注：向上转型），并于稍后 delete 该指针。除非 policy class 定义了―个虚析构函数（virtual destructor ),否则 delete 一个指向 policy class 的指针，会产生不可预期的结果，如下所示：

``` c++
typedef WidgetManager<PrototypeCreator> MyWidgetMgr;
...
MywidgetManager wm;
PrototypeCreator<Widget>* pCreator = $wm; // dubious, but legal
delete pCreator; // compile fine, but has undefined behavior
```

然而如果为 policy 定义了一个虚析构函数，会妨碍 policy 的静态连结特性，也会影响执行效率。

许多 policies 并无任何数据成员，纯粹只规范行为。第―一个虚函数被加入后会为对象大小带来额外开销（译注：因为引入一份 vptr)，所以虚析构函数应该尽可能避免。

一个解法是，当 host class 自 policy class 派生时，采用 protected 继承或 private 继承。然而这样会失去丰富的 policies 特性（1.6节）。 policies 应该采用一个轻便而有效率的解法——定义一个 non-virtual protected 析构函数：

``` c++
struct OpNewCreator
{
    template <class T>
    static T* Create()
    {
        return new T;
    }

protected:
    ~OpNewCreator() {}
}
```

由于析构函数属于 protected 层级，所以只有派生而得的 classes 才可以摧毁这个 policy 对象。这样一来外界就不可能 delete 一个指向 policy class 的指针。而由于析构函数并非虚函数，所以不会有大小或速度上的额外开销。

## 1.8 通过不完全具现化而获得的选择性机能



