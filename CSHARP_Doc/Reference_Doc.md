# C# Reference Type 實務完整指南

## 目錄
1. [參考型別基礎概念](#參考型別基礎概念)
2. [淺拷貝與深拷貝](#淺拷貝與深拷貝)
3. [實務常見問題](#實務常見問題)
4. [最佳實踐](#最佳實踐)
5. [進階技巧](#進階技巧)

---

## 參考型別基礎概念

### 什麼是參考型別 (Reference Type)

參考型別在記憶體中存儲的是物件的記憶體位址，而不是物件本身的值。這意味著多個變數可以指向同一個物件。

```csharp
// 參考型別範例
public class Promotion
{
    public int PromotionId { get; set; }
    public string Name { get; set; }
    public DateTime StartDate { get; set; }
    public decimal DiscountRate { get; set; }
}

// 建立物件
var promotion1 = new Promotion { PromotionId = 1, Name = "新年特價" };
var promotion2 = promotion1; // promotion2 指向同一個物件

// 修改其中一個會影響另一個
promotion2.Name = "春節特價";
Console.WriteLine(promotion1.Name); // 輸出: "春節特價"
```

### 常見的參考型別

```csharp
// 類別 (Class)
public class Product { }

// 陣列 (Array)
int[] numbers = new int[10];

// 集合 (Collections)
List<string> items = new List<string>();
Dictionary<int, string> lookup = new Dictionary<int, string>();

// 字串 (String) - 特殊的參考型別
string text = "Hello World";

// 委派 (Delegate)
Action<string> action = Console.WriteLine;
```

---

## 淺拷貝與深拷貝

### 淺拷貝問題實例

這是您提到的經典問題：**修改淺拷貝資料也會改到原始的**

```csharp
public class PromotionExample
{
    public static void DemonstrateProblem()
    {
        var all = new List<Promotion>
        {
            new Promotion { PromotionId = 1, Name = "A" },
            new Promotion { PromotionId = 2, Name = "B" },
        };
    
        // 淺拷貝：只複製了 List，但 Promotion 物件仍是相同的參考
        var filtered = all.Where(p => p.PromotionId == 1).ToList();
        
        // 修改 filtered 中的物件，會影響到 all 中的原始物件
        filtered[0].Name = "Hello";
    
        Console.WriteLine(all[0].Name); // 輸出: "Hello" (原始資料被修改了!)
    }
}
```

### 為什麼會發生這個問題？

```csharp
// 視覺化記憶體結構
/*
原始列表 all:
┌─────────────┐
│  List<T>    │ ──┐
└─────────────┘   │
                  ▼
┌─────────────┐   ┌─────────────┐
│ Promotion1  │   │ Promotion2  │
│ Id: 1       │   │ Id: 2       │
│ Name: "A"   │   │ Name: "B"   │
└─────────────┘   └─────────────┘
      ▲
      │ 相同的參考
┌─────────────┐
│  List<T>    │ ──┘
│ (filtered)  │
└─────────────┘
*/
```

### 解決方案

#### 1. 實作深拷貝

```csharp
public class Promotion : ICloneable
{
    public int PromotionId { get; set; }
    public string Name { get; set; }
    public DateTime StartDate { get; set; }
    public decimal DiscountRate { get; set; }
    
    // 實作深拷貝
    public object Clone()
    {
        return new Promotion
        {
            PromotionId = this.PromotionId,
            Name = this.Name,
            StartDate = this.StartDate,
            DiscountRate = this.DiscountRate
        };
    }
    
    // 泛型版本
    public Promotion DeepCopy()
    {
        return new Promotion
        {
            PromotionId = this.PromotionId,
            Name = this.Name,
            StartDate = this.StartDate,
            DiscountRate = this.DiscountRate
        };
    }
}

// 使用深拷貝解決問題
public static void SolvedExample()
{
    var all = new List<Promotion>
    {
        new Promotion { PromotionId = 1, Name = "A" },
        new Promotion { PromotionId = 2, Name = "B" },
    };

    // 建立新的物件實例
    var filtered = all.Where(p => p.PromotionId == 1)
                     .Select(p => p.DeepCopy())
                     .ToList();
    
    filtered[0].Name = "Hello";
    
    Console.WriteLine(all[0].Name);      // 輸出: "A" (原始資料未被修改)
    Console.WriteLine(filtered[0].Name); // 輸出: "Hello"
}
```

#### 2. 使用記錄型別 (Record Type) - C# 9.0+

```csharp
public record PromotionRecord(int PromotionId, string Name, DateTime StartDate, decimal DiscountRate);

public static void RecordExample()
{
    var all = new List<PromotionRecord>
    {
        new(1, "A", DateTime.Now, 0.1m),
        new(2, "B", DateTime.Now, 0.2m),
    };

    // Record 提供自動的 with 表達式來建立新實例
    var filtered = all.Where(p => p.PromotionId == 1)
                     .Select(p => p with { Name = "Hello" })
                     .ToList();
    
    Console.WriteLine(all[0].Name);      // 輸出: "A"
    Console.WriteLine(filtered[0].Name); // 輸出: "Hello"
}
```

#### 3. 使用 AutoMapper 進行物件複製

```csharp
// 安裝 NuGet 套件: AutoMapper
using AutoMapper;

public static void AutoMapperExample()
{
    var config = new MapperConfiguration(cfg => {
        cfg.CreateMap<Promotion, Promotion>();
    });
    var mapper = config.CreateMapper();

    var all = new List<Promotion>
    {
        new Promotion { PromotionId = 1, Name = "A" },
        new Promotion { PromotionId = 2, Name = "B" },
    };

    var filtered = all.Where(p => p.PromotionId == 1)
                     .Select(p => mapper.Map<Promotion>(p))
                     .ToList();
    
    filtered[0].Name = "Hello";
    
    Console.WriteLine(all[0].Name);      // 輸出: "A"
    Console.WriteLine(filtered[0].Name); // 輸出: "Hello"
}
```

---

## 實務常見問題

### 1. 集合操作的參考陷阱

```csharp
public class CollectionReferenceTrap
{
    public static void Example1()
    {
        var originalList = new List<int> { 1, 2, 3 };
        var newList = originalList; // 參考複製，不是值複製
        
        newList.Add(4);
        Console.WriteLine(originalList.Count); // 輸出: 4 (原始列表也被修改了)
    }
    
    public static void Example2()
    {
        var originalList = new List<int> { 1, 2, 3 };
        var newList = new List<int>(originalList); // 建立新的 List 實例
        
        newList.Add(4);
        Console.WriteLine(originalList.Count); // 輸出: 3 (原始列表未被修改)
    }
}
```

### 2. 方法參數的參考傳遞

```csharp
public class ParameterReference
{
    public static void ModifyObject(Promotion promotion)
    {
        promotion.Name = "Modified"; // 會修改原始物件
    }
    
    public static void ReassignObject(Promotion promotion)
    {
        promotion = new Promotion { Name = "New Object" }; // 不會影響原始物件
    }
    
    public static void ModifyWithRef(ref Promotion promotion)
    {
        promotion = new Promotion { Name = "New Object" }; // 會替換原始物件參考
    }
    
    public static void Example()
    {
        var original = new Promotion { PromotionId = 1, Name = "Original" };
        
        ModifyObject(original);
        Console.WriteLine(original.Name); // 輸出: "Modified"
        
        ReassignObject(original);
        Console.WriteLine(original.Name); // 輸出: "Modified" (未改變)
        
        ModifyWithRef(ref original);
        Console.WriteLine(original.Name); // 輸出: "New Object"
    }
}
```

### 3. 空參考問題

```csharp
public class NullReferenceHandling
{
    public static void SafeExample()
    {
        Promotion promotion = null;
        
        // 使用 null 條件運算子
        var name = promotion?.Name ?? "未知";
        
        // 使用 null 合併運算子
        promotion ??= new Promotion { Name = "預設" };
        
        // 模式匹配檢查
        if (promotion is { Name: string promotionName })
        {
            Console.WriteLine($"促銷名稱: {promotionName}");
        }
    }
}
```

---

## 最佳實踐

### 1. 不可變物件設計

```csharp
public class ImmutablePromotion
{
    public int PromotionId { get; }
    public string Name { get; }
    public DateTime StartDate { get; }
    public decimal DiscountRate { get; }
    
    public ImmutablePromotion(int promotionId, string name, DateTime startDate, decimal discountRate)
    {
        PromotionId = promotionId;
        Name = name ?? throw new ArgumentNullException(nameof(name));
        StartDate = startDate;
        DiscountRate = discountRate;
    }
    
    // 建立修改版本的方法
    public ImmutablePromotion WithName(string newName)
    {
        return new ImmutablePromotion(PromotionId, newName, StartDate, DiscountRate);
    }
    
    public ImmutablePromotion WithDiscountRate(decimal newRate)
    {
        return new ImmutablePromotion(PromotionId, Name, StartDate, newRate);
    }
}
```

### 2. 防禦性複製

```csharp
public class PromotionService
{
    private readonly List<Promotion> _promotions = new();
    
    // 不要直接回傳內部集合
    public IReadOnlyList<Promotion> GetAllPromotions()
    {
        return _promotions.ToList(); // 回傳複製品
    }
    
    // 或使用唯讀介面
    public IReadOnlyList<Promotion> GetPromotionsReadOnly()
    {
        return _promotions.AsReadOnly();
    }
    
    // 接受參數時進行複製
    public void UpdatePromotions(IEnumerable<Promotion> promotions)
    {
        _promotions.Clear();
        _promotions.AddRange(promotions.Select(p => p.DeepCopy()));
    }
}
```

### 3. 使用結構型別來避免參考問題

```csharp
// 對於簡單的資料結構，考慮使用 struct
public struct PromotionInfo
{
    public int PromotionId { get; }
    public string Name { get; }
    public decimal DiscountRate { get; }
    
    public PromotionInfo(int promotionId, string name, decimal discountRate)
    {
        PromotionId = promotionId;
        Name = name;
        DiscountRate = discountRate;
    }
}

// 或使用 readonly struct
public readonly struct ReadOnlyPromotionInfo
{
    public int PromotionId { get; }
    public string Name { get; }
    
    public ReadOnlyPromotionInfo(int promotionId, string name)
    {
        PromotionId = promotionId;
        Name = name;
    }
}
```

---

## 進階技巧

### 1. 自訂相等比較

```csharp
public class Promotion : IEquatable<Promotion>
{
    public int PromotionId { get; set; }
    public string Name { get; set; }
    
    public bool Equals(Promotion other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;
        return PromotionId == other.PromotionId && Name == other.Name;
    }
    
    public override bool Equals(object obj)
    {
        return Equals(obj as Promotion);
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(PromotionId, Name);
    }
    
    public static bool operator ==(Promotion left, Promotion right)
    {
        return Equals(left, right);
    }
    
    public static bool operator !=(Promotion left, Promotion right)
    {
        return !Equals(left, right);
    }
}
```

### 2. 使用 ICloneable 與泛型複製

```csharp
public interface IDeepCloneable<T>
{
    T DeepClone();
}

public class Promotion : IDeepCloneable<Promotion>
{
    public int PromotionId { get; set; }
    public string Name { get; set; }
    public List<string> Tags { get; set; } = new();
    
    public Promotion DeepClone()
    {
        return new Promotion
        {
            PromotionId = this.PromotionId,
            Name = this.Name,
            Tags = new List<string>(this.Tags) // 深拷貝集合
        };
    }
}

// 擴充方法
public static class CloneExtensions
{
    public static List<T> DeepCloneList<T>(this IEnumerable<T> source) 
        where T : IDeepCloneable<T>
    {
        return source.Select(item => item.DeepClone()).ToList();
    }
}
```

### 3. 序列化深拷貝

```csharp
using System.Text.Json;

public static class DeepCopyHelper
{
    public static T DeepCopy<T>(T obj) where T : class
    {
        if (obj == null) return null;
        
        var json = JsonSerializer.Serialize(obj);
        return JsonSerializer.Deserialize<T>(json);
    }
}

// 使用範例
var original = new Promotion { PromotionId = 1, Name = "原始" };
var copy = DeepCopyHelper.DeepCopy(original);
copy.Name = "複製品";
Console.WriteLine(original.Name); // 輸出: "原始"
```

---