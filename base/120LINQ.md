# LINQ

<!-- TOC -->

- [LINQ](#linq)
    - [LINQ概述](#linq概述)
        - [列表和实体](#列表和实体)
        - [LINQ查询](#linq查询)
        - [扩展方法](#扩展方法)
        - [推迟查询的执行](#推迟查询的执行)
    - [标准的查询操作符](#标准的查询操作符)
        - [筛选](#筛选)
        - [用索引筛选](#用索引筛选)
        - [类型筛选](#类型筛选)
        - [复合的from子句](#复合的from子句)

<!-- /TOC -->
## LINQ概述
- LINQ(Language Integrated Query, 语言集成查询)在C#编程语言中集成了查询语法，可以用相同的语法访问不同的数据源。LINQ提供了不同数据源的抽象层，所以可以使用相同的语法。

### 列表和实体
- 一共有4个类：
    - Racer是赛车手实体。
    - Team是车队。
    - Formula1是一个静态方法，返回一些数据实体。
    - ChampionShip是冠军信息，包括第二第三名和年份。

### LINQ查询
- 使用这些准备好的列表和实体，进行LINQ查询，例如，查询出来自巴西的所有世界冠军，并按照夺冠次数排序。使用LINQ的语法非常简单。
```
static void FromBrazil()
{
    var query = from r in Formula1.GetChampions()
                where r.Country == "Brazil"
                orderby r.Wins descending
                select r;
    foreach (Racer r in query)
    {
        Debug.WriteLine($"{r:A}");
    }
}
//
Ayrton Senna, Brazil; Starts: 161, wins: 41
Nelson Piquet, Brazil; Starts: 204, wins: 23
Emerson Fittipaldi, Brazil; Starts: 143, wins: 14
```
- 表达式
```
var query = from r in Formula1.GetChampions()
                where r.Country == "Brazil"
                orderby r.Wins descending
                select r;
```
- 是一个LINQ查询。子句from where orderby descending select都是这个查询中预定义的关键字。

- 查询表达式必须以from子句开头，以select或group子句结束。在这两个子句之间，可以使用where orderby join let和其他from子句。

- 变量query只指定了LINQ查询。该查询不是通过这个赋值语句执行的，只要使用foreach循环访问查询，该查询就会执行。

### 扩展方法
- 编译器会转换LINQ查询，以调用方法而不是LINQ查询。LINQ为`IEnumerable<T>`接口提供了各种扩展方法，以便用户在实现了该接口的任意集合上使用LINQ查询。扩展方法在静态类中声明，定义为一个静态方法，其中第一个参数定义了它扩展的类型。
```
public static class Extension{
    public static Method(this ClassType obj, param...){
        //obj就是调用方法的对象，也即是java的隐式参数
        // obj.Method(param);
    }
}
```
- 扩展方法可以将方法写入最初没有提供该方法的类中。还可以把方法添加到实现某个特定皆苦的任何类中，这样多个类就可以使用相同的实现代码。

- 例如，String类没有Foo方法。String类是密封的，所以不能从这个类中继承。但可以创建一个扩展方法：
```
public static class StringExtension{
    public static void Foo(this string s){
        WriteLine($"Foo invoked for {s}");
    }
}
static void ExtensionTest()
{
    string s = "test";
    s.Foo();// = StringExtension.Foo(s);
}
```

- Foo方法扩展了String类，因为他的第一个参数定义为String类型。为了区分扩展方法和一般的静态方法，扩展方法还需要对第一个参数使用this关键字。

- 也许这看起来违反了面向对象的规则，因为给一个类型定义了新方法，但没有改变该类型或派生自他的类型。但实际上这只是一个语法糖，扩展方法不能访问它扩展的类型的私有成员，和直接传参调用静态方法是一样的。
```
s.Foo() = StringExtension.Foo(s)
```

- 定义LINQ扩展方法的一个类是System.Linq名称空间中的Enumerable。下面列出了Where扩展方法的实现代码。
```
public static IEnumerable<TSource> where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate){
    foreach(TSource item in source){
        if(predicate(item)){
            yield return item;
        }
    }
}
```
- 谓词是返回布尔值的方法。
- 因为Where作为一个泛型方法实现，所以他可以用于包含在集合中的任意类型。实现了`IEnumerable<T>`接口的任意集合都支持它。

- 现在就可以使用Enumerable类中的扩展方法Where OrderByDescending和Select。这些方法都返回`IEnumerable<TSource>`，所以可以使用 前面的结果依次调用这些方法。
```
IEnumerable<Racer> brazilChampions = Formula1.GetChampions().Where(r => r.Country == "Brazil").
                OrderByDescending(r => r.Wins).
                Select(r => r);
foreach (var r in brazilChampions)
{
    Debug.WriteLine($"{r:A}");
}
```

### 推迟查询的执行
- 在运行期间定义查询表达式时，查询就不会运行，查询会在迭代数据项时运行。

- 再看看扩展方法Where。它使用yield return语句返回谓词为true的元素。因为使用了yield return语句，所以编译器会创建一个枚举器，在范围跟枚举中的项后，就返回它们。

- 这是一个非常有趣也非常重要的结果。也就是说如果我们在定义查询之后，往集合中新添加元素，新添加的元素也会被迭代。
```
static void LazyQuery()
{
    List<string> names = new List<string>() { "Nino", "Alberto", "Juan" };
    var nameWithJ = from ns in names
                    where ns.Contains("J")
                    orderby ns
                    select ns;
    foreach (var name in nameWithJ)
    {
        Debug.WriteLine(name);
    }
    Debug.WriteLine("second");
    names.Add("June");
    names.Add("Jack");
    foreach (var name in nameWithJ)
    {
        Debug.WriteLine(name);
    }
}
//
Juan
second
Jack
Juan
June
```

- 当然，必须注意，每次在迭代中使用查询时，都会调用扩展方法。在大多数情况下，这是非常有效的，因为我们可以检测出源数据的变化。但是在一些情况下，这是不可行的。这时候我们可以调用ToList方法遍历集合，返回一个IList的集合。这时候新的集合就和源数据分开了，对源集合的操作不会再影响该集合。
```
static void ToListTest()
{
    List<string> names = new List<string>() { "Nino", "Alberto", "Juan" };
    var nameWithJ = from ns in names
                    where ns.Contains("J")
                    orderby ns
                    select ns;
    var list = nameWithJ.ToList();
    foreach (var name in list)
    {
        Debug.WriteLine(name);
    }
    Debug.WriteLine("second");
    names.Add("June");
    names.Add("Jack");
    foreach (var name in list)
    {
        Debug.WriteLine(name);
    }
}
//
Juan
second
Juan
```

---
## 标准的查询操作符
- where OrderByDescending 和Select只是LINQ定义的几个查询操作符。LINQ查询为最常用的操作符定义了一个声明语法。还有许多查询操作符可用于Enumerable类。
- P345页 表13-1 说明了这些操作符

### 筛选
- 使用where子句，可以合并多个表达式。传递个where子句的表达式的结果类型应是布尔类型。
```
var champions = Formula1.GetChampions();
var racers = from r in champions
                where r.Wins > 15 && r.Country == "Brazil"
                select r;
```

- 并不是所有的查询都可以用LINQ查询语法完成。也不是所有的扩展方法都映射到LINQ查询子句上。高级查询需要使用扩展方法。与上面同等的扩展方法是:
```
var racers2 = champions.Where(r => r.Wins > 15 && r.Country == "Brazil").Select(r => r);
```

### 用索引筛选
- 不能使用LINQ查询的一个例子是Where方法的重载。在Where方法的重载中，可以传递第二个参数——索引。索引是筛选器返回的每个结果的计数器。可以在表达式中使用这个索引，执行基于索引的计算。下面的代码使用索引返回姓氏以A开头、索引为偶数的赛车手：
```
var champions = Formula1.GetChampions();
var racers = champions.Where((r, index) => r.LastName.StartsWith("A") && index % 2 == 0);
Print(racers);
```

### 类型筛选
- 为了进行基于类型的筛选，可以使用OfType扩展方法。这里数组数据包含string和int对象，使用OfType扩展方法，把string类传送给泛型参数，就从集合中仅返回字符串。
```
object[] data = { "1", 1, "2", 2, "3", 3 };
var query = data.OfType<string>();
Print(query);
```

### 复合的from子句