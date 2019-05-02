# 委托、lambda和事件
<!-- TOC -->

- [委托、lambda和事件](#委托lambda和事件)
    - [引用方法](#引用方法)
    - [委托](#委托)
        - [声明委托](#声明委托)
        - [使用委托](#使用委托)
        - [简单的委托示例](#简单的委托示例)
        - [Action<T>和Func<T>委托](#actiont和funct委托)
        - [BubbleSroter示例](#bubblesroter示例)
        - [多播委托](#多播委托)
        - [匿名方法](#匿名方法)
    - [lambda表达式](#lambda表达式)
        - [参数](#参数)
        - [多行代码](#多行代码)
        - [闭包](#闭包)
    - [事件](#事件)
        - [事件发布程序](#事件发布程序)
        - [事件侦听器](#事件侦听器)
        - [弱事件](#弱事件)

<!-- /TOC -->
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
class Employee
{
    public Employee(string name, decimal salary)
    {
        Name = name;
        Salary = salary;
    }
    public string Name { get; }
    public decimal Salary { get; private set; }

    public override string ToString()
    {
        return $"name : {Name}, salary: {Salary} ";
    }
    public static bool CompareSalary(Employee left, Employee right) => left.Salary > right.Salary;
}
static void BubbleSorterTest()
{
    Employee[] es =
    {
        new Employee("cake", 3000),
        new Employee("army",1000),
        new Employee("bubble", 2000),
        new Employee("design", 1500)
    };
    Func<Employee, Employee, bool> func = Employee.CompareSalary;
    BubbleSorter.Sort(es, func);
    foreach (var item in es)
    {
        Debug.WriteLine(item);
    }
    //BubbleSorter.Sort(es, Employee.CompareSalary);
}
```

### 多播委托
- 前面使用的每个委托都只包含一个方法调用。调用二五i托的次数与调用方法的次数相同。如果要调用多个方法，就需要多次显示调用这个委托。

- 但是，委托也可以包含多个方法。这种委托称为 `多播委托` .如果调用多播委托，就可以按顺序连续调用多个方法。

- 为此，多播委托的签名必须返回void，否则，就只能得到委托调用的最后一个方法。

- 多播委托可以识别运算符 `+` 和 `+=`。用于往委托中添加方法。

- `-` 和 `-=` 用于从委托中删除方法调用。
```
static void MulticastDelegateTest()
{
    Action action = MulticastDelegateExample.PrintA;
    action += MulticastDelegateExample.PrintB;
    action += MulticastDelegateExample.PrintC;
    action();
}
    delegate void Print();
    class MulticastDelegateExample
    {
        public static void PrintA() => Debug.WriteLine("A");
        public static void PrintB() => Debug.WriteLine("B");
        public static void PrintC() => Debug.WriteLine("C");
    }

//print
A
B
C
```

- 多播委托调用一次就会运行所有的方法调用。

- 多播委托实际上是派生自MulticastDelegate的类，该类又派生自基类Delegate。MulitcastDelegate的其他成员允许把多个方法调用链接为一个列表。

- 如果正在使用多播委托，就应知道对同一个委托，调用其方法链的顺序并未正式定义，因此应避免编写依赖以特定顺序执行方法的代码。

- 通过一个委托调用多个方法还可能导致一个更严重的问题。多播委托包含一个逐个调用的委托集合。如果通过委托调用的其中一个方法抛出一个异常，整个迭代就会停止。
```
 public static void One()
{
    Debug.WriteLine("this is one");
    throw new Exception("this is Exception");
}
public static void Two()
{
    Debug.WriteLine("This is two");
}
static void MulticastDelegateExceptionTest()
{
    Action action = MulticastDelegateExample.One;
    action += MulticastDelegateExample.Two;
    try
    {
        action();
    }
    catch (Exception)
    {
        Debug.WriteLine("Ex");
    }
}
//output
this is one
引发的异常:“System.Exception”(位于 C9_DelegateAndEvent.exe 中)
Ex

```

- 在这种情况下，为了避免这个问题，应自己迭代方法列表。Delegate类定义GetInvocationList()方法，它返回一个Delegate对象数组，现在可以使用这个委托调用与委托直接相关的方法，捕获异常，并继续下一个迭代：
```
static void DelegateCustomerIterationTest()
{
    Action action = MulticastDelegateExample.One;
    action += MulticastDelegateExample.Two;
    foreach (Action method in action.GetInvocationList())
    {
        try
        {
            method();
        }
        catch (Exception)
        {
            Debug.WriteLine("Ex");
        }
    }
    
}
//output
this is one
引发的异常:“System.Exception”(位于 C9_DelegateAndEvent.exe 中)
Ex
This is two
```

### 匿名方法
- 到目前为止，要想使委托工作，方法必须已经存在(即委托通过其将调用方法的相同签名定义)。但还有另一种使用委托的方式：通过匿名方法。

- 用匿名方法定义委托的语法与前面的定义并没有区别。但在实例化时，会出现区别：
```
static void AnonymousMethod()
{
    int i = 1;
    Action action = delegate ()
    {
        Debug.WriteLine($"this is Anonymous method,{i}");
    };
    action();
}
```
- 可以看到，匿名方法的语法是 使用关键字delegate，后面是参数列表。

- 可以看出，改代码块使用方法级的变量i，该变量是在匿名方法的外部定义的。这里遵循的原则和JAVA的应该是一致的，对外部变量修改的话会导致里面的值也改变。

- 匿名方法的使用优点是减少了要编写的代码，不必定义仅有委托使用的方法。有助于降低代码的复杂性。使用匿名方法时，代码执行速度并没有加快。编译器仍定义了一个方法，该方法只有一个自动制定的名称。

- 使用匿名方法时，必须遵循两条规则：
    1. 方法中不能使用跳转语句跳到外部
    2. 外部的跳转语句不能跳到内部

- 在匿名方法内部不能访问不安全的代码。另外，也不能访问在匿名方法外部使用的ref和out 参数 ,但可以使用在匿名方法外部定义的其他变量。

- 匿名方法的语法在C#2中引入，在新的程序中，并不需要这个语法，在3.0开始可以使用lambda代替它。

---

## lambda表达式

- C# 3.0以后，可以使用信誉发把实现代码赋予委托:lambda表达式。只要有委托参数类型的地方，就可以使用lambda表达式。前面的匿名方法可以改为：
```
static void SampleLambda()
{
    Action action = () => Debug.WriteLine("this is lambda");
    action();
}
```

- lambda 运算符 `=>` 的左边列出了需要的参数，而其右边定义了赋予lambda变量的方法的实现代码。

### 参数
- lambda表达式有几种定义参数的方式，如果只有一个参数，只写参数名即可。下面的lambda表达式使用了参数s。因为委托类型定义了一个string参数，所以s的类型就是string。
```
Action<string> a1 = s => Debug.WriteLine(s);
```

- 如果使用多个参数，就把参数名放在圆括号中。
```
Action<int, int> a2 = (i1, i2) => Debug.WriteLine(i1+i2);

```

- 可以在圆括号中给变量名添加参数类型。如果编译器不能匹配重载后的版本，那么使用参数类型可以帮助找到匹配的委托。

```
Func<int, int, bool> a3 = (int left, int right) => left == right;

```

### 多行代码

- 如果lambda表达式只有一条语句，在方法块内就不需要花括号和return语句。因为编译器会自动加上。

- 但是也可以手动加上这些。
```
Func<int,bool> func = x =>{
    return x == 0;
};
```
- 如果有多条语句，就必须添加这些
```
Func<int,int,bool> func = (x,y) =>{
    x++;
    return x == y;
};
```

### 闭包
- 通过lambda表达式可以访问lambda表达式块外部的变量，这称为 `闭包`。闭包如果使用不当，会非常危险。

- 如下示例
```
static void ClosureTest()
{
    int val = 5;
    Func<int, int> func = x => x + val;
    val = 10;
    Debug.WriteLine(func(5));
}
```
- 代码中使用了外部变量val，但是在后面修改了val的值，于是在调用lambda表达式时，会使用val的新值。所以最后输出结果是15。

- 同样，在lambda中修改闭包的值时，可以在lambda表达式外部访问已改动的值。
```
int val = 10;
Action action = () =>
{
    val = 15;
};
Debug.WriteLine(val);//10
action();
Debug.WriteLine(val);//15
```

- 为了理解如何在lambda表达式的内部访问lambda表达式外部的变量，我们看看编译器做了什么。

- 对于lambda表达式 `x=>x+val`，编译器会创建一个匿名类，他有一个构造函数来传递外部变量。该构造函数取决于从外部访问的变量数。
```
//反编译代码
[CompilerGenerated]
private sealed class <>c__DisplayClass16_0
{
	public int val;

	internal int <ClosureTest>b__0(int x)
	{
		return x + val;
	}

	internal void <ClosureTest>b__1()
	{
		val = 15;
	}
}
```

- 使用lambda表达式并调用该方法，会创建一个实例，并传递调用该方法时变量的值。
```
AnonymousClass obj = new AnonymousClass();
obj.val = 5;
Func<int, int> func = new Func<int, int>(obj.<ClosureTest>b__0);
obj.val = 10;
Debug.WriteLine(func(5));
Action action = new Action(obj.<ClosureTest>b__1);
Debug.WriteLine(obj.val);
action();
Debug.WriteLine(obj.val);
```

- 如果给多个线程使用闭包，就可能遇到并发冲突。最好仅给闭包使用不变的类型。这样可以确保不改变值，也不需要同步。

- lambda表达式也可以用于类型为委托的任意地方。类型是Expression或`Expression<T>`时，也可以使用lambda表达式。

---
## 事件

- 事件基于委托，为委托提供了一种发布/订阅机制。在`.NET`架构内到处都能看到事件。在Windows应用程序中，Button类提供了Click事件。这类事件就是委托。除法Click事件时调用的处理程序方法需要得到定义，而其参数由委托类型定义。

- 本节示例中，事件用于连接CarDealer类和Consumer类。CarDealer类提供了一个新车到达时除法的事件。Consumer类订阅该事件，以获得新车到达的通知。

### 事件发布程序
- CarDealer类基于事件提供一个订阅。CarDealer类用`event`关键字定义了类型为 `EventHandler<CarInfoEventArgs>` 的NewCarInfo事件。在NewCar()方法中，通过调用`RaiseNewCarInfo`方法触发NewCarInfo事件。
```
    public class CarInfoEventArgs:EventArgs
    {
        public string Car { get; }
        public CarInfoEventArgs(string car)
        {
            Car = car;
        }
    }
        public class CarDealer
    {
        public event EventHandler<CarInfoEventArgs> NewCarInfo;
        public void NewCar(string car)
        {
            Debug.WriteLine($"CarDealer, new car {car}");

            NewCarInfo?.Invoke(this, new CarInfoEventArgs(car));
            
        }
    }
```

- CarDealer类提供了`EventHandler<CarInfoEventArgs>`类型的NewCarInfo事件。

- 作为一个预定，事件一般使用带两个参数的方法：
    - 第一个参数是一个对象，包含事件的发送者
    - 第二个参数提供了事件的相关信息

- 第二个参数随不同的事件类型而改变。`.NET` 1.0为所有不同数据类型的事件定义了几百个委托。有了泛型委托`EventHandler<T>`后，就不再需要这些委托了。

- `EventHandler<TEventArgs>` 定义了一个处理程序，它返回void，接收两个参数。 对于`EventHandler<TEventArgs>`，第一个参数必须是object类型，第二个参数是T类型。 `EventHandler<TEventArgs>` 还定义了一个关于T的约束：他必须派生自基类EventArgs。
```
public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e) where TEventArgs: EventArgs;
```

- 在一行上定义事件是C#的简化记法。编译器会创建一个`EventHandler<CarInfoEventArgs>`委托类型的变量，并添加方法，以便于从委托中订阅和取消订阅。

- 该简化记法的较长形式如下：这非常类似于自动属性和完整属性之间的关系。 对于事件，使用 `add` 和 `remove` 关键字添加和删除委托的处理程序：
```
        private EventHandler<CarInfoEventArgs> newCarInfo;
        public event EventHandler<CarInfoEventArgs> NewCarInfo
        {
            add
            {
                newCarInfo += value;
            }
            remove
            {
                newCarInfo -= value;
            }
        }
```

- 如果不仅需要添加和删除事件处理程序，定义事件的长记法就很有用，例如，需要为多个线程访问添加同不操作。WPF空间使用长记法给事件添加冒泡和隧道功能。

- CarDealer类通过调用委托的RaiseNewCarInfo方法触发事件。(?应该是其本身定义的Raise方法吧)。使用委托NewCarInfo和小括号可以调用给事件订阅的所有处理程序。
```
var info = newCarInfo;
if(info != null)
{
    //使用小括号调用方法
    info(this, new CarInfoEventArgs(car));
}
//C# 6.0以后可以用null传播运算符代替：
newCarInfo?.Invoke(this, new CarInfoEventArgs(car));

```

- 注意，与之前的多播委托一样，方法的调用顺序无法保证，为了更多地控制处理程序的调用，可以使用Delegate类的GetInvocationList方法，访问委托类表中的每一项，并独立第调用每个方法。

- 在C#6.0之前，触发事件比较复杂。首先需要检查事件是否为空。因为在null检查和触发事件之间，可以使用另一个线程把事件设置为null，所以使用一个局部变量，避免调用null对象。
```
//为了避免newCarInfo被设置为Null
var info = newCarInfo;
if(info != null)
{
    //使用小括号调用方法
    info(this, new CarInfoEventArgs(car));
}
```

### 事件侦听器

- Consumer类用作事件侦听器。这个类订阅了CarDeler类的事件，并定义了NewCarISHere方法，该方法满足 `EventHandler<CarInfoEventArgs>` 委托的要求，该委托的参数类型是object和CarInfoEventArgs。
```
public class Consumer
{
    private string _name;
    public Consumer(string name)
    {
        _name = name;
    }
    public void NewCarIsHere(object sender, CarInfoEventArgs e)
    {
        Debug.WriteLine($"{_name} : car {e.Car} is new");
    }
}
```

- 现在需要连接事件发布程序和订阅器。为此使用CarDealer类的NewCarInfo事件，通过 `+=` 创建一个订阅。消费者michael(变量)订阅了事件，接着消费者sebastian(变量)也订阅了事件，然后michael(变量)通过 `-=` 取消了订阅。
```
static void EventTest()
{
    var dealer = new CarDealer();
    var michael = new Consumer("Michael");
    dealer.NewCarInfo += michael.NewCarIsHere;

    dealer.NewCar("Mercedes");

    var sebastian = new Consumer("Sebastian");
    dealer.NewCarInfo += sebastian.NewCarIsHere;

    dealer.NewCar("Ferrari");

    dealer.NewCarInfo -= michael.NewCarIsHere;

    dealer.NewCar("Red Bull Racing");
}
//output
CarDealer, new car Mercedes
Michael : car Mercedes is new
CarDealer, new car Ferrari
Michael : car Ferrari is new
Sebastian : car Ferrari is new
CarDealer, new car Red Bull Racing
Sebastian : car Red Bull Racing is new
```

- 分析程序，其实就是一个观察者模式。
    - Subject是NewCarInfo
    - Observer是Consumer

- 通过委托完成订阅功能。使用事件NewCarInfo完成发布功能。

### 弱事件

- 通过事件，可直接连接发布程序(subject)和侦听器(observer)。但是，垃圾回收方面存在问题。例如，如果不再直接引用侦听器，发布程序就仍有一个引用。垃圾回收器不能清空侦听器占用的内存，因为发布程序仍保有一个引用，会针对侦听器触发事件。

- 这种强连接可以通过弱事件模式来解决。即使用WeakEventManager作为发布程序和侦听器之间的中介。

- 现在修改前面的示例，以使用弱事件模式。

- `WeakEventManager<T>` 在System.Windows程序集中定义，不属于`.NET Core`。

- 动态创建订阅器是，为了避免出现资源泄露，必须特别留意事件。也就是说，需要在订阅器离开作用域(不再需要它)之前，确保取消对事件的订阅，或者使用弱事件。事件常常是应用程序中内存泄漏的一个原因，因为订阅器有长时间存在的作用域，所以源代码也不能被垃圾回收。

- 使用弱事件，不需要改变事件发布器。无论使用紧密耦合的事件还是弱事件都没有关系，其实现是一样的。不同的是使用者的实现。使用者需要实现接口IWeakEventListener。这个接口定义了方法ReceiveWeakEvent，在事件触发时会在弱事件管理器中调用该方法。该方法的实现充当代理，调用方法NewCarIsHere。
