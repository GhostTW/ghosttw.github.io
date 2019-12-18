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

## Benchmark

* Summary *

BenchmarkDotNet=v0.12.0, OS=Windows 10.0.19041
AMD Ryzen 7 3700X, 1 CPU, 16 logical and 8 physical cores
.NET Core SDK=3.1.100
  [Host]    : .NET Core 3.1.0 (CoreCLR 4.700.19.56402, CoreFX 4.700.19.56404), X64 RyuJIT
  RyuJitX64 : .NET Core 3.1.0 (CoreCLR 4.700.19.56402, CoreFX 4.700.19.56404), X64 RyuJIT

Jit=RyuJit

|                             Method |       Job | Platform |    Count |             Mean |          Error |         StdDev | Ratio | RatioSD |
|----------------------------------- |---------- |--------- |--------- |-----------------:|---------------:|---------------:|------:|--------:|
|                  IterateValueTypes | RyuJitX64 |      X64 |      100 |         71.27 ns |       0.513 ns |       0.480 ns |  1.00 |    0.00 |
|              IterateReferenceTypes | RyuJitX64 |      X64 |      100 |        179.57 ns |       0.687 ns |       0.574 ns |  2.52 |    0.02 |
|     IterateValueTypesDeconstructor | RyuJitX64 |      X64 |      100 |         54.79 ns |       0.392 ns |       0.367 ns |  0.77 |    0.01 |
| IterateReferenceTypesDeconstructor | RyuJitX64 |      X64 |      100 |         53.14 ns |       0.517 ns |       0.484 ns |  0.75 |    0.01 |
|                                    |           |          |          |                  |                |                |       |         |
|                  IterateValueTypes | RyuJitX64 |      X64 |   100000 |     70,376.91 ns |     276.061 ns |     258.228 ns |  1.00 |    0.00 |
|              IterateReferenceTypes | RyuJitX64 |      X64 |   100000 |    178,247.95 ns |     643.249 ns |     570.223 ns |  2.53 |    0.01 |
|     IterateValueTypesDeconstructor | RyuJitX64 |      X64 |   100000 |     50,637.21 ns |     225.525 ns |     210.956 ns |  0.72 |    0.00 |
| IterateReferenceTypesDeconstructor | RyuJitX64 |      X64 |   100000 |     52,488.01 ns |     317.409 ns |     265.051 ns |  0.75 |    0.01 |
|                                    |           |          |          |                  |                |                |       |         |
|                  IterateValueTypes | RyuJitX64 |      X64 | 10000000 |  8,931,029.35 ns |  54,181.022 ns |  48,030.065 ns |  1.00 |    0.00 |
|              IterateReferenceTypes | RyuJitX64 |      X64 | 10000000 | 21,009,876.46 ns | 155,797.185 ns | 145,732.783 ns |  2.35 |    0.02 |
|     IterateValueTypesDeconstructor | RyuJitX64 |      X64 | 10000000 |  8,521,969.61 ns | 163,983.042 ns | 195,210.146 ns |  0.95 |    0.03 |
| IterateReferenceTypesDeconstructor | RyuJitX64 |      X64 | 10000000 | 17,867,946.65 ns | 188,879.313 ns | 167,436.591 ns |  2.00 |    0.02 |

```csharp
Tuple<int, int>[] arrayOfRef;
ValueTuple<int, int>[] arrayOfVal;
//...

[Benchmark(Baseline = true)]
public int IterateValueTypes()
{
    int item1Sum = 0, item2Sum = 0;

    var array = arrayOfVal;
    for (int i = 0; i < array.Length; i++)
    {
        ref ValueTuple<int, int> reference = ref array[i];
        item1Sum += reference.Item1;
        item2Sum += reference.Item2;
    }

    return item1Sum + item2Sum;
}

[Benchmark]
public int IterateReferenceTypes()
{
    int item1Sum = 0, item2Sum = 0;

    var array = arrayOfRef;
    for (int i = 0; i < array.Length; i++)
    {
        ref Tuple<int, int> reference = ref array[i];
        item1Sum += reference.Item1;
        item2Sum += reference.Item2;
    }

    return item1Sum + item2Sum;
}

[Benchmark]
public int IterateValueTypesDeconstructor()
{
    int item1Sum = 0, item2Sum = 0;

    var array = arrayOfVal;
    for (int i = 0; i < array.Length; i++)
    {
        var (item1, item2) = array[i];
        item1Sum += item1;
        item2Sum += item2;
    }

    return item1Sum + item2Sum;
}

[Benchmark]
public int IterateReferenceTypesDeconstructor()
{
    int item1Sum = 0, item2Sum = 0;

    var array = arrayOfRef;
    for (int i = 0; i < array.Length; i++)
    {
        var (item1, item2) = array[i];
        item1Sum += item1;
        item2Sum += item2;
    }

    return item1Sum + item2Sum;
}
```

## References

[此次程式碼](https://github.com/GhostTW/demos/tree/master/CSharpValueTuple)

[Benchmark](https://adamsitnik.com/Value-Types-vs-Reference-Types/)
