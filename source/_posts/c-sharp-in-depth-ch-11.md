---
title: c sharp in depth ch 11
date: 2018-10-24 21:02:26
tags:
---

# ch 11 查询表达式和 LINQ to Objects
## intro
- Language Integrated Query (LINQ)， C# 3的新特性。
- 标准查询操作符 -> 构成查询表达式 -> 转译为普通c#3 代码 -> 编译(推断、重载、Lambda表达式)
- 序列：一次只能取当前的那个元素。IEnumerable
- 延迟执行和流处理：查询表达式被创建时，不会访问数据，而是在内存中生成了这个查询的表现形式。过滤判断通过委托实例来表示，只有在访问结果的第一个元素的时候，才开始执行。每次只处理一个元素。
- 范围变量 (range variable)
- 声明式而非命令式，函数式编程思想

思考
- 何时使用查询表达式、何时使用点标记？
## 选择元素: from select
### from 范围变量 in 数据源 where 过滤 select 投影
```csharp
//11-2
from user in SampleData.AllUsers 
            select user;
```
> 转译=> `SampleData.AllUsers.Select(user => user);`

### 显式类型的范围变量：Cast, OfType
范围变量都可以是隐式类型。Cast，OfType将飞类型化序列转化为强类型。
遇到不匹配类型，Cast报错，OfType跳过。
```csharp
//11-5
ArrayList list = new ArrayList { "First", "Second", "Third"};
IEnumerable<string> strings = list.Cast<string>();

list = new ArrayList { 1, "not an int", 2, 3};
IEnumerable<int> ints = list.OfType<int>();
```

## 筛选排序：where, orderby

### where 
```csharp
//11-8
User tim = SampleData.Users.TesterTim;
var query = from defect in Sample
    where defect.Status != Status.Closed
    where defect.AssignedTo = tim
    select defect.Summary;
```
- Q. 多个where，何时合并？
- A. 逻辑上关联的条件合并在一起，逻辑上不关联的保持独立。这里两个where可以合并。

### 退化的查询表达式：select item
```csharp
.Select(defect => defect)
```
查询表达式的结果和源数据永远不会是同一个对象，对返回数据集的改变也不会影响到“主”数据。
> - `orderby descending` =>转译=> `OrderByDescending()` 或者 `ThenByDescending()`
> - `orderby` =>转译=> `OrderBy()` 或者 `ThenBy()`，默认ascending
> - `ThenBy()` 对之前的一个或多个排序规则起辅助作用，是定义为 `IOrderedEnumerable<T>`的扩展方法

## 透明标识符： let 

<CSharp 4 Specification> 7.16.2.4 From, let, where, join and orderby clauses

> A query expression with a let clause
`from x in e let y = f`
is translated into
`from * in ( e ) . Select ( x => new { x , y = f } )`


## 连接：join
### 内连接：inner join
inner = left, outer = right 内连接从某个对象导航到另一个对象。对右边序列进行缓冲，对左边序列进行流处理。
在SQL中，内连接通常是把某个表的外键和另一个表的主键进行连接。
```csharp
//11-12
from defect in SampleData.AllDefects
    join subscription in SampleData.AllSubscriptions
        on defect.Project equals subcription.Project
    select new {defect.Summary, subscription.EmailAdress};
```
send defect to subcripter's emailbox
> =>转译=>
``` csharp
leftSequence.Join(rightSequence,
    leftKeySelector,
    rightKeySelector,
    resultSelector)
```
### where 
- in left sequence
``` csharp
from defect in SampleData.AllDefects
    where defect.Status == Status.Closed
    join subscription in SampleData.AllSubscriptions
        on defect.Project equals subcription.Project
    select new {defect.Summary, subscription.EmailAdress};
```
- in right sequence
``` csharp
from subscription in SampleData.AllSubscriptions
    join defect in (from defect in SampleData.AllDefects
            where defect.Status == Status.Closed
            select defect)
        on defect.Project equals subcription.Project
    select new {defect.Summary, subscription.EmailAdress};
```
### 分组连接：join into
``` csharp
//11-13
from defect in SampleData.AllDefects
    join subscription in SampleData.AllSubscriptions
        on defect.Project equals subcription.Project
        into groupedSubscriptions
    select new { Defect = defect, Subscriptions = groupedSubscriptions};
```
group join != group by: 对于分组连接来说，左边序列和结果序列是一对一，左边元素不匹配任何右边元素时，嵌入序列是空的。
> =>转译=> `.GroupJoin()`

### 交叉连接：cross join
不存在序列间的匹配操作，结果包含了所有可能的元素对。笛卡尔积(cartesian join)
```csharp
//11-15
from user in SampleData.AllUsers
    from project in SampleData.AllProjects
    select new {User = user, Project = project}
```
- 右边序列依赖于左边元素
```csharp
// 11-16
from left in Enumerable.Range(1, 4)
    from right in Enumerable.Range(11, left)
    select new {Left = left, Right = right};
```
> =>转译=> `.SelectMany()`

## 分组和延续：group by, into 
### group by
键和序列的组合封装于`IGrouping<TKey, TElement> : IEnemerable<TElement>`中。
```csharp
// 11-17
from defect in SampleData.AllDefects
            where defect.AssignedTo != null
            group defect by defect.AssignedTo;
```
> =>转译=>  `.GroupBy()`

group by 的投影
```csharp
//11-18
from defect in SampleData.AllDefects
            where defect.AssignedTo != null
            group defect.Summary by defect.AssignedTo;
```
- 所有查询表达式要以select 或者 group by 子句来结尾。

### 查询延续（query continuations）
- 把一个查询表达式的结果用作另个一查询表达式的初始序列
- select / group by + into 引入新的范围变量（清除之前的范围变量，只有在延续中声明的范围变量才能在后续使用）。

Q. find scope of defect, grouped and result
```csharp
// 11-20 
from defect in SampleData.AllDefects
            where defect.AssignedTo != null
            group defect by defect.AssignedTo into grouped
            select new { Assignee=grouped.Key, 
                         Count=grouped.Count() } into result
            orderby result.Count descending
            select result;
```
> =>转译=> 
```csharp
SampleData.AllDefects
    .Where(defect => defect.AssignedTo != null)
    .GroupBy(defect => defect.AssignedTo)
    .Select(gouped => new { Assignee = grouped.Key,
                            Count = grouped.Count()})
    .OrderByDescending(result => result.Count);
```

## 查询表达式和点标记：dot notation
转译成extension method：查询表达式在编译之前，先被转译为普通的C# （查询操作符、点标记：Enumerable中的扩展方法）.

| 查询表达式 | 点标记 |
| ------- | ------- |
| - | 没有相应查询表达式: .Reverse(), .ToDictionary() 等|
| - | 特定重载 |
| - | 自定义比较器 |
| - | 清晰可读 |
| 多个lambda表达式，多个调用：join 的键选择 | - |
| 排序的多个优先级：orderby f1, f2 | .OrderBy(f1).ThenBy(f2) |

```csharp
// 在匿名类型中直接使用lambda表达式参数
sequence.Select((Item, Index) => new {Item, Index});
```

## reference
[c sharp in depth](http://csharpindepth.com/)
[LINQ to Objects (C#) MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects)
