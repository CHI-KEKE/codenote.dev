# 🔢 C# 數字處理完整指南

> 歡迎來到 C# 數字處理的奇妙世界！從基礎的解析到進階的數學運算，讓我們一起探索數字的無限可能！

## 📋 目錄

1. [數字解析與轉換](#數字解析與轉換)
2. [前導零處理](#前導零處理)
3. [四捨五入運算](#四捨五入運算)
4. [數字格式化](#數字格式化)
5. [數學運算函式](#數學運算函式)
6. [數字範圍與邊界](#數字範圍與邊界)
7. [效能考量](#效能考量)
8. [最佳實踐](#最佳實踐)

---

## 🔄 數字解析與轉換

### 基本轉換方法

```csharp
// 字串轉整數的各種方法
string numberStr = "123";

// 方法 1: Parse (會拋出例外)
int number1 = int.Parse(numberStr);

// 方法 2: TryParse (安全轉換)
if (int.TryParse(numberStr, out int number2))
{
    Console.WriteLine($"轉換成功: {number2}");
}

// 方法 3: Convert 類別
int number3 = Convert.ToInt32(numberStr);
```

### 浮點數轉換

```csharp
string floatStr = "123.45";

// 轉換為 double
double doubleValue = double.Parse(floatStr);

// 轉換為 decimal (推薦用於金融計算)
decimal decimalValue = decimal.Parse(floatStr);

// 轉換為 float
float floatValue = float.Parse(floatStr);
```

---

## 🎯 前導零處理

### ✨ 前導零的神奇特性

您知道嗎？C# 的數字解析器可以輕鬆處理前導零！

```csharp
// 🎉 前導零會被自動忽略
if (int.Parse("02").ToString() == "2")
{
    "耶！前導零處理成功！".Dump(); // ✅ 這會執行
}

// 更多前導零範例
Console.WriteLine(int.Parse("000123"));    // 輸出: 123
Console.WriteLine(int.Parse("007"));       // 輸出: 7
Console.WriteLine(int.Parse("0"));         // 輸出: 0
```

### 前導零的實際應用

```csharp
// 處理檔案編號
string[] fileNumbers = { "001", "002", "010", "100" };
var sortedNumbers = fileNumbers
    .Select(x => int.Parse(x))
    .OrderBy(x => x)
    .ToList();

foreach (var num in sortedNumbers)
{
    Console.WriteLine($"檔案編號: {num:D3}"); // 格式化為3位數
}
```

### ⚠️ 注意事項

```csharp
// 小心八進位陷阱（在某些語言中）
// 在 C# 中，前導零不會被視為八進位
int number = int.Parse("08"); // ✅ 結果是 8，不是錯誤

// 但在程式碼中直接寫數字時要小心
// int octal = 08; // ❌ 編譯錯誤，8 不是有效的八進位數字
```

---

## 🎲 四捨五入運算

### Math.Round 的威力

```csharp
double value = 2.5;

// 不同的四捨五入策略
double rounded1 = Math.Round(value);                           // 2 (預設為 ToEven)
double rounded2 = Math.Round(value, MidpointRounding.AwayFromZero); // 3
double rounded3 = Math.Round(value, MidpointRounding.ToEven);        // 2
```

### 🕐 時間差的四捨五入實例

```csharp
DateTime startTime = DateTime.Now.AddDays(-5.7);
DateTime currentTime = DateTime.Now;

// 計算天數差並四捨五入
double daysDifference = Math.Round(
    currentTime.Subtract(startTime).TotalDays, 
    MidpointRounding.AwayFromZero
);

Console.WriteLine($"經過了 {daysDifference} 天");
```

### 四捨五入策略比較

| 策略 | 2.5 的結果 | 3.5 的結果 | 說明 |
|------|------------|------------|------|
| `ToEven` (預設) | 2 | 4 | 銀行家四捨五入 |
| `AwayFromZero` | 3 | 4 | 遠離零值 |
| `ToZero` | 2 | 3 | 趨向零值 |
| `ToNegativeInfinity` | 2 | 3 | 向負無窮大 |
| `ToPositiveInfinity` | 3 | 4 | 向正無窮大 |

```csharp
// 實際範例
double[] testValues = { 2.5, 3.5, -2.5, -3.5 };

foreach (var value in testValues)
{
    Console.WriteLine($"原值: {value}");
    Console.WriteLine($"  ToEven: {Math.Round(value, MidpointRounding.ToEven)}");
    Console.WriteLine($"  AwayFromZero: {Math.Round(value, MidpointRounding.AwayFromZero)}");
    Console.WriteLine();
}
```

---

## 🎨 數字格式化

### 基本格式化

```csharp
int number = 1234567;
double price = 1234.56;

// 千分位分隔符
Console.WriteLine(number.ToString("N0"));    // 1,234,567
Console.WriteLine(price.ToString("N2"));     // 1,234.56

// 貨幣格式
Console.WriteLine(price.ToString("C"));      // $1,234.56

// 百分比格式
Console.WriteLine(0.1234.ToString("P2"));    // 12.34%
```

### 前導零格式化

```csharp
int id = 42;

// 建立指定長度的前導零
Console.WriteLine(id.ToString("D5"));        // 00042
Console.WriteLine(id.ToString("00000"));     // 00042
Console.WriteLine($"{id:D3}");              // 042
```

### 自訂格式

```csharp
double value = 1234.5678;

// 自訂小數位數
Console.WriteLine(value.ToString("F2"));     // 1234.57
Console.WriteLine(value.ToString("0.00"));   // 1234.57

// 科學記號
Console.WriteLine(value.ToString("E2"));     // 1.23E+003
```

---

## 🧮 數學運算函式

### 常用數學函式

```csharp
double x = 16.7;

// 基本運算
Console.WriteLine($"絕對值: {Math.Abs(-x)}");           // 16.7
Console.WriteLine($"平方根: {Math.Sqrt(x)}");           // 4.087...
Console.WriteLine($"冪運算: {Math.Pow(x, 2)}");         // 278.89
Console.WriteLine($"對數: {Math.Log(x)}");              // 2.814...

// 取整函式
Console.WriteLine($"無條件捨去: {Math.Floor(x)}");       // 16
Console.WriteLine($"無條件進位: {Math.Ceiling(x)}");     // 17
Console.WriteLine($"截斷小數: {Math.Truncate(x)}");      // 16
```

### 最大值與最小值

```csharp
int[] numbers = { 5, 2, 8, 1, 9, 3 };

// 使用 LINQ
int max = numbers.Max();
int min = numbers.Min();

// 使用 Math 類別（兩個值比較）
int result = Math.Max(10, 20);  // 20
int result2 = Math.Min(10, 20); // 10
```

---

## 📊 數字範圍與邊界

### 整數型別範圍

```csharp
// 查看各種整數型別的範圍
Console.WriteLine($"byte: {byte.MinValue} ~ {byte.MaxValue}");
Console.WriteLine($"short: {short.MinValue} ~ {short.MaxValue}");
Console.WriteLine($"int: {int.MinValue} ~ {int.MaxValue}");
Console.WriteLine($"long: {long.MinValue} ~ {long.MaxValue}");
```

### 浮點數特殊值

```csharp
// 特殊的浮點數值
double infinity = double.PositiveInfinity;
double negInfinity = double.NegativeInfinity;
double notANumber = double.NaN;

// 檢查特殊值
Console.WriteLine($"是無窮大: {double.IsInfinity(infinity)}");
Console.WriteLine($"是 NaN: {double.IsNaN(notANumber)}");
Console.WriteLine($"是正常數字: {double.IsNormal(123.45)}");
```

### 溢位處理

```csharp
// checked 關鍵字檢查溢位
try
{
    checked
    {
        int maxInt = int.MaxValue;
        int overflow = maxInt + 1; // 會拋出 OverflowException
    }
}
catch (OverflowException ex)
{
    Console.WriteLine("發生溢位！");
}

// unchecked 允許溢位
unchecked
{
    int maxInt = int.MaxValue;
    int wrapped = maxInt + 1; // 會變成 int.MinValue
    Console.WriteLine($"溢位後的值: {wrapped}");
}
```

---

## ⚡ 效能考量

### 數字類型選擇

```csharp
// 效能比較：整數運算最快
int intCalc = 1000000;
long longCalc = 1000000L;
float floatCalc = 1000000f;
double doubleCalc = 1000000.0;
decimal decimalCalc = 1000000m;

// decimal 最精確但最慢，適合金融計算
// double 是預設浮點類型，效能良好
// float 節省記憶體但精度較低
// int/long 整數運算最快
```

### TryParse vs Parse

```csharp
string input = "123";

// TryParse 效能較好（不拋出例外）
if (int.TryParse(input, out int result))
{
    // 處理成功的情況
}

// Parse 需要處理例外（效能較差）
try
{
    int result2 = int.Parse(input);
}
catch (FormatException)
{
    // 處理錯誤
}
```

---

## 🎯 最佳實踐

### 1. 選擇適當的數字類型

```csharp
// ✅ 好的做法
decimal bankBalance = 1000.50m;    // 金融計算用 decimal
int userAge = 25;                  // 整數用 int
double scientificValue = 1.23e-4;  // 科學計算用 double

// ❌ 避免的做法
float bankBalance2 = 1000.50f;     // 金融計算不要用 float
```

### 2. 安全的數字轉換

```csharp
// ✅ 推薦的轉換方式
public static bool TryParseToInt(string input, out int result)
{
    return int.TryParse(input?.Trim(), out result);
}

// 使用範例
string userInput = " 123 ";
if (TryParseToInt(userInput, out int number))
{
    Console.WriteLine($"成功轉換: {number}");
}
```

### 3. 四捨五入的一致性

```csharp
// ✅ 在整個應用程式中保持一致的四捨五入策略
public static class MathHelper
{
    public static double RoundCurrency(double value)
    {
        return Math.Round(value, 2, MidpointRounding.AwayFromZero);
    }
    
    public static double RoundPercentage(double value)
    {
        return Math.Round(value, 4, MidpointRounding.AwayFromZero);
    }
}
```

### 4. 格式化的可讀性

```csharp
// ✅ 使用描述性的格式化
public static class NumberFormatter
{
    public static string FormatCurrency(decimal amount)
    {
        return amount.ToString("C2");
    }
    
    public static string FormatFileSize(long bytes)
    {
        string[] sizes = { "B", "KB", "MB", "GB", "TB" };
        double len = bytes;
        int order = 0;
        
        while (len >= 1024 && order < sizes.Length - 1)
        {
            order++;
            len = len / 1024;
        }
        
        return $"{len:0.##} {sizes[order]}";
    }
}
```

---

## 🎪 有趣的數字技巧

### 數字魔法

```csharp
// 🎭 交換兩個數字而不使用第三個變數
int a = 5, b = 10;
Console.WriteLine($"交換前: a={a}, b={b}");

a = a + b;  // a = 15
b = a - b;  // b = 5
a = a - b;  // a = 10

Console.WriteLine($"交換後: a={a}, b={b}");

// 🎯 快速判斷奇偶數
int number = 42;
bool isEven = (number & 1) == 0;
Console.WriteLine($"{number} 是偶數: {isEven}");

// 🚀 快速計算 2 的冪次
int powerOfTwo = 1 << 5; // 2^5 = 32
Console.WriteLine($"2的5次方: {powerOfTwo}");
```

### 數字驗證

```csharp
// 📱 驗證手機號碼格式
public static bool IsValidPhoneNumber(string phone)
{
    return phone.All(char.IsDigit) && phone.Length == 10;
}

// 🆔 驗證身分證號碼檢查碼（台灣）
public static bool IsValidTaiwanId(string id)
{
    if (id.Length != 10) return false;
    
    // 簡化版驗證邏輯
    return char.IsLetter(id[0]) && id.Skip(1).All(char.IsDigit);
}
```

---