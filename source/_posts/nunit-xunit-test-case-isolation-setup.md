---
title: NUnit 與 xUnit 在 TestCase 預設成員變數生命週期差異
date: 2019-12-17 23:23:56
tags: ['dotnet', 'xunit', 'nunit']
categories:
---

以前寫了好幾年的 xUnit 都沒在用 `Setup` `OneTimeSetup` ，TestCase 的 arrange 都擠在開頭，最近用 NUnit 寫，被教導說這樣的重用性很低，應該要擅用 `Setup` `OneTimeSetup` 的特性讓程式更乾淨、重用性更高。

順便測試一下兩種 framework 的 TestCase 對類別成員的生命週期特性差異。

## 環境

* Windows 10 Pro version 2004 build 19033.1
* VisualStudio 2019
* dotnet sdk 3.1.100
* NUnit 3.12.0
* xUnit 2.4.1

## NUnit NonParallel

在 NUnit 的流程為 Constructor => OneTimeSetup => Setup => TestCase => TearDown => (next setup) => Dispose => OneTimeTearDown

當在 NUnit 使用類別成員變數在不同地方建立實體會有不同效果

* `list` 在 constructor 建立實體，在類別建立時即有實體，後續的各 TestCase 都會用到同一份實體。
* `listOneTimeSetup` 在 `OneTimeSetup` 建立實體，後續的各 TestCase **還是**會用到同一份實體。
* `listSetup` 在 `Setup` 建立實體，每個 TestCase 開始前會重新建立一個實體給變數，因為是照順序執行 TestCase 就會使用到新的實體。

我這邊用大量的 TestCase 模擬多 `Test` 的行為。

```csharp
[TestFixture]
public class NunitUnitTest
{
  public List<int> list = new List<int>();
  
  public List<int> listSetup;
  
  public List<int> listOneTimeSetup;
  
  public NunitUnitTest() => list = new List<int>();

  [OneTimeSetUp]
  public void OneTimeInit() => listOneTimeSetup = new List<int>();

  [SetUp]
  public void Init() => listSetup = new List<int>();
  
  [Test]
  [TestCase(1)]
  [TestCase(2)]
  [TestCase(3)]
  [TestCase(4)]
  [TestCase(5)]
  [TestCase(6)]
  [TestCase(7)]
  [TestCase(8)]
  [TestCase(9)]
  [TestCase(10)]
  [TestCase(11)]
  [TestCase(12)]
  [TestCase(13)]
  [TestCase(14)]
  [TestCase(15)]
  [TestCase(16)]
  public void TestA(int i)
  {
      list.Add(i);
      listSetup.Add(i);
      listOneTimeSetup.Add(i);
      Console.WriteLine(list.Count);
      Console.WriteLine(listOneTimeSetup.Count);
      Assert.AreEqual(1, listSetup.Count);
      Assert.AreEqual(list.Count, listOneTimeSetup.Count);
  }
}
```

{% asset_img nunit-nonparallel-variables.png nunit-nonparallel-variables %}

## xUnit NonParallel

這邊只測了 xUnit 預設行為，測試結果是每個 TestCase 都是新的獨立的類別實體。

```csharp
public class xUnitUnitTest
{
    public List<int> list = new List<int>();

    public List<int> listSetup = new List<int>();

    public List<int> listOneTimeSetup = new List<int>();
    private ITestOutputHelper output;

    public xUnitUnitTest(ITestOutputHelper output) => this.output = output;

    [Theory]
    [InlineData(1)]
    [InlineData(2)]
    [InlineData(3)]
    [InlineData(4)]
    [InlineData(5)]
    [InlineData(6)]
    [InlineData(7)]
    [InlineData(8)]
    [InlineData(9)]
    [InlineData(10)]
    [InlineData(11)]
    [InlineData(12)]
    [InlineData(13)]
    [InlineData(14)]
    [InlineData(15)]
    [InlineData(16)]
    public void TestA(int i)
    {
        list.Add(i);
        listSetup.Add(i);
        listOneTimeSetup.Add(i);
        output.WriteLine(list.Count.ToString());
        output.WriteLine(listOneTimeSetup.Count.ToString());
        Assert.Single(listSetup);
        Assert.Equal(list.Count, listOneTimeSetup.Count);
    }
}
```

{% asset_img xunit-nonparallel-variables.png xunit-nonparallel-variables %}

## References

[此次程式碼](https://github.com/GhostTW/demos/tree/master/nunit-xunit-test-case-isolation-setup/NUnitTestProject/)
[NUnit SetUp-and-TearDown-Changes](https://github.com/nunit/docs/wiki/SetUp-and-TearDown-Changes)
[xunit shared-context](https://xunit.net/docs/shared-context)
