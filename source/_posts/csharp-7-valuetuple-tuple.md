---
title: C# 7 Tuple and ValueTuple
date: 2019-12-19 00:16:02
tags: ['dotnet', 'csharp', 'c#7']
categories:
---

舊功能筆記，太久沒用有些細節忘記了，被提醒了一波，還是筆記下來好了。

* Tuple 是 class
* ValueTuple 是 struct 所以不會被 GC 管控，節省 GC 效能。

## 實驗程式

1. `TypeCheck` 驗證各種使用方式實際上是用到哪種 Tuple
1. `ValueTypeCheck` 驗證兩種 Tuple 哪個是 ValueType
1. `Deconstruction` 驗證使用 Deconstruction 的方式
1. `Deconstruction_AsIgnoreVariable` 驗證使用 ignore variable 方式

```csharp
public class Program
{

    public Tuple<string, int> SimpleTuple => new Tuple<string, int>("testTuple", 1);

    public (string, int) SimpleValueTuple => ("testValueTuple", 2);

    public (string Name, int Value) ValueTupleName => (Name: "testValueTupleName", Value: 3);

    public ValueTuple<string, int> TypeValueTyple => (Name: "testTypeValueTup", Value: 4);

    public ValueTuple<string, int> TypeValueTypleA => ValueTuple.Create("testTypeValueTup", 4);
}

public class Tests
{
    private Program _sut;

    [SetUp]
    public void Setup()
    {
        _sut = new Program();
    }

    [Test]
    public void TypeCheck()
    {
        Assert.IsTrue(_sut.SimpleTuple.GetType() == typeof(Tuple<string, int>));
        Assert.IsTrue(_sut.SimpleValueTuple.GetType() == typeof(ValueTuple<string, int>));
        Assert.IsTrue(_sut.ValueTupleName.GetType() == typeof(ValueTuple<string, int>));
        Assert.IsTrue(_sut.TypeValueTyple.GetType() == typeof(ValueTuple<string, int>));
        Assert.IsTrue(_sut.TypeValueTypleA.GetType() == typeof(ValueTuple<string, int>));
    }


    [Test]
    public void ValueTypeCheck()
    {
        Assert.IsFalse(_sut.SimpleTuple.GetType().IsValueType);
        Assert.IsTrue(_sut.SimpleValueTuple.GetType().IsValueType);
    }

    [Test]
    public void Deconstruction()
    {
        var (name, value) = _sut.SimpleTuple;
        Assert.AreEqual(name, "testTuple");
        Assert.AreEqual(value, 1);

        (name, value) = _sut.SimpleValueTuple;
        Assert.AreEqual(name, "testValueTuple");
        Assert.AreEqual(value, 2);
    }

    [Test]
    public void Deconstruction_AsIgnoreVariable()
    {
        var (name, _) = _sut.SimpleTuple;
        Assert.AreEqual(name, "testTuple");
        //Assert.AreEqual(_, 1); //Error CS0103  The name '_' does not exist in the current context CSharpValueTuple


        (name, _) = _sut.SimpleValueTuple;
        Assert.AreEqual(name, "testValueTuple");
        //Assert.AreEqual(_, 2); //Error CS0103  The name '_' does not exist in the current context CSharpValueTuple
    }
```

## References

[此次程式碼](https://github.com/GhostTW/demos/tree/master/CSharpValueTuple)
