# 🔍 Reflection

## 📖 目錄
- [一、typeof 與 GetType()](#一typeof-與-gettype)
  - [1.1 typeof 是什麼？](#11-typeof-是什麼)
  - [1.2 什麼是 Type 物件？](#12-什麼是-type-物件)
  - [1.3 簡單的範例](#13-簡單的範例)
  - [1.4 和 GetType() 的差別](#14-和-gettype-的差別)
  - [1.5 為什麼要用 typeof](#15-為什麼要用-typeof)
  - [1.6 應用範例](#16-應用範例)
    - [1.6.1 搭配 is 判斷型別](#161-搭配-is-判斷型別)
    - [1.6.2 搭配泛型寫通用程式](#162-搭配泛型寫通用程式)
    - [1.6.3 取得類別的所有屬性（反射）](#163-取得類別的所有屬性反射)
    - [1.6.4 用於 attribute](#164-用於-attribute)
    - [1.6.5 取得物件 constants](#165-取得物件-constants)

---

## 一、typeof 與 GetType()

### 1.1 typeof 是什麼？

`typeof` 是 C# 中的一個運算子，作用是**在編譯期取得某個型別對應的 Type 物件**。

##### 📝 基本概念

- **輸入**：型別名稱（像 `int`、`string`、類別名）
- **輸出**：`Type` 物件
- **時機**：編譯期就確定

```csharp
// 基本語法
Type intType = typeof(int);
Type stringType = typeof(string);
Type myClassType = typeof(MyClass);
```

### 1.2 什麼是 Type 物件？

`Type` 是 .NET 裡的類別，代表「型別的描述資訊」。這個物件可以告訴你這個型別有哪些方法、屬性、欄位等等。就像拿到一張「型別的說明書」。

##### 🎯 Type 物件能提供的資訊

- 📋 **方法清單**：該型別有哪些方法
- 🏷️ **屬性清單**：該型別有哪些屬性
- 📦 **欄位清單**：該型別有哪些欄位
- 🔧 **建構函式**：如何建立該型別的實例
- 🧬 **繼承關係**：父類別、介面實作等

### 1.3 簡單的範例

```csharp
void Main()
{
    Console.WriteLine(typeof(int));         
    Console.WriteLine(typeof(string));      
    Console.WriteLine(typeof(List<string>));
}
```

##### 📊 輸出結果

```
System.Int32
System.String
System.Collections.Generic.List`1[System.String]
```

> **💡 注意**：輸出的是完整的 .NET 型別名稱，不是 C# 的別名。

### 1.4 和 GetType() 的差別

| 特性 | `typeof` | `GetType()` |
|------|----------|-------------|
| **執行時機** | 編譯期就知道型別 | 執行期取得物件的型別 |
| **需要物件** | 不需要物件 | 必須先有物件 |
| **使用方式** | `typeof(型別名稱)` | `物件.GetType()` |

##### 🔍 實際範例比較

```csharp
string name = "Allen";

Console.WriteLine(typeof(string));   // 用型別名稱，編譯期確定
Console.WriteLine(name.GetType());   // 用物件來問，執行期確定
```

**兩者輸出結果相同：**
```
System.String
System.String
```

##### ⚡ 使用場景差異

**`typeof` 適用於：**
- ✅ 已知型別名稱
- ✅ 泛型程式設計
- ✅ 屬性標記
- ✅ 型別比較

**`GetType()` 適用於：**
- ✅ 執行期型別檢查
- ✅ 多型物件的實際型別
- ✅ 動態型別判斷

### 1.5 為什麼要用 typeof

因為有時候程式需要**在執行前就鎖定一個型別的描述**，例如：

- 🔍 **用於反射**：`typeof(MyClass).GetMethods()`
- 🏷️ **指定屬性 (attribute) 的型別參數**
- ⚖️ **做型別比對**
- 🧩 **泛型約束和型別參數**

### 1.6 應用範例

#### 1.6.1 搭配 is 判斷型別

```csharp
void Main()
{
    object x = 123;

    if (x.GetType() == typeof(int))
    {
        Console.WriteLine("這是 int");
    }
}
```

##### 💡 更好的寫法

```csharp
void Main()
{
    object x = 123;

    // 使用 is 關鍵字更簡潔
    if (x is int)
    {
        Console.WriteLine("這是 int");
    }
    
    // 或使用模式匹配
    if (x is int intValue)
    {
        Console.WriteLine($"這是 int，值為 {intValue}");
    }
}
```

#### 1.6.2 搭配泛型寫通用程式

```csharp
void Main()
{
    PrintType<string>();   // 輸出 System.String
    PrintType<DateTime>(); // 輸出 System.DateTime
}

void PrintType<T>()
{
    Console.WriteLine(typeof(T));
    Console.WriteLine($"型別名稱: {typeof(T).Name}");
    Console.WriteLine($"命名空間: {typeof(T).Namespace}");
    Console.WriteLine($"是否為數值型別: {typeof(T).IsValueType}");
    Console.WriteLine("---");
}
```

##### 📊 輸出結果

```
System.String
型別名稱: String
命名空間: System
是否為數值型別: False
---
System.DateTime
型別名稱: DateTime
命名空間: System
是否為數值型別: True
---
```

#### 1.6.3 取得類別的所有屬性（反射）

```csharp
void Main()
{
    var type = typeof(DateTime);
    var props = type.GetProperties();
    
    Console.WriteLine($"{type.Name} 的所有屬性：");
    foreach (var p in props)
    {
        Console.WriteLine($"- {p.Name} ({p.PropertyType.Name})");
    }
}
```

這段會列出 `DateTime` 的所有屬性（Year、Month、Day...）。

##### 📊 部分輸出結果

```
DateTime 的所有屬性：
- Date (DateTime)
- Day (Int32)
- DayOfWeek (DayOfWeek)
- DayOfYear (Int32)
- Hour (Int32)
- Kind (DateTimeKind)
- Millisecond (Int32)
- Month (Int32)
- Year (Int32)
...
```

#### 1.6.4 用於 attribute

```csharp
void Main()
{
    // 在依賴注入中使用
    var service = provider.GetRequiredService(typeof(MyService));
    
    // 在屬性標記中使用
    var attributes = typeof(MyClass).GetCustomAttributes(typeof(MyAttribute), false);
}

[AttributeUsage(AttributeTargets.Class)]
public class MyAttribute : Attribute
{
    public string Description { get; set; }
}

[My(Description = "這是我的類別")]
public class MyClass
{
    // 類別內容
}
```

#### 1.6.5 取得物件 constants

以下範例展示如何使用反射取得類別中的所有常數值：

```csharp
void Main()
{
    typeof(StripeAccountTypeConstants).Dump();
    
    var validTypes = typeof(StripeAccountTypeConstants)
        .GetFields(BindingFlags.Public | BindingFlags.Static)
        .Where(fi => fi.IsLiteral && !fi.IsInitOnly && fi.FieldType == typeof(string))
        .Select(fi => (string)fi.GetValue(null));
        
    foreach(var V in validTypes)
    {
        V.ToString().Dump();
    }
}

public static class StripeAccountTypeConstants
{
    /// <summary>
    /// Custom
    /// </summary>
    public const string Custom = nameof(Custom);

    /// <summary>
    /// Custom Test
    /// </summary>
    public const string CustomTest = nameof(CustomTest);

    /// <summary>
    /// Custom UAT
    /// </summary>
    public const string CustomUAT = nameof(CustomUAT);

    /// <summary>
    /// Custom UAT Test
    /// </summary>
    public const string CustomUATTest = nameof(CustomUATTest);

    /// <summary>
    /// Standard
    /// </summary>
    public const string Standard = nameof(Standard);

    /// <summary>
    /// Standard UAT
    /// </summary>
    public const string StandardUAT = nameof(StandardUAT);

    /// <summary>
    /// Test for Custom
    /// </summary>
    public const string Test = nameof(Test);
}
```

##### 🔍 程式碼解析

**BindingFlags 說明：**
- `Public`：只取得公開的成員
- `Static`：只取得靜態成員

**篩選條件說明：**
- `IsLiteral`：是常數（const）
- `!IsInitOnly`：不是唯讀欄位（readonly）
- `FieldType == typeof(string)`：型別是字串

**GetValue(null) 說明：**
- 因為是靜態常數，所以傳入 `null` 作為實例參數

##### 📊 預期輸出結果

```
Custom
CustomTest
CustomUAT
CustomUATTest
Standard
StandardUAT
Test
```

##### 💡 實際應用場景

這種技巧常用於：
- 🔧 **設定檔驗證**：檢查設定值是否為有效的常數
- 📋 **下拉選單建立**：動態產生選項清單
- ✅ **資料驗證**：確保輸入值在允許的範圍內
- 🎯 **單元測試**：測試所有定義的常數

**實用的擴展方法：**
```csharp
public static class ConstantsHelper
{
    public static IEnumerable<string> GetStringConstants<T>()
    {
        return typeof(T)
            .GetFields(BindingFlags.Public | BindingFlags.Static)
            .Where(fi => fi.IsLiteral && !fi.IsInitOnly && fi.FieldType == typeof(string))
            .Select(fi => (string)fi.GetValue(null));
    }
    
    public static bool IsValidConstant<T>(string value)
    {
        return GetStringConstants<T>().Contains(value);
    }
}

// 使用方式
var allTypes = ConstantsHelper.GetStringConstants<StripeAccountTypeConstants>();
bool isValid = ConstantsHelper.IsValidConstant<StripeAccountTypeConstants>("Custom");
```

> **🌟 重點提醒**
> 
> 1. **編譯期 vs 執行期**：`typeof` 在編譯期確定，`GetType()` 在執行期確定
> 2. **效能考量**：`typeof` 比 `GetType()` 效能更好，因為是編譯期操作
> 3. **null 安全**：`typeof` 不會有 null 問題，`GetType()` 需要注意 null 檢查
> 4. **反射效能**：反射操作相對較慢，避免在高頻呼叫的程式碼中使用