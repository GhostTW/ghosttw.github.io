---
title: IEnumerable, Enumerator, LinQ 底層研究及 IEnumerable 不可信的原因!
date: 2019-12-26 21:22:44
tags: ['csharp']
categories:
---

## 源由

昨天被朋友問了一個問題，乍看之下以為不是什麼大問題，後來發現自己還是對 `IEnumerable` 的應用不夠熟，一般也很少這樣寫，沒直覺想到這會是個陷阱。

題目大概如下，請問會印出值嘛?

```csharp
public static void Process(IEnumerable<ReferenceClass> sources)
{
  foreach (ReferenceClass item in sources)
      item.Status = true;
  ShowSuccessStatus(sources);
}

private static void ShowSuccessStatus(IEnumerable<ReferenceClass> sources)
{
  foreach (ReferenceClass item in sources)
  {
    if (!item.Status)
        continue;
  
    Console.WriteLine($"{nameof(item.Status)}: {item.Status}");
  }
}

public class ReferenceClass { public bool Status { get; set; }}

```

答案是，有可能 `True` 也有可能什麼都印不出來。

## IEnumerable 來源

這是題目給的資料源

```csharp
static void Main()
{
  //var sources = new List<ReferenceClass> { new ReferenceClass(), new ReferenceClass(), new ReferenceClass() };
  var sources = (new List<int> { 1,2,3 }).Select(item => new ReferenceClass());
  Process(sources);
}
```

比較常用的情境資料來源通常是

`var sources = new List<ReferenceClass> { new ReferenceClass(), new ReferenceClass(), new ReferenceClass() };`

但這題的陷阱在於資料集是由這個衍生的

`var sources = new List<int> { 1,2,3 }).Select(item => new ReferenceClass());`

這樣會導致每次 foreach 時在 Select 才產出 `ReferenceClass` 物件，所以每次 foreach 都是拿到新一份的物件，第一次 foreach 改得值跟第二次 foreach 改得都不是同一份。

但是到這邊還不夠呀，我還是很好奇底層做了什麼事，為什麼會讓行為跟我原先預期的不相同!? 好吧，基礎不扎實才是原因呀...
我就去看了 List, Enumerable 的實作，不過要先提到 foreach 的運作方式。

## foreach 運作方式

[微軟這篇](https://docs.microsoft.com/en-us/archive/msdn-magazine/2017/april/essential-net-understanding-csharp-foreach-internals-and-custom-iterators-with-yield)講得很清楚，網路上也有很多文章可以看 foreach 運作原理。
不外乎就是先跟 `IEnumerable.GetEnumerator` 拿 Enumerator 然後每次先 `MoveNext()` 判斷有沒有下個值，並將 Current 換成下一個值，使用時拿 Current 使用。

如果 foreach 是直接使用 List, Array 都沒問題，因為每次 call by reference 都是拿到原本資料集的實體，但是被 Select 包裝過後就不一樣了。

## Linq Select

[Enumerable.Select](https://github.com/microsoft/referencesource/blob/17b97365645da62cf8a49444d979f94a59bbb155/System.Core/System/Linq/Enumerable.cs#L38) 的程式碼在這

```csharp
public static IEnumerable<TResult> Select<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector) {
  ...
  if (source is Iterator<TSource>) return ((Iterator<TSource>)source).Select(selector);
  if (source is TSource[]) return new WhereSelectArrayIterator<TSource, TResult>((TSource[])source, null, selector);
  if (source is List<TSource>) return new WhereSelectListIterator<TSource, TResult>((List<TSource>)source, null, selector);
  return new WhereSelectEnumerableIterator<TSource, TResult>(source, null, selector);
}
```

這邊會看到 Select 依 source 類型不同用不同實作包裝，每個類別都是實作 `Iterator<TResult>` 並把 [Select 的 lambda 行為存放在 selector delegate 變數](https://github.com/microsoft/referencesource/blob/17b97365645da62cf8a49444d979f94a59bbb155/System.Core/System/Linq/Enumerable.cs#L373)，如果你的 IEnumerable 被各種 Linq 包裝過都不會執行。

```csharp
class WhereSelectEnumerableIterator<TSource, TResult> : Iterator<TResult>
{
  IEnumerable<TSource> source;
  Func<TSource, bool> predicate;
  Func<TSource, TResult> selector;
  IEnumerator<TSource> enumerator;
  public WhereSelectEnumerableIterator(IEnumerable<TSource> source, Func<TSource, bool> predicate, Func<TSource, TResult> selector) {
      this.source = source;
      this.predicate = predicate;
      this.selector = selector;
  }
```

一直到最後使用到 `GetEnumerator` 的 `MoveNext()` [才是真正執行的地方](https://github.com/microsoft/referencesource/blob/17b97365645da62cf8a49444d979f94a59bbb155/System.Core/System/Linq/Enumerable.cs#L399)。

```csharp
public override bool MoveNext() {
  switch (state) {
      case 1:
          enumerator = source.GetEnumerator();
          state = 2;
          goto case 2;
      case 2:
          while (enumerator.MoveNext()) { // 這裡會逐步拆包
              TSource item = enumerator.Current;
              if (predicate == null || predicate(item)) {
                  current = selector(item); // 這裡執行各包的 select lambda
                  return true;
              }
          }
          Dispose();
          break;
  }
  return false;
}
```

## 結論

再次認知到不能信任 IEnumerable 的東西。

話說你看到 goto 了嘛.................