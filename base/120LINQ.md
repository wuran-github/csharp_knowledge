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
        - [排序](#排序)
        - [分组](#分组)
        - [LINQ查询中的变量](#linq查询中的变量)
        - [对嵌套的对象分组](#对嵌套的对象分组)
        - [内连接](#内连接)
        - [左外连接](#左外连接)
        - [总结](#总结)

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
- 如果需要根据对象的一个成员进行筛选，而该成员本身是一个系列，就可以使用复合的from子句。如Racer的Cars是字符串数组，因此可以使用第一个from访问racers，第二个from访问cars，然后根据赛车去筛选racer。
```
var ferrariDrivers = from r in Formula1.GetChampions()
                    from c in r.Cars
                    where c == "Ferrari"
                    orderby r.LastName
                    select r.FirstName + " " + r.LastName;
Print(ferrariDrivers);
```

- C#编译器把符合的from子句和LINQ查询转换为SelectMany扩展方法。SelectMany可以用于迭代序列的序列。

- SelectMany方法的重载版本如下：
```
public static IEnumerable<TResult> SelectMany<TSource, TCollection, TResult>(
    this IEnumerable<TSource> source,
    Func<Tsource, IEnumerable<TCollection>> collectionSelector,
    Func<TSource, TCollection, TResult> resultSelector);
)
```
- 第一个是隐式参数。第二个参数是collectionSelector委托，其中定义了内部序列。在lambda表达式r => r.Cars中，应返回赛车集合。
- 第三个参数是一个委托，现在为每个赛车调用该委托，接收Racer和Car对象。lambda表达式创建了一个匿名类型，它有Racer和Car属性。
  
- 这个SelectMany方法的结果是摊平了赛车手和赛车的层次结构，为每辆赛车返回匿名类型的一个新对象集合。

```
 var champions = Formula1.GetChampions();
var ferrariDrivers = champions.SelectMany(r => r.Cars, (r, c) => new { Racer = r, Car = c })
        .Where(r => r.Car == "Ferrari")
        .OrderBy(r => r.Racer.LastName)
        .Select(r => r.Racer.FirstName + " " + r.Racer.LastName);
Print(ferrariDrivers);
```

- 当然还可以使用where来达到这个效果，直接判断racer的Cars里是否含有Ferrari的赛车。
```
var Drivers = champions.Where(r => r.Cars.Any(c => c == "Ferrari"))
        .OrderBy(r => r.LastName)
        .Select(r => r.FirstName + " " + r.LastName);
```

- 但是SelectMany可以改变原来的层次结构。也就是说我可以把赛车手和赛车放到同一层次输出：
```
var champions = Formula1.GetChampions();
var driversAndCar = champions.SelectMany(r => r.Cars, (r, c) => new { Racer = r, Car = c })
    .OrderBy(r => r.Car)
    .Select(r => "Car:" + r.Car + ", racer:" + r.Racer.FirstName + " " + r.Racer.LastName);
```

### 排序
- 要对序列排序，前面使用了orderby子句。orderby descending可以降序排序。

- orderby子句解析为OrderBy方法，orderby descending 子句解析为OrderByDescending方法：
```
var racers = Formula1.GetChampions()
    .Where(r => r.Country == "Brazil")
    .OrderByDescending(r => r.Wins);
Print(racers);
```

- OrderBy和OrderByDescending方法返回`IOrderEnumerable<TSource>`。这个接口派生自`IEnumerable<T>`接口，担保函一个额外的方法`CreateOrderedEnumerable<TSource>()`。这个方法用于进一步给序列排序。

- 如果根据关键字选择器来排序，其中有两项相同，就可以使用ThenBy和ThenByDescending方法继续排序。这两个方法需要`IOrderEnumerable<TSource>`接口，但也返回该接口，所以可以无限次调用这两个方法进行排序。

- 使用LINQ查询时，只需要把所有用于排序的不同关键字(用逗号分隔开)添加到orderby子句中。
```
var racers = (from r in Formula1.GetChampions()
                orderby r.Country, r.LastName, r.FirstName
                select $"{r:A}").Take(10);
Print(racers);
//
Juan Manuel Fangio, Argentina; Starts: 51, wins: 24
Jack Brabham, Australia; Starts: 125, wins: 14
Alan Jones, Australia; Starts: 115, wins: 12
Niki Lauda, Austria; Starts: 173, wins: 25
Jochen Rindt, Austria; Starts: 60, wins: 6
Emerson Fittipaldi, Brazil; Starts: 143, wins: 14
Nelson Piquet, Brazil; Starts: 204, wins: 23
Ayrton Senna, Brazil; Starts: 161, wins: 41
Jacques Villeneuve, Canada; Starts: 165, wins: 11
Mika Hakkinen, Finland; Starts: 160, wins: 20
```

### 分组
- 要根据一个关键字值对查询结果分组，可以使用group子句。现在一级方程式冠军应按照国家分组，并列出一个国家的冠军赛车手数。
    - 子句group r by r.Country into g根据Country属性组合所有的赛车手，并定义一个新的标识符g，它用于访问分组的结果信息。
    - group子句的结果根据应用到分组结果上的扩展方法Count来排序，如果冠军书相同，就根据关键字来排序，该关键字是国家，这是分组所使用的关键字。
    - where子句根据至少有2项的分组来筛选结果
    - select子句创建一个带Country和Count属性的匿名类型
```
var champions = Formula1.GetChampions();
var countries = from r in champions
                group r by r.Country into g
                orderby g.Count() descending, g.Key
                where g.Count() >= 2
                select new
                {
                    Country = g.Key,
                    Count = g.Count()
                };
```

- 要用扩张方法执行相同的操作，应把groupby子句解析为GroupBy方法。在GroupBy方法的声明中，注意它返回实现了IGrouping接口的枚举对象。
- IGrouping接口定义了Key属性，所以在定义了对这个方法的调用后，可以访问分组的关键字。
```
public static IEnumerable<IGrouping<TKey, TSource>> GroupBy<Tsource, Tkey>(
    this IEnumerable<TSource> source, Func<Tsource, TKey> keySelector);
)

- 把子句group r by r.Country into g解析为GroupBy(r=> r.Country)，返回分组序列。
```
```
 var extension = champions.GroupBy(r => r.Country)
    .OrderByDescending(g => g.Count())
    .ThenBy(g => g.Key)
    .Where(g => g.Count() >= 2)
    .Select(g => new { Country = g.Key, Count = g.Count() });
```

### LINQ查询中的变量
- 在为分组编写的LINQ查询中，Count方法调用了多次。使用let子句可以改变这种方式。let允许在LINQ查询中定义变量：
```
var countries = from r in champions
                group r by r.Country into g
                let count = g.Count()
                orderby count descending, g.Key
                where count >= 2
                select new
                {
                    Country = g.Key,
                    Count = count
                };
```
- 使用方法语法，Count方法也调用了多次。为了定义传递给下一个方法的额外数据(let子句执行的操作)，可以使用Select方法来创建匿名类型。这里创建了一个带Group和Count属性的匿名类型。带有这些属性的一组项可以传递给OrderByDescending方法。
```
var extension = champions.GroupBy(r => r.Country)
    .Select(g => new {Group = g, Count = g.Count() })
    .OrderByDescending(g => g.Count)
    .ThenBy(g => g.Group.Key)
    .Where(g => g.Count >= 2)
    .Select(g => new { Country = g.Group.Key, Count = g.Count });
```

- 应考虑根据let子句或Select方法创建的临时对象的数量。查询大列表时，创建的大量对象需要以后进行垃圾收集，这可能对性能产生巨大影响。

### 对嵌套的对象分组
- 如果分组的对象应包含嵌套的序列，就可以改变select子句创建的匿名类型。

- 在下面的例子中，所返回的国家不仅应包含郭家铭和赛车手数量两个属性，还应包含赛车手的名序列。这个序列用一个赋予Racers属性的from/in内部子句指定，内部的from子句使用分组标识符g获得该分组中的所有赛车手，用姓氏对他们排序，再根据姓名创建一个新字符串。
```
var champions = Formula1.GetChampions();
var countries = from r in champions
                group r by r.Country into g
                let count = g.Count()
                orderby count descending, g.Key
                where count >= 2
                select new
                {
                    Country = g.Key,
                    Count = count,
                    
                    Racers = from r1 in g
                                orderby r1.LastName
                                select r1.FirstName + " " + r1.LastName
                };
```
- 使用扩展方法对应的方法如下：
```
var extension = champions.GroupBy(r => r.Country)
    .Select(g => new { Group = g, Count = g.Count() })
    .OrderByDescending(g => g.Count)
    .ThenBy(g => g.Group.Key)
    .Where(g => g.Count >= 2)
    .Select(g => new
    {
        Country = g.Group.Key,
        Count = g.Count,
        Racers = g.Group.OrderBy(r => r.LastName)
        .Select(r => r.FirstName + " " + r.LastName)
    });
```

### 内连接
- 使用join子句可以根据特定的条件合并两个数据源，但之前要获得两个要连接的列表。下面是连接赛车手冠军和车队冠军，列出每年的赛车手冠军和车队冠军。
  
- 首先要含有年份的车队和赛车手的两个查询，然后使用join语句连接它们。
- join的语法是，先是第一个表的常规from，然后是join代替from，连接第二个表。on代表连接条件
```
var query = from t1 in table1
            join t2 in table2 on t1.condition equals t2.condition
            ... 
```
```
var racers = from r in Formula1.GetChampions()
                from y in r.Years
                select new
                {
                    Year = y,
                    Name = r.FirstName + " " + r.LastName
                };
var teams = from t in Formula1.GetConstructorChampions()
            from y in t.Years
            select new
            {
                Year = y,
                Name = t.Name
            };
var racersAndTeams = from r in racers
                        join t in teams on r.Year equals t.Year
                        select new
                        {
                            r.Year,
                            Champion = r.Name,
                            Constructor = t.Name
                        };
```
- 也可以合并成一个LINQ查询。

- 扩展方法使用Join，隐式参数是第一个表，第一个参数是第二个表，第二个参数和第三个参数是要匹配的条件，第四个参数是创建的新类型。
```
var extension = racers.Join(teams, r => r.Year, t => t.Year,
    (r, t) => new {
        r.Year,
        Champion = r.Name,
        Constructor = t.Name
    });
```

### 左外连接
- 内连接是根据连接条件，当两个数据源都有数据是才会产生连接数据。有时候我们需要把所有的数据都列出来。

- 这时候使用左外连接。如上述的我们需要包含所有的年份，而1958年开始才有车队冠军。

- 左外连接用Join和DefaultIfEmpty方法定义。
```
var racersAndTeams = from r in racers
                    join t in teams on r.Year equals t.Year into rt
                    from t2 in rt.DefaultIfEmpty() //t2会代替掉t
                    orderby r.Year
                    select new
                    {
                        Year = r.Year,
                        Champion = r.Name,
                        Constructor = t2 == null ? "no constructor" : t2.Name
                    };
```
- 解释过程：
    - 首先在join之后加了一个into关键字，代表把t的数据全部into到rt中，这样t的作用域就结束了
    - 后面使用from 从rt中查询，rt.DefaultIfEmpty是指如果rt中没数据就返回Default数据，这时候rt的作用域也结束了
    - 后面就可以去创建新的类型了，如果r有数据而team没数据，就会返回null，只需要判断t2是不是Null即可。

- 扩展方法：
```

```
### 总结
- linq子句总是会被编译器翻译成扩展方法。
- groupby语句分出来的group里包含着组成对应组的所有行信息。