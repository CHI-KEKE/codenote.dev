# 🎯 C# 字串處理完整指南

## 📖 目錄
- [🔍 字串空值檢查](#字串空值檢查)
- [✂️ 字串擷取技巧](#字串擷取技巧)
- [🏷️ nameof 的魔法](#nameof-的魔法)
- [🔪 字串分割實戰](#字串分割實戰)
- [🧬 字串的不可變性](#字串的不可變性)
- [🔗 字串串接大法](#字串串接大法)
- [🧹 字串清理技巧](#字串清理技巧)
- [🎭 字元與編碼深度解析](#字元與編碼深度解析)
- [🎪 字串擴充方法](#字串擴充方法)

---

## 🔍 字串空值檢查

### ☘️ `IsNullOrWhiteSpace` vs `IsNullOrEmpty`

```csharp
void Main()
{
    var settings = "   ";  // 只有空格
    if (string.IsNullOrWhiteSpace(settings))
    {
        "這會執行！".Dump();  // 結果：true
    }
}
```

**🎯 關鍵差異：**

| 方法 | `null` | `""` | `"   "` (空格) |
|------|--------|------|----------------|
| `IsNullOrEmpty` | ✅ | ✅ | ❌ |
| `IsNullOrWhiteSpace` | ✅ | ✅ | ✅ |

**💡 使用時機：**
- `IsNullOrEmpty`：只檢查 null 或空字串
- `IsNullOrWhiteSpace`：還要檢查只有空白字元的情況（推薦！）

---

## ✂️ 字串擷取技巧

### ☘️ 取得信用卡後 4 碼

```csharp
// 🎯 Substring 的本質：「告訴程式：我要從哪裡開始，拿多少個字」
string cardNumber = "1234567890123456";
string last4Digits = cardNumber.Substring(cardNumber.Length - 4);
Console.WriteLine(last4Digits); // "3456"

// 實際應用範例
if (dbInfo.No.Substring(dbInfo.No.Length - 4) == stripeAPIresult.CreditCardInfo.Last4)
{
    // 信用卡後四碼比對成功
}
```

**🔑 Substring 使用技巧：**
```csharp
string text = "Hello World";
text.Substring(6);        // "World" - 從位置6到結尾
text.Substring(0, 5);     // "Hello" - 從位置0開始取5個字元
text.Substring(text.Length - 5); // "World" - 取最後5個字元
```

---

## 🏷️ nameof 的魔法

### 🎯 編譯時期的字串魔法

```csharp
public class Person
{
    public string Name { get; set; }
    
    public void ValidateName()
    {
        if (string.IsNullOrEmpty(Name))
        {
            throw new ArgumentException($"{nameof(Name)} 不能為空");
            // 編譯後變成："Name 不能為空"
        }
    }
}
```

**✨ nameof 的特色：**
- 🏗️ **編譯時執行**：在程式編譯階段就轉換成字串
- 🔄 **重構安全**：當屬性名稱改變時，nameof 會自動更新
- 🎯 **無執行成本**：執行時期完全不做任何事情

**📝 實用場景：**
```csharp
// 1. 參數驗證
public void SetValue(string value)
{
    if (value == null)
        throw new ArgumentNullException(nameof(value));
}

// 2. 屬性變更通知
public event PropertyChangedEventHandler PropertyChanged;
protected void OnPropertyChanged([CallerMemberName] string propertyName = "")
{
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
```

---

## 🔪 字串分割實戰

### Split 方法的實際應用

```csharp
/// <summary>
/// 取得自訂規則 2
/// </summary>
/// <param name="customField2">格式：key:value</param>
/// <param name="defaultValue">預設值</param>
/// <returns>轉換後的值</returns>
public override T GetCustomField2ToEntity<T>(string customField2, T defaultValue)
{
    var splitCustomField = customField2?.Split(":");
    
    // 防禦性程式設計
    if (string.IsNullOrEmpty(customField2) || 
        splitCustomField == null || 
        splitCustomField.Length < 2)
    {
        return defaultValue;
    }

    return (T)Convert.ChangeType(splitCustomField[1], typeof(T));
}
```

**🛡️ Split 最佳實務：**
```csharp
// ✅ 好的做法
string data = "apple,banana,orange";
string[] fruits = data.Split(',');
if (fruits.Length >= 2)
{
    string secondFruit = fruits[1]; // 安全存取
}

// ❌ 危險的做法
string secondFruit = data.Split(',')[1]; // 可能越界錯誤
```

---

## 🧬 字串的不可變性

### 為什麼 string 是不可變的？

```csharp
string original = "Hello";
string modified = original + " World";
Console.WriteLine(original);  // 還是 "Hello"
Console.WriteLine(modified);  // "Hello World"
```

**🔒 不可變性的 5 大優點：**

#### 1️⃣ 🛡️ 安全性（Thread-Safety）
```csharp
// 多執行緒環境下安全
string shared = "共享字串";
Task.Run(() => Console.WriteLine(shared)); // 安全
Task.Run(() => Console.WriteLine(shared)); // 安全
```

#### 2️⃣ 🔑 Hash 快取與 Dictionary 最佳化
```csharp
var dict = new Dictionary<string, int>();
string key = "myKey";
dict[key] = 100;
// key 內容永遠不變，HashCode 穩定可靠
```

#### 3️⃣ 💾 String Interning 記憶體最佳化
```csharp
string a = "hello";
string b = "hello";
Console.WriteLine(Object.ReferenceEquals(a, b)); // True - 共用記憶體！
```

#### 4️⃣ 🎯 設計簡化與語言一致性
- 傳參數時不用擔心被修改
- 物件比較邏輯簡單明確
- 程式行為更可預測

#### 5️⃣ 🗑️ 垃圾回收最佳化
- 短生命週期物件回收效率高
- GC 對小物件有特殊最佳化

### 🔍 Reference Type 但表現像 Value Type

```csharp
string foo = "hello";  // foo 是指向 heap 中字串的參考

void ModifyString(string input)
{
    input = "modified";  // 只是改變區域變數的參考
}

ModifyString(foo);
Console.WriteLine(foo); // 還是 "hello"
```

---

## 🔗 字串串接大法

### 1️⃣ 使用 + 運算子（簡單場景）

```csharp
string foo = "Morning!";
string bar = "Nice day for fishing, ain't it?";
string result = foo + " " + bar;
Console.WriteLine(result); // "Morning! Nice day for fishing, ain't it?"
```

### 2️⃣ 使用 String.Format()（格式化）

```csharp
string result = String.Format("{0} {1}", foo, bar);
// ⚠️ 注意：插入點 {} 數量要與參數匹配
```

### 3️⃣ 使用 String.Concat()（多個字串）

```csharp
string result = String.Concat(foo, " ", bar);
// 可以串接多個字串、陣列或 IEnumerable<string>
```

### 4️⃣ 使用字串插值（現代推薦）

```csharp
string result = $"{foo} {bar}";
// 🎯 可讀性高，現代 C# 首選方式
```

### 5️⃣ 使用 String.Join()（陣列組合）

```csharp
string[] fruits = { "apple", "banana", "orange" };
string result = String.Join(", ", fruits); // "apple, banana, orange"

int[] numbers = { 1, 2, 3, 4, 5 };
string numberString = String.Join(",", numbers); // "1,2,3,4,5"
```

### 6️⃣ 使用 StringBuilder（大量串接）

```csharp
var sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
string result = sb.ToString(); // "Hello World"
```

**⚡ StringBuilder 使用時機：**
- 迴圈中大量字串操作
- 動態組合複雜字串
- 需要高效記憶體使用

**🎯 效能比較：**
```csharp
// ❌ 效能差（大量串接時）
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString(); // 每次都建立新字串
}

// ✅ 效能好
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i.ToString()); // 在內部緩衝區修改
}
string result = sb.ToString();
```

---

## 🧹 字串清理技巧

### 移除換行符號

```csharp
string cleanTitle = shopAlbumEntity.Title
    .Replace("\r\n", "")  // Windows 換行
    .Replace("\n", "")    // Unix 換行
    .Replace("\r", "");   // Mac 換行
```

### 參數化字串建構

```csharp
public static string BuildStringByParams(string separator, params string[] strings)
{
    if (strings.Length == 0)
        return string.Empty;
    
    var builder = new StringBuilder();
    foreach (var subString in strings)
    {
        builder.Append(subString);
        builder.Append(separator);
    }
    
    // 移除最後一個分隔符
    var result = builder.ToString();
    var lastSeparatorIndex = result.LastIndexOf(separator);
    return result.Remove(lastSeparatorIndex);
}

// 使用範例
string result = BuildStringByParams(",", "aa", "final", "qjq");
// 結果："aa,final,qjq"
```

### StartsWith 實用範例

```csharp
private string GetStripeMode(string publishableKey)
{
    var liveModePubKeyPattern = "pk_live_";
    var testModePubKeyPattern = "pk_test_";
    
    // 使用 ?. 安全導航避免 null 例外
    if (publishableKey?.StartsWith(liveModePubKeyPattern) == true)
        return "Live";
    
    if (publishableKey?.StartsWith(testModePubKeyPattern) == true)
        return "Test";
    
    return "Default"; // 預設值
}
```

---

## 🎭 字元與編碼深度解析

### ✅ C# 中的 char 是什麼？

**🎯 核心概念：**
- `char` 不是一個「字元」，而是一個 **UTF-16 編碼單元**
- 每個 `char` 是 16 位元（2 bytes）
- 負責承載「部分」或「全部」的字元資料

### 🧠 Unicode 與 UTF-16 的關係

```csharp
// Unicode 代碼點範例
// 你 → U+4F60
// 𠜎 → U+2070E
// 😄 → U+1F604
```

**📊 編碼規則：**

| 字元類型 | Unicode 範圍 | 需要幾個 char | 範例 |
|----------|--------------|---------------|------|
| 基本字元 | U+0000 ~ U+FFFF (BMP) | 1 個 char | 中文、英文、數字 |
| 擴展字元 | 超過 U+FFFF | 2 個 char (代理對) | Emoji、罕見漢字 |

### 🔍 實際範例

```csharp
// ✅ 一般中文字（BMP）
char ch = '中';
Console.WriteLine((int)ch); // 20013 (U+4E2D)

// ❌ 錯誤：Emoji 不能直接用 char
// char emoji = '😄'; // 編譯錯誤！

// ✅ 正確：使用字串
string emoji = "😄";
Console.WriteLine(emoji.Length); // 2，因為是代理對
Console.WriteLine(char.IsSurrogate(emoji[0])); // true
Console.WriteLine(char.IsSurrogate(emoji[1])); // true
```

### 🧬 代理對（Surrogate Pair）檢測

```csharp
public static void AnalyzeString(string input)
{
    Console.WriteLine($"字串：{input}");
    Console.WriteLine($"Length：{input.Length}");
    
    for (int i = 0; i < input.Length; i++)
    {
        char ch = input[i];
        Console.WriteLine($"[{i}] {ch} (U+{(int)ch:X4}) " +
                         $"IsSurrogate: {char.IsSurrogate(ch)}");
    }
}

// 測試
AnalyzeString("A中😄");
// 輸出：
// 字串：A中😄
// Length：4
// [0] A (U+0041) IsSurrogate: False
// [1] 中 (U+4E2D) IsSurrogate: False  
// [2] 😄 (U+D83D) IsSurrogate: True   <- 代理對前半
// [3] 😄 (U+DE04) IsSurrogate: True   <- 代理對後半
```

### 🔄 比喻理解

**想像字元編碼像拼圖：**
- 🧩 **Unicode 字元（Code Point）** = 一個完整角色
- 🔷 **UTF-16 編碼單元（char）** = 拼圖塊

**拼圖規則：**
- 👤 一般角色（BMP）只需要 **1塊拼圖** 就完成
- 🤖 特殊角色（超過 BMP）需要 **2塊拼圖** 湊在一起

```csharp
// 1塊拼圖的角色
string simple = "你好"; // 每個字都是1個char
Console.WriteLine(simple.Length); // 2

// 2塊拼圖的角色  
string complex = "😄🎉"; // 每個Emoji都是2個char
Console.WriteLine(complex.Length); // 4
```

### 💡 實務建議

1. **字串長度計算時要小心：**
```csharp
// ❌ 錯誤的字元數計算
string text = "Hello😄";
int wrongCount = text.Length; // 7，不是6個字元

// ✅ 正確的字元數計算
int correctCount = text.EnumerateRunes().Count(); // 6個真正的字元
```

2. **字串處理時考慮代理對：**
```csharp
// ✅ 安全的字串遍歷
foreach (var rune in text.EnumerateRunes())
{
    Console.WriteLine($"字元：{rune}");
}
```

---

## � 字串擴充方法

### 9.1 FirstUpper - 首字母大寫擴充

有時候我們需要將字串的第一個字元轉為大寫，其餘保持不變。透過擴充方法，我們可以讓程式碼更加優雅：

```csharp
void Main()
{
    "fuck".ToFirstUpper().Dump();  // 輸出：Fuck
    "efew".ToUpper().Dump();       // 輸出：EFEW (對比用)
    
    // 更多測試案例
    "hello world".ToFirstUpper();  // Hello world
    "c#".ToFirstUpper();           // C#
    "".ToFirstUpper();             // (空字串)
    ((string)null).ToFirstUpper(); // null
}

/// <summary>
/// 字串擴充方法類別
/// </summary>
public static class StringExtention
{
    /// <summary>
    /// 將字串的第一個字元轉換為大寫，其餘字元保持不變
    /// </summary>
    /// <param name="value">要處理的字串</param>
    /// <returns>首字母大寫的字串</returns>
    public static string ToFirstUpper(this string value)
    {
        // 🛡️ 防禦性程式設計：處理 null 或空字串
        if (string.IsNullOrEmpty(value))
        {
            return value;
        }

        // 🎯 單一字元的特殊處理
        if (value.Length == 1)
        {
            return value.ToUpper();
        }

        // ✨ 核心邏輯：首字元大寫 + 其餘字元不變
        return char.ToUpper(value[0]) + value.Substring(1);
    }
}
```

**🎯 ToFirstUpper 的設計重點：**

#### 🛡️ 安全性考量
```csharp
// 處理各種邊界情況
Console.WriteLine(((string)null).ToFirstUpper());      // null，不會拋出例外
Console.WriteLine("".ToFirstUpper());                  // 空字串
Console.WriteLine("a".ToFirstUpper());                 // "A"，單字元處理
Console.WriteLine("ALREADY_UPPER".ToFirstUpper());     // "ALREADY_UPPER"，已是大寫
```