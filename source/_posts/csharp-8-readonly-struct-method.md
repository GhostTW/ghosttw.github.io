---
title: C# 8 readonly struct method 會有 defensive copy
date: 2020-01-07 23:47:42
tags: ['dotnet', 'csharp', 'c#8']
categories:
---

今天抽空在看 C# 8 的新特性，有一個 readonly method 讓我有了興趣，也順便試了新功能不同的組合應用。

## Readonly Method

這個功能主要是限制及提示該 Method 的應用不會更改到該 instance 的值。

我就來試看看了先是加在 class 上的 method，結果就出現了 compile error `CS0106 The modifier 'readonly' is not valid for this item`

原來這東西不能用在 class 上，文件沒有好好細讀就想試...

另外這個功能與 C# 7.2 的 ref readonly 是不一樣的

### Readonly Method in Class

```csharp
void Main()
{
  var t = new Test();
  t.Number = 1;
  Console.WriteLine(t.ToString());
  Console.WriteLine(t.ToString());
}

public class Test
{
  public int Number;
  public  int Square => Number++;
  
  public readonly override string ToString() // CS0106 The modifier 'readonly' is not valid for this item
  {
    return Square.ToString();
  }
}
```

### Readonly Method in Struct

那我們改到 struct 來試看看，不過在使用新的 readonly method 前，我們先來看這個例子，你們覺得會印出什麼值?

```csharp
void Main()
{
  var t = new Test();
  t.Number = 1;
  Console.WriteLine(t.ToString());
  Console.WriteLine(t.ToString());
}

public struct Test
{
  public int Number;
  public int Square => Number++;
  
  public override string ToString()
  {
    return Square.ToString();
  }
}
```

```txt
1
2
```

那使用 readonly 之後呢?

```csharp
void Main()
{
  var t = new Test();
  t.Number = 1;
  Console.WriteLine(t.ToString());
  Console.WriteLine(t.ToString());
}

public struct Test
{
  public int Number;
  public int Square => Number++;
  
  public readonly override string ToString()
  {
    return Square.ToString();
  }
}
```

```txt
1
1
```

readonly method 因為無法判斷所使用的欄位有沒有去改到值，無論你是不是只有用 property 而且只實作 get 而已，它還是只能給你個提醒

{% asset_img warning-readonly.png warning-readonly %}

然後在 IL code 上面做一個 defensive copy 將整個 struct 再複製一份到 stack 上面使用 `IL_0001:  ldobj     UserQuery.Test`
這樣的結果會讓每次執行 ToString() 都使用一份新的實體，而不會去改到原本的實體，所以最後印出來兩次都是 `1`。

```il
Test.ToString:
IL_0000:  ldarg.0
IL_0001:  ldobj       UserQuery.Test // defensive copy
IL_0006:  stloc.0
IL_0007:  ldloca.s    00
IL_0009:  call        UserQuery+Test.get_Square
IL_000E:  stloc.1
IL_000F:  ldloca.s    01
IL_0011:  call        System.Int32.ToString
IL_0016:  ret  
```

那我們該如何避免這個狀況呢?

我們依照 Compiler 的提示將使用的 readonly 補上，並換個情境不去修改自己的值，如果實務上有需要的話，就不應該使用 readonly method。

```csharp
void Main()
{
  var t = new Test();
  t.Number = 1;
  t.ToString().Dump();
  t.ToString().Dump();
}

public struct Test
{
  public int Number;
  public readonly int Square => Number + Number;
  
  public readonly override string ToString()
  {
    return Square.ToString();
  }
}
```

```il
Test.ToString:
IL_0000:  ldarg.0
IL_0001:  call        UserQuery+Test.get_Square
IL_0006:  stloc.0
IL_0007:  ldloca.s    00
IL_0009:  call        System.Int32.ToString
IL_000E:  ret
```

這樣就沒有 defensive copy 了!

## References

[code](https://github.com/GhostTW/demos/tree/master/ReadonlyStructMethod)
[csharp-8#readonly-members](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#readonly-members)
[opcodes.ldobj](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.opcodes.ldobj?view=netcore-3.1)
[7.2readonly-ref](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-7.2/readonly-ref#readonly-ref-locals)