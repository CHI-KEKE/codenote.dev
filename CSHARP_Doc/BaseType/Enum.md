# 🌺 C# Enum 完整指南 - 從基礎到進階

## 📋 目錄

1. [🔰 基礎操作](#-基礎操作)
2. [🏴 Flags 屬性詳解](#-flags-屬性詳解)
3. [🔍 Enum 解析與驗證](#-enum-解析與驗證)
4. [📊 Enum 資訊取得](#-enum-資訊取得)
5. [⚙️ 實用工具方法](#️-實用工具方法)
6. [🌐 JSON 序列化](#-json-序列化)
7. [⚠️ 常見問題與解決方案](#️-常見問題與解決方案)

---

## 🔰 基礎操作

### GetValues + Cast 組合技

```csharp
void Main()
{
    var values = Enum.GetValues(typeof(PromotionEngineTypeEnum));
    values.Cast<PromotionEngineTypeEnum>().Select(x => x.ToString()).Dump();
    
    // 💡 說明：GetValues 回傳的是 Array，需要 Cast 轉型才能使用泛型方法
}

public class Promotion
{
    public int PromotionId { get; set; }
    public string Name { get; set; }
    public PromotionEngineTypeEnum PromotionEngineType { get; set; }
}

/// <summary>
/// 活動引擎類型列舉
/// </summary>
[Flags]
public enum PromotionEngineTypeEnum
{
    /// <summary>
    /// 第N件打折
    /// </summary>
    DiscountNthPieceWithRate = 1,

    /// <summary>
    /// 第N件固定價
    /// </summary>
    DiscountNthPieceWithPrice = 2,

    /// <summary>
    /// 全部KOL推薦活動
    /// </summary>
    AllPromoCodeKOL = PromoCodeKOLNoDiscount | PromoCodeKOLReachPriceAmount | PromoCodeKOLReachPriceRate,

    /// <summary>
    /// 全部登記活動
    /// </summary>
    AllRegister = RegisterReachPiece | RegisterReachPrice,

    /// <summary>
    /// 全部的贈品活動
    /// </summary>
    /// <remarks>
    /// ⚠️ 注意：只有一種會造成後續 GetFlagValue 後 toString 都變成 AllGift，
    /// 造成 FreeGiftReachPriceWithAmount 都撈不到資料
    /// </remarks>
    AllGift = FreeGiftReachPriceWithAmount | DiscountReachPieceWithFreeGift | DiscountReachPriceWithFreeGift,

    /// <summary>
    /// 全部
    /// </summary>
    All = AllDiscount | AllReward | AllPromoCode | AllGift | AllRegister
}
```

---

## 🏴 Flags 屬性詳解

### 基本 Flags Enum 設計

```csharp
[Flags]
public enum SalesDayOfWeekEnum : byte
{
    Monday = 1,         // 0000 0001
    Tuesday = 2,        // 0000 0010
    Wednesday = 4,      // 0000 0100
    Thursday = 8,       // 0000 1000
    Friday = 0x10,      // 0001 0000 (16)
    Saturday = 0x20,    // 0010 0000 (32)
    Sunday = 0x40       // 0100 0000 (64)
}
```

> 💡 **小知識**：byte 是 8 位元（0~255），最多可存 8 個標誌（目前只用了 7 個，還可以加一個）。

### 🎭 Flags 操作範例

#### 1. 聯集操作 (|)
```csharp
// 🌟 建立工作日組合
SalesDayOfWeekEnum workDays = SalesDayOfWeekEnum.Monday | 
                              SalesDayOfWeekEnum.Wednesday | 
                              SalesDayOfWeekEnum.Friday;

Console.WriteLine(workDays); 
// 有 [Flags] 輸出: "Monday, Wednesday, Friday"
// 沒有 [Flags] 輸出: "21"（對應數值）
```

#### 2. 檢查是否包含 (HasFlag)
```csharp
bool isFridayIncluded = workDays.HasFlag(SalesDayOfWeekEnum.Friday);
Console.WriteLine(isFridayIncluded); // True
```

#### 3. 交集操作 (&)
```csharp
var intersection = workDays & SalesDayOfWeekEnum.Friday;
// 如果 workDays 包含 Friday，則 intersection 為 Friday，否則為 0
```

---

## 🔍 Enum 解析與驗證

### IsDefined - 驗證值是否有效

```csharp
SalesDayOfWeekEnum days = SalesDayOfWeekEnum.Monday;

if (Enum.IsDefined(typeof(SalesDayOfWeekEnum), days))
{
    Console.WriteLine("✅ 有效的 Enum 值");
}
else
{
    Console.WriteLine("❌ 無效的 Enum 值");
}
```

### Parse - 字串轉 Enum

```csharp
// 🎯 新版本寫法（推薦）
var dayOfWeekEnum = Enum.Parse<DayOfWeek>("Monday");

// 🔄 舊版本寫法
DayOfWeek oldWayEnum = (DayOfWeek)Enum.Parse(typeof(DayOfWeek), "Monday");
```

### TryParse - 安全解析

```csharp
// 🛡️ 舊框架安全解析作法
PromotionScopeModifyTypeDefEnum modifyType;
if (Enum.TryParse(entity.ModifyType, true, out modifyType) == false)
{
    throw new ApplicationException("無效的操作類型");
}

// 🚀 新版本寫法
if (Enum.TryParse<PromotionScopeModifyTypeDefEnum>(entity.ModifyType, true, out var newModifyType))
{
    // 解析成功
}
```

---

## 📊 Enum 資訊取得

### GetName - 取得單一名稱

```csharp
DayOfWeek today = DayOfWeek.Monday;
string name = Enum.GetName(today);
Console.WriteLine(name); // "Monday"

// ⚠️ 注意：GetName 只能用於單一值
SalesDayOfWeekEnum combined = SalesDayOfWeekEnum.Monday | SalesDayOfWeekEnum.Tuesday;
string combinedName = Enum.GetName(typeof(SalesDayOfWeekEnum), combined);
Console.WriteLine(combinedName); // null (因為 12 不是單一定義的值)
```

> 🔍 **重要提醒**：`Enum.GetName()` 本質用途是找出「這個 value 對應的單一 Enum 名稱」。如果傳入的是 Flags 組合值，會回傳 null。

### GetNames - 取得所有名稱

```csharp
// 🎨 在 MVC 中建立下拉選單
public enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}

// Controller 內
public IActionResult CreateOrder()
{
    ViewBag.OrderStatusList = Enum.GetNames(typeof(OrderStatus));
    return View();
}
```

```html
<!-- View 中使用 -->
<select name="orderStatus">
    @foreach (var status in ViewBag.OrderStatusList)
    {
        <option value="@status">@status</option>
    }
</select>
```

---

## ⚙️ 實用工具方法

### 🔧 取得 Flags 的所有組成項目

```csharp
/// <summary>
/// 取得 Flags Enum 的所有組成項目
/// </summary>
/// <typeparam name="T">Enum 類型</typeparam>
/// <param name="input">輸入的 Flags Enum</param>
/// <returns>包含的所有 Flags</returns>
public static IEnumerable<T> GetFlags<T>(Enum input)
{
    var flags = new List<Enum>();
    foreach (Enum value in Enum.GetValues(input.GetType()))
    {
        if (input.HasFlag(value))
        {
            flags.Add(value);
        }
    }

    return flags.Cast<T>();
}

// 🎮 使用範例
var workDays = SalesDayOfWeekEnum.Monday | SalesDayOfWeekEnum.Friday;
var individualFlags = GetFlags<SalesDayOfWeekEnum>(workDays);
// 結果：[Monday, Friday]
```

---

## 🌐 JSON 序列化

### 字串型 JSON 序列化

```csharp
[JsonConverter(typeof(JsonStringEnumConverter))]
public enum ApiResponseStatus
{
    Success,
    Error,
    Warning
}

// 🔄 或者使用舊版 Newtonsoft.Json
[JsonConverter(typeof(StringEnumConverter))]
public enum LegacyStatus
{
    Active,
    Inactive
}
```

### JSON 處理範例

```csharp
// 🎯 處理原始 JSON 物件中的 Flags
var originalRuleObject = JObject.Parse(entity.Rule);
jsonRuleObject.MatchedSalesChannels = (int)originalRuleObject[nameof(RewardReachPriceWithPoint.MatchedSalesChannels)];

// 💡 提醒：掛 [Flags] 的 Enum 在 JSON 序列化時會變成字串組合
```

---

## ⚠️ 常見問題與解決方案

### 問題 1：JSON 反序列化錯誤

```
❌ 錯誤訊息：
"The JSON value could not be converted to System.String. 
Path: $.promotionRules[0].type | LineNumber: 0 | BytePositionInLine: 45."
```

**🧪 原因分析：**
- .NET 預期 JSON 傳進來的是數字（int）對應的 enum 數值
- enum 本質上就是特殊包裝的整數
- System.Text.Json 預設期望數字格式

**🐧 解決方案：**
```csharp
[JsonConverter(typeof(JsonStringEnumConverter))]
public enum PromotionType
{
    Discount,
    Reward,
    Gift
}
```

### 問題 2：預設值為 0 的問題

```csharp
// 🌋 不建議的設計 - 沒有明確定義 0 值
public enum BadEnum
{
    FirstValue = 1,  // 如果忘記設定，預設會是 0
    SecondValue = 2
}

// ✅ 建議的設計 - 明確定義 0 值
public enum GoodEnum
{
    None = 0,        // 明確定義預設值
    FirstValue = 1,
    SecondValue = 2
}
```

### 問題 3：Flags Enum 的 ToString() 行為

```csharp
[Flags]
public enum Features
{
    None = 0,
    FeatureA = 1,
    FeatureB = 2,
    FeatureC = 4,
    AllFeatures = FeatureA | FeatureB | FeatureC  // 值為 7
}

var combined = Features.FeatureA | Features.FeatureB;  // 值為 3
Console.WriteLine(combined.ToString());  // 輸出: "FeatureA, FeatureB"

var all = Features.AllFeatures;
Console.WriteLine(all.ToString());  // 輸出: "AllFeatures" (因為剛好等於定義的值)
```

---

## 🎉 最佳實踐建議

### ✅ 做對的事

1. **明確定義 0 值**：總是為 enum 定義一個有意義的 0 值
2. **使用 Flags 時採用 2 的次方**：1, 2, 4, 8, 16, 32...
3. **加上 XML 註解**：為每個 enum 值提供清楚的說明
4. **使用 byte 節省記憶體**：當值範圍允許時，使用 byte 而非 int

### ❌ 避免的陷阱

1. **不要混合 Flags 和非 Flags**：一個 enum 要嘛是 Flags，要嘛不是
2. **避免使用 GetName() 處理 Flags 組合**：會回傳 null
3. **注意 JSON 序列化格式**：確保前後端格式一致
4. **避免在 Flags 中使用組合值作為基礎值**：會造成 ToString() 混淆

---

## 🔗 相關資源

- [Microsoft Docs - Enum](https://docs.microsoft.com/en-us/dotnet/api/system.enum)
- [C# Flags Attribute](https://docs.microsoft.com/en-us/dotnet/api/system.flagsattribute)
- [System.Text.Json](https://docs.microsoft.com/en-us/dotnet/api/system.text.json)

---

*📝 本文件最後更新：2025年7月12日*
*🏷️ 標籤：C#, Enum, Flags, JSON, 最佳實踐*
