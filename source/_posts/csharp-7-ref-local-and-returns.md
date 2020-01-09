---
title: C# 7.0 ref local and returns
date: 2020-01-09 22:49:00
tags: ['dotnet', 'csharp', 'c#7']
categories:
---

實驗一下 C# 7.0 ref local 和 returns 的功能

使用 [LINQPad 6](https://www.linqpad.net/) 測試

```csharp
void Main()
{
  PriceA.Dump("A"); // 199
  PriceB.Dump("B"); // 99
  PriceC.Dump("C"); // 9
  
  "---".Dump("GetPriceA()");
  ref decimal price = ref GetPriceA();
  PriceA.Dump("A"); // 199
  (++price).Dump("price++");
  PriceA.Dump("A"); // 200
  
  "---".Dump("GetPriceB()");
  
  price = GetPriceB();
  price.Dump("price"); // 99
  PriceA.Dump("A"); // 99
  PriceB.Dump("B"); // 99
  (++price).Dump("price++"); // 100
  PriceA.Dump("A"); // 100
  PriceB.Dump("B"); // 99
  
  "---".Dump("ref GetPriceC()");
  
  price = ref GetPriceC();
  price.Dump("price"); // 9
  PriceA.Dump("A"); // 100
  PriceB.Dump("B"); // 99
  PriceC.Dump("C"); // 9
  (++price).Dump("price++"); // 10
  PriceA.Dump("A"); // 100
  PriceB.Dump("B"); // 99
  PriceC.Dump("C"); // 10
}

private decimal PriceA = 199;
private decimal PriceB = 99;
private decimal PriceC = 9;

public ref decimal GetPriceA()
{
  return ref this.PriceA;
}

public decimal GetPriceB()
{
  return this.PriceB;
}

public ref decimal GetPriceC()
{
  return ref this.PriceC;
}

```

## Reference

[Code](https://gist.github.com/GhostTW/bae9c6b000db9193dd8600e18083c26f)
[csharp-7#ref-locals-and-returns](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7#ref-locals-and-returns)
