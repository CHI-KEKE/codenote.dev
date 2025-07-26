# C# 程式碼品質提升指南

## 目錄
- [一、程式碼重構與共用能力](#一程式碼重構與共用能力)
  - [1.1 抽取共用驗證邏輯](#11-抽取共用驗證邏輯)
  - [1.2 關於靜態與實例方法](#12-關於靜態與實例方法)
    - [1.2.1 舉例 Substring() vs char.ToUpper()](#121-舉例-substring-vs-chartoupper)
---

## 一、程式碼重構與共用能力

### 1.1 抽取共用驗證邏輯9

在促銷引擎的開發中，常常需要對「指定商品類型」和「排除商品類型」進行相同的驗證邏輯。如果每次都重複寫相同的驗證程式碼，會造成：

透過建立共用的驗證函式 `ValidateSalePageTypeList`，我們可以優雅地解決上述問題：

```csharp
/// <summary>
/// 指定排除類型 / 指定排除商品 對應驗證
/// </summary>
/// <param name="entity">The entity.</param>
/// <returns></returns>
private ValidationFailure ValidateTargetTypeProductList(PromotionEngineBaseEntity entity)
{
    return ValidateSalePageTypeList(
        entity.TargetTypeDef, 
        entity.IncludeTargetIdList, 
        entity.IncludeProductSkuOuterIdList, 
        nameof(PromotionEngineBaseEntity.TargetTypeDef), 
        "指定商品類型, 請指定商品或料號")
        ?? ValidateSalePageTypeList(
        entity.ExcludeTargetTypeDef, 
        entity.ExcludeTargetIdList, 
        entity.ExcludeProductSkuOuterIdList, 
        nameof(PromotionEngineBaseEntity.ExcludeTargetTypeDef), 
        "排除商品類型, 請排除商品或料號");
}

/// <summary>
/// Validates the type list.
/// </summary>
/// <param name="typeDef">The type definition.</param>
/// <param name="targetIdList">The target identifier list.</param>
/// <param name="productSkuOuterIdList">The product sku outer identifier list.</param>
/// <param name="propertyName">Name of the property.</param>
/// <param name="errorMessage">The error message.</param>
/// <returns> ValidationFailure </returns>
private ValidationFailure ValidateSalePageTypeList(
    string typeDef, 
    List<long> targetIdList, 
    List<string> productSkuOuterIdList, 
    string propertyName, 
    string errorMessage)
{
    var noTargetIdSelected = targetIdList == null || targetIdList.Any() == false;
    var noProductSkuOuterIdSelected = productSkuOuterIdList == null || productSkuOuterIdList.Any() == false;
    
    if (typeDef == nameof(PromotionEngineTargetTypeDefEnum.SalePage) && 
        noTargetIdSelected && 
        noProductSkuOuterIdSelected)
    {
        return new ValidationFailure(propertyName, errorMessage);
    }

    return null;
}
```

### 1.2 關於靜態與實例方法

在 C# 程式開發中，理解靜態方法與實例方法的差異是非常重要的。這兩種方法的呼叫方式和設計理念完全不同，讓我們透過具體範例來深入理解。

#### 1.2.1 舉例 Substring() vs char.ToUpper()

透過比較 `Substring()` 和 `char.ToUpper()` 這兩個常用方法，我們可以清楚看出靜態與實例方法的本質差異。

##### 🔍 Substring() - 實例方法範例

`Substring()` 定義在 `System.String` class 裡面（實際型別是 `sealed class String`）

**原始方法定義（.NET BCL）：**
```csharp
namespace System
{
    public sealed class String : IComparable, ICloneable, IEnumerable<char>
    {
        public string Substring(int startIndex);
        public string Substring(int startIndex, int length);
        // 其他方法 ...
    }
}
```

**呼叫方式：**
```csharp
string name = "hello";
var part = name.Substring(1);  // 這裡是「name 這個物件」自己執行 Substring
Console.WriteLine(part);       // 輸出：ello
```

> **💡 重點**：`Substring()` 是「字串物件自己的方法」，呼叫形式是 `物件.方法()`。

##### ⚡ char.ToUpper() - 靜態方法範例

`char.ToUpper()` 定義在 `System.Char` struct 裡面（型別是 `public readonly struct Char`）

**原始方法定義（.NET BCL）：**
```csharp
namespace System
{
    public readonly struct Char : IComparable, IEquatable<char>
    {
        public static char ToUpper(char c);
        public static char ToUpper(char c, CultureInfo culture);
        // 其他方法 ...
    }
}
```

**呼叫方式：**
```csharp
char c = 'a';
char upper = char.ToUpper(c); // 這裡是用「類別名」呼叫靜態方法
Console.WriteLine(upper);     // 輸出：A
```

> **💡 重點**：`ToUpper()` 是一個靜態方法 (static)，呼叫形式是 `類別.方法()`，而不是靠物件。

##### 📊 兩者差異重點整理

| 方法名稱 | 定義類別 | 方法型態 | 呼叫方式 | 記憶體配置 |
|----------|----------|----------|----------|------------|
| `Substring()` | `System.String` | 實例方法 | `myString.Substring()` | 需要物件實例 |
| `ToUpper()` | `System.Char` | 靜態方法 | `char.ToUpper(myChar)` | 不需要物件實例 |

##### 🤔 為什麼呼叫方式看起來不一樣？

**實例方法：**
- 必須先有一個「物件」(string)，方法才能用
- 方法會用到這個物件內的資料
- 呼叫時透過 `物件.方法()` 的形式

```csharp
string text = "Hello World";
// text 物件包含了字元陣列資料
string result = text.Substring(6); // 方法需要存取 text 物件的內部資料
```

**靜態方法：**
- 不需要物件，方法跟類別本身綁在一起
- 只要給參數即可，不依賴任何物件的狀態
- 呼叫時透過 `類別.方法()` 的形式

```csharp
char letter = 'a';
// ToUpper 不需要依賴任何物件的狀態，純粹是功能性的轉換
char upperLetter = char.ToUpper(letter); // 只是一個純函式轉換
```