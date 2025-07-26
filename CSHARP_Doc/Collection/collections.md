# 🎯 C# 集合處理完整指南

## 📖 目錄
- [一、集合常見問題](#一集合常見問題)
  - [1.1 Contains() 型別不匹配問題](#11-contains-型別不匹配問題)
    - [1.1.1 問題現象與原因分析](#111-問題現象與原因分析)
    - [1.1.2 解決方案](#112-解決方案)
  - [1.2 Select 可以利用 index](#12-select-可以利用-index)
    - [1.2.1 基本語法與應用](#121-基本語法與應用)
    - [1.2.2 實際應用範例](#122-實際應用範例)

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