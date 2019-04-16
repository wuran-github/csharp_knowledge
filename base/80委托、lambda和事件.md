# 委托、lambda和事件

## 引用方法
- 委托是寻址方法的`.NET`版本。在C++中，函数指针只不过是一个指向内存位置的指针，他不是类型安全的。我们无法判断这个指针实际指向什么，像参数和返回类型更无从得知。

- 而委托完全不同；委托是类型安全的类，它定义了返回类型和参数的类型。委托类不仅包含对方法的引用，也可以包含对多个方法的引用。

- lambda表达式与委托直接相关。当参数是委托类型时，就可以使用lambda表达式实现委托引用的方法。

----

## 委托
- 当要把方法传送给其他方法时，就需要使用委托。
```
int i = int.Parse("999");
```

- 这是简单的传递参数。但是有时候我们需要给方法传递另一个方法。有时某个方法执行的操作并不是针对数据进行的，而是要对另一个方法进行调用。更麻烦的是，在编译时我们不知道第二个方法是什么，之恶个信息只能在运行时得到，所以需要把第二个方法作为参数传递给第一个方法。（有点像接口?

- 例如：
    1. 启动线程和任务——在C#中，可以告诉计算机并行运行某些新的执行序列，同时运行当前的任务。这种序列就称为线程，在一个基类Thread的实例上使用方法Start，就可以启动一个线程。

    - 如果要告诉计算机启动一个新的执行序列，就必须说明要在哪里启动该序列；必须为计算机提供开始启动的方法的细节，即Thread类的构造函数必须带有一个参数，该参数定义了线程调用的方法。（JAVA是通过接口实现的）

    2. 通用库类——许多库包含执行各种标准任务的代码。这些库通常可以自我包含，这样在编写库时，就会找到任务该如何执行。但是有时在任务中还包含子任务，只有使用该库的客户端代码才知道如何执行这些子任务。

    3. 事件——一般的思路是通知代码发生了什么事件。GUI编程主要处理事件。在引发事件时，运行库需要知道应执行哪个方法。


- 在进行面向对象编程时，几乎没有方法是孤立存在的，而是在调用方法前通常需要与类实例相关联。所以`.NET Framework`在语法上不允许使用这种直接方法。

- 如果要传递方法，就必须把方法的细节封装在一个新的对象类型中，即`委托`。

- 委托只是一个特殊类型的对象，其特殊之处在于，我们以前定义的所有对象都包含数据，而委托包含的只是一个或多个方法的地址。

### 声明委托

- 在C#中使用一个类需要两个步骤：
    1. 定义类
    2. 实例化

- 委托也同样。

- 首先必须定义要使用的委托，对于委托，定义它就是告诉编译器这种类型的委托表示哪种类型的方法。

- 然后，必须创建该委托的一个或多个实例。编译器在后台将创建表示该委托的一个类。声明委托的语法如下：
```
delegate void IntMethodInvoker(int x);
```

- 声明了一个委托 `IntMethodInvoker` ，并指定该委托的每个实例都可以包含一个方法的引用，该方法带有一个int参数，并返回void。理解委托的一个要点是他们的类型安全非常高，在定义委托时，必须给出他所表示的方法的签名和返回类型等全部细节。

- 理解委托的一种好方式是把委托视为给方法的签名和返回类型指定名称。

- 委托的语法类似于方法的定义，但没有方法主体，且定义的前面要加上关键字delegate。
- 因为定义委托基本上是定义一个新类，所以可以在定义类的任何相同地方定义委托。也就是说，可以在类的内部定义，也可以在外部定义，还可以在名称空间中把委托定义为顶层对象。

- 根据定义的可见性和委托的作用域，可以在委托的定义上应用任何常见的访问修饰符:private public protected等。
```
public delegate void IntMethodInvoker(int x);
internal delegate string GetAString();
class DelegateExample
{
    protected delegate double TwoLongsOp(long first, long second);
    private delegate int Max(int x, int y);

}
```

- 实际上，定义一个委托是指定义一个新类。委托实现为派生自基类`System.MulticastDelegate`的类，该类又派生自基类`System.Delegate`。C#编译器能识别这个类，会使用其委托语法。

```
//IL代码
.class public auto ansi sealed C9_DelegateAndEvent.DelegateNS.IntMethodInvoker
	extends [mscorlib]System.MulticastDelegate
```

- 定义好委托后，就可以创建他的一个实例，从而用该实例存储特定方法的细节。

### 使用委托

- 如下代码：
```
int x = 40;
GetAString get = new GetAString(x.ToString);
Debug.WriteLine(get());
//Debug.WriteLine(get.Invoke());
```
- 在这段代码中，实例化类型为GetAString的委托，并对他初始化，使其引用整形变量x的ToString方法。

- 在C#中，委托在语法上总是接收一个参数的构造函数，这个参数就是委托引用的方法。这个方法必须匹配最初定义委托时的签名。

- 在任何代码中，都应提供委托实例的名称，后的圆括号中应包含调用该委托中的方法时使用的任何等效参数。

- 实际上，给委托实例提供圆括号与调用委托类的Invoke()方法完全相同。因为get是委托类型的一个变量，所以C#编译器会用get.Invoke()代替get()。

- 在需要委托实例的每个位置可以只传送地址的名称，不需要使用`mew delegate`。这称为委托推断。只要编译器可以把委托实例解析为特定的类型，这个C#特性就是有效的。
```
get = x.ToString;
```
- C#编译器创建的代码是一样的。

- 注意，输入形式是x.ToString而不是x.ToString()。输入圆括号会调用一个方法，不输入的话则代表方法地址。

- 委托推断可以在需要委托实例的任何地方使用。委托推断也可以用于事件，因为事件基于委托。

- 委托的一特征是类型安全，可以确保被调用的方法的签名是正确的。但有趣的是，他们不关心在什么类型的对象上调用该方法，甚至不考虑该方法是静态还是实例方法。

- 给定委托的实例可以引用任何类型的任何对象上的实例方法或静态方法——只要方法的签名匹配委托的签名即可。

- 下面是一个静态方法的调用：
```
class DelegateExample
{
    public static string GetStr() => "Example";
}
static void DelegateStatic()
{
    GetAString get = DelegateExample.GetStr;
    Debug.WriteLine(get());
}
```

- 下面给出两个实例

### 简单的委托示例
- 下面是一个使用委托调用两个方法的例子：
```
delegate double MathOp(double value);
class MathOperations
{
    public static double MultiplyByTwo(double value) => value * 2;
    public static double Square(double value) => value * value;
}
static void SampleDelegate()
{
    MathOp[] ops =
    {
        MathOperations.MultiplyByTwo,
        MathOperations.Square
    };
    double d = 20.5;
    MathOpInvoker(ops[0], d);
    MathOpInvoker(ops[1], d);
}
static void MathOpInvoker(MathOp math, double value)
{
    Debug.WriteLine(math(value));
}
```

- 在这段代码中，实例化了一个MathOp委托的数组(定义了委托后，就可以像类一样处理，因此数组也可以使用)，该数组的每个元素都初始化指向由MathOperations类实现的不同操作。然后可以通过索引来访问这些委托。

- 对于委托数组，在语法上:
    - `ops[i]` 表示 `这个委托`。
    - `ops[i](2.0)` 表示调用这个方法。


### Action<T>和Func<T>委托

- 除了为每个参数和返回类型定义一个新委托类型之外，还可以使用`Action<T>` 和 `Func<T>` 委托。

- 泛型 `Action<T>`委托表示引用一个void返回类型的方法。这个委托类存在不同的变体，可以传递至多16种不同的参数类型。没有泛型参数的Action类可调用没有参数的方法。`Action<in T>` 调用带一个参数的方法，`Action<in T1, in T2>`调用带两个参数的方法，以此类推。


- `Func<T>` 委托可以以类似的方法使用。 `Func<T>` 允许调用带返回类型的方法。与Action类似，`Func<T>` 也定义了不同的变体，至多也可以传递16个参数类型和一个返回类型。 `Func<out Result>` 委托类型可以调用带返回类型且无参数的方法，`Func<in T, out Result>`调用带一个参数的方法，以此类推。

- 以下是示例：
```
class ActionSample
{
    public static void Print()
    {
        Debug.WriteLine("no param action");
    }
    public static void Print(string msg)
    {
        Debug.WriteLine($"has param action {msg}");
    }
}
class FuncSample
{
    public static string GetStr() => "no param func";
    public static string GetStr(int i) => $"has param func {i}";
}
static void ActionAndFuncTest()
{
    Action action = ActionSample.Print;
    Action<string> action2 = ActionSample.Print;

    Func<string> func = FuncSample.GetStr;
    Func<int, string> func2 = FuncSample.GetStr;

    action();
    action2("hello world");

    Debug.WriteLine(func());
    Debug.WriteLine(func2(1));
}
```

### BubbleSroter示例

- 下面的示例将说明委托的真正用途。我们要编写一个类BubbleSorter，它实现一个静态方法Sort()，这个方法的第一个参数是一个对象数组，把该数组按照升序重新排列。

- 我们这里使用冒泡排序进行排序。

- 我们希望Sort方法能给任何对象排序。但是不是每个对象都可以使用 大于和小于运算符，一些没有重载该运算符的类我们要如何操作？有两种方法
    1. 使用Comparable接口

    2. 使用委托

- 我们接下来介绍委托的方法

- 我们可以传递一个委托，返回一个bool型变量，参数是两个对应的对象。使用这个方法进行比较
```
class BubbleSorter
{
    public void Sort<T>(IList<T> origin, Func<T,T,bool> compare)
    {
        bool swapped = true;
        do
        {
            swapped = false;
            for (int i = 0; i < origin.Count - 1; i++)
            {
                if (compare(origin[i], origin[i + 1]))
                {
                    T temp = origin[i];
                    origin[i] = origin[i + 1];
                    origin[i + 1] = temp;
                    swapped = true;
                }
            }
        } while (swapped);
    }
}
```

- 为了使用这个类，我们定义另一个类，建立要排序的数组。我们建立一个员工类，根据他们的薪水进行排序。
```

```