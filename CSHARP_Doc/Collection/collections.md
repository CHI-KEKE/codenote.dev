# 🎯 C# 集合處理完整指南

## 📖 目錄
- [一、集合常見問題](#一集合常見問題)
  - [1.1 Contains() 型別不匹配問題](#11-contains-型別不匹配問題)
    - [1.1.1 問題現象與原因分析](#111-問題現象與原因分析)
    - [1.1.2 解決方案](#112-解決方案)
  - [1.2 Select 可以利用 index](#12-select-可以利用-index)
    - [1.2.1 基本語法與應用](#121-基本語法與應用)
    - [1.2.2 實際應用範例](#122-實際應用範例)
  - [1.3 yield return 的延遲效果](#13-yield-return-的延遲效果)
    - [1.3.1 延遲執行的工作原理](#131-延遲執行的工作原理)
    - [1.3.2 實際範例](#132-實際範例)
    - [1.3.3 yield return demo2](#133-yield-return-demo2)

---

## 一、集合常見問題

### 1.1 Contains() 型別不匹配問題

在使用 C# 集合時，經常會遇到型別不匹配的問題，特別是在處理 nullable 型別時。讓我們透過一個實際案例來深入理解這個問題。

#### 1.1.1 問題現象與原因分析

##### 🚨 範例程式碼

```csharp
void Main()
{
    IEnumerable<PromotionEngineTypeDefEnum> _pointsPayDisplayPromotionTypes = new[] {
        PromotionEngineTypeDefEnum.DiscountReachPriceWithFreeGift,
        PromotionEngineTypeDefEnum.RewardReachPriceWithPoint2,
        PromotionEngineTypeDefEnum.RewardReachPriceWithRatePoint2,
        PromotionEngineTypeDefEnum.RewardReachPriceWithCoupon
    };
    
    var c = new a()
    {
        number = 1,
        b = PromotionEngineTypeDefEnum.DiscountByMemberWithPrice
    };
    
    // ❌ 這行會產生編譯錯誤
    _pointsPayDisplayPromotionTypes.Contains(c?.b).Dump();
}

public class a
{
    public int number { get; set; }
    public PromotionEngineTypeDefEnum b { get; set; }
}

public enum PromotionEngineTypeDefEnum
{
    DiscountReachPriceWithFreeGift,
    RewardReachPriceWithPoint2,
    RewardReachPriceWithRatePoint2,
    RewardReachPriceWithCoupon,
    DiscountByMemberWithPrice
}
```

##### 🔍 錯誤訊息分析

```
CS1929 'IEnumerable<UserQuery.PromotionEngineTypeDefEnum>' does not contain a definition for 'Contains' 
and the best extension method overload 'MemoryExtensions.Contains<UserQuery.PromotionEngineTypeDefEnum?>(ReadOnlySpan<UserQuery.PromotionEngineTypeDefEnum?>, UserQuery.PromotionEngineTypeDefEnum?)' 
requires a receiver of type 'System.ReadOnlySpan<UserQuery.PromotionEngineTypeDefEnum?>'
```

##### 📊 型別不匹配分析

| 項目 | 型別 | 說明 |
|------|------|------|
| `_pointsPayDisplayPromotionTypes` | `IEnumerable<PromotionEngineTypeDefEnum>` | 非 nullable 的 enum 集合 |
| `c?.b` | `PromotionEngineTypeDefEnum?` | nullable enum（因為使用了 ?. 運算子） |
| `Contains()` 預期參數 | `PromotionEngineTypeDefEnum` | 非 nullable 型別 |

> **🎯 問題核心**
> 
> 1. `_pointsPayDisplayPromotionTypes` 的型別是 `IEnumerable<PromotionEngineTypeDefEnum>`（非 nullable）
> 2. `c?.b` 的型別是 `PromotionEngineTypeDefEnum?`（nullable enum）
> 3. `Contains()` 方法需要傳入非 nullable 的 `PromotionEngineTypeDefEnum`，所以型別不匹配

#### 1.1.2 解決方案

##### ✅ 方案一：使用 Null-Conditional 運算子檢查

```csharp
void Main()
{
    IEnumerable<PromotionEngineTypeDefEnum> _pointsPayDisplayPromotionTypes = new[] {
        PromotionEngineTypeDefEnum.DiscountReachPriceWithFreeGift,
        PromotionEngineTypeDefEnum.RewardReachPriceWithPoint2,
        PromotionEngineTypeDefEnum.RewardReachPriceWithRatePoint2,
        PromotionEngineTypeDefEnum.RewardReachPriceWithCoupon
    };
    
    var c = new a()
    {
        number = 1,
        b = PromotionEngineTypeDefEnum.DiscountByMemberWithPrice
    };
    
    // ✅ 解決方案 1：確保 c 不為 null 再檢查
    bool result = c != null && _pointsPayDisplayPromotionTypes.Contains(c.b);
    result.Dump();
}
```

##### ✅ 方案二：使用 HasValue 和 Value 屬性

```csharp
void Main()
{
    // ... 同上設定 ...
    
    // ✅ 解決方案 2：直接存取 Value（如果確定不為 null）
    PromotionEngineTypeDefEnum? nullableEnum = c?.b;
    bool result = nullableEnum.HasValue && _pointsPayDisplayPromotionTypes.Contains(nullableEnum.Value);
    result.Dump();
}
```

##### ✅ 方案三：使用模式匹配

```csharp
void Main()
{
    // ... 同上設定 ...
    
    // ✅ 解決方案 3：使用 is 模式匹配
    bool result = c?.b is PromotionEngineTypeDefEnum enumValue && 
                  _pointsPayDisplayPromotionTypes.Contains(enumValue);
    result.Dump();
}
```

---

### 1.2 Select 可以利用 index

#### 1.2.1 實際應用範例

以下範例展示如何在分組處理中使用 `Select` 的索引功能來為每個群組內的元素分配序號：

```csharp
void Main()
{
    var ccc = new List<PromotionRewardAmortizationBaseEntity>()
    {
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "AAA", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "AAA", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "AAA", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "AAA", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "bbb", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "bbb", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "bbb", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "bbb", Seq = 0},
        new PromotionRewardAmortizationBaseEntity{TradesOrderSlaveCode = "bbb", Seq = 0},
    };

    // 依據 TradesOrderSlaveCode 分組，處理 seq
    var orderSlaveCodeGroups = ccc.GroupBy(x => x.TradesOrderSlaveCode);
    foreach (var group in orderSlaveCodeGroups)
    {
        group.Select((entity, index) =>
        {
            entity.Seq = index;  // 使用 index 設定序號
            return entity;
        }).ToList();
    }

    // 顯示結果
    ccc.Select((text) =>
    {
        Console.WriteLine($"{text.TradesOrderSlaveCode} and {text.Seq}");
        return text;
    }).ToList();
}

/// <summary>
/// 攤提資訊Base Entity
/// </summary>
public class PromotionRewardAmortizationBaseEntity
{
    /// <summary>
    /// TS Code
    /// </summary>
    public string TradesOrderSlaveCode { get; set; }

    /// <summary>
    /// 流水號
    /// </summary>
    public int Seq { get; set; }
}
```

##### 📊 輸出結果

```
AAA and 0
AAA and 1
AAA and 2
AAA and 3
bbb and 0
bbb and 1
bbb and 2
bbb and 3
bbb and 4
```

---

### 1.3 yield return 的延遲效果

`yield return` 是 C# 中一個強大的功能，它可以建立一個迭代器 (Iterator)，實現延遲執行 (Lazy Evaluation)。這意味著數據不會在方法被呼叫時立即產生，而是在實際需要時才逐一產生。

#### 1.3.1 延遲執行的工作原理

##### 🔄 迭代器基本概念

使用 `yield return` 的方法會返回一個 `IEnumerable<T>`，但實際的資料產生是按需進行的：

```csharp
// 傳統方式 - 立即執行
public List<int> GetNumbersTraditional()
{
    var numbers = new List<int>();
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"產生數字 {i}");
        numbers.Add(i);
    }
    return numbers; // 所有數字立即產生
}

// yield return 方式 - 延遲執行
public IEnumerable<int> GetNumbersWithYield()
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"產生數字 {i}");
        yield return i; // 只在需要時產生
    }
}
```

##### ⚡ 執行時機對比

```csharp
void Main()
{
    Console.WriteLine("=== 傳統方式 ===");
    var traditionalNumbers = GetNumbersTraditional(); // 這裡會立即印出所有訊息
    Console.WriteLine("方法已返回");
    
    Console.WriteLine("\n=== yield return 方式 ===");
    var yieldNumbers = GetNumbersWithYield(); // 這裡不會有任何輸出
    Console.WriteLine("方法已返回");
    
    Console.WriteLine("\n開始迭代:");
    foreach (var num in yieldNumbers) // 這裡才開始產生數字
    {
        Console.WriteLine($"取得: {num}");
    }
}
```

#### 1.3.2 實際範例

以下範例展示了 `yield return` 在產品資料處理中的延遲執行效果：

```csharp
void Main()
{
    var store = new DemoStore();
    var productList = store.GetProducts();
    
    // 注意：這行被註解掉了，如果執行會觸發所有產品的產生
    // var total = productList.Count();
    // $"Total products {total}".Dump();

    // 只有在實際迭代時，產品才會被逐一產生
    foreach (var product in productList)
    {
        $"{product.Id}，{product.Name}".Dump();
    }
}

public class DemoStore
{
    public IEnumerable<Product> GetProducts()
    {
        int count = 5;
        for (int i = 0; i < count; i++)
        {
            $"{i}產生Product中".Dump(); // 只有在需要時才會執行
            yield return new Product()
            {
                Id = i,
                Name = $"Pro {i}"
            };
        }
    }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

##### 📊 預期輸出結果

```
0產生Product中
0，Pro 0
1產生Product中
1，Pro 1
2產生Product中
2，Pro 2
3產生Product中
3，Pro 3
4產生Product中
4，Pro 4
```

##### 🔍 關鍵觀察點

**延遲執行的特徵：**
1. **方法呼叫時不執行**：`store.GetProducts()` 被呼叫時，方法內的程式碼並未執行
2. **按需產生**：只有在 `foreach` 迭代時，每次需要下一個元素才會執行 `yield return`
3. **逐一處理**：每次產生一個元素，不會一次性產生所有元素

**效能優勢：**
- 🚀 **記憶體效率**：不需要在記憶體中同時儲存所有元素
- ⏱️ **即時回應**：第一個元素可以立即返回，不需等待所有元素產生
- 🔄 **可中斷處理**：可以在任何時候停止迭代，已產生的元素不會浪費

#### 1.3.3 yield return demo2

以下範例更清楚地展示了 `yield return` 的延遲執行特性，以及重複迭代時的行為：

```csharp
void Main()
{
    var store = new DemoStore();
    Console.WriteLine("1. 調用 GetProducts()");
    var productList = store.GetProducts();

    Console.WriteLine("\n2. 什麼都不做，只是持有 IEnumerable");

    Console.WriteLine("\n3. 開始計算 Count");
    var count = productList.Count();

    Console.WriteLine("\n4. 開始 foreach");
    foreach (var p in productList)
    {
        p.Id.Dump();
    }
}

public class DemoStore
{
    public IEnumerable<Product> GetProducts()
    {
        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine($"生成產品 {i}");
            yield return new Product { Id = i };
        }
    }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

##### 📊 預期輸出結果

```
1. 調用 GetProducts()

2. 什麼都不做，只是持有 IEnumerable

3. 開始計算 Count
生成產品 0
生成產品 1
生成產品 2

4. 開始 foreach
生成產品 0
0
生成產品 1
1
生成產品 2
2
```

##### 🔍 重要觀察點

**步驟分析：**
1. **第一步**：`GetProducts()` 被呼叫，但沒有任何輸出，因為方法內容尚未執行
2. **第二步**：只是持有 `IEnumerable` 參考，依然沒有執行
3. **第三步**：呼叫 `Count()` 時，迭代器開始執行，產生所有產品來計算數量
4. **第四步**：再次 `foreach` 時，迭代器重新執行，又產生了一次所有產品

**⚠️ 重複執行的陷阱：**
- 每次對 `IEnumerable<T>` 進行迭代時，`yield return` 方法都會重新執行
- `Count()` 和 `foreach` 各自觸發了一次完整的產品產生過程
- 這意味著如果產生過程有副作用（如資料庫查詢、日誌記錄等），會被執行多次

##### 💡 避免重複執行的解決方案

**方案一：具體化為 List**
```csharp
void Main()
{
    var store = new DemoStore();
    Console.WriteLine("1. 調用 GetProducts() 並立即具體化");
    var productList = store.GetProducts().ToList(); // 立即執行並儲存結果

    Console.WriteLine("\n2. 計算 Count (不會重新執行)");
    var count = productList.Count;

    Console.WriteLine("\n3. 開始 foreach (不會重新執行)");
    foreach (var p in productList)
    {
        p.Id.Dump();
    }
}
```

**預期輸出（具體化後）：**
```
1. 調用 GetProducts() 並立即具體化
生成產品 0
生成產品 1
生成產品 2

2. 計算 Count (不會重新執行)

3. 開始 foreach (不會重新執行)
0
1
2
```

**方案二：快取結果的迭代器**
```csharp
public class CachedDemoStore
{
    private List<Product> _cache;

    public IEnumerable<Product> GetProducts()
    {
        if (_cache == null)
        {
            Console.WriteLine("首次產生產品，建立快取");
            _cache = GenerateProducts().ToList();
        }
        else
        {
            Console.WriteLine("使用快取的產品");
        }
        
        return _cache;
    }

    private IEnumerable<Product> GenerateProducts()
    {
        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine($"生成產品 {i}");
            yield return new Product { Id = i };
        }
    }
}
```

##### 🎯 最佳實踐建議

**何時使用 `yield return`：**
- ✅ 處理大型資料集，記憶體有限制時
- ✅ 資料產生成本很高，可能不需要全部資料時
- ✅ 實作自訂的 LINQ 操作符時
- ✅ 需要無限序列或延遲計算時

**何時避免 `yield return`：**
- ❌ 需要頻繁迭代相同資料時
- ❌ 產生過程有副作用（如資料庫操作）時
- ❌ 需要隨機存取或知道集合大小時
- ❌ 資料量小且產生成本低時