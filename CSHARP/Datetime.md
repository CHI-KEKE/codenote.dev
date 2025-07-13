# C# 時間處理完整指南 📅

## 目錄
- [基本時間操作](#基本時間操作)
- [時間比較](#時間比較)
- [字串解析](#字串解析)
- [時間種類判斷](#時間種類判斷)
- [時間格式化](#時間格式化)
- [文化特性設定](#文化特性設定)
- [精確格式解析](#精確格式解析)
- [時間計算](#時間計算)
- [時間戳處理](#時間戳處理)
- [時區處理](#時區處理)
- [高精度時間](#高精度時間)
- [最佳實務](#最佳實務)

---

## 基本時間操作

### 取得當前時間並進行日期操作

```csharp
// 取得當前時間
var now = DateTime.Now;
Console.WriteLine($"現在時間: {now}");

// 取得昨天的日期（只包含日期部分，時間為 00:00:00）
var startDate = now.Date.AddDays(-1);
Console.WriteLine($"昨天日期: {startDate}");

// 其他常用操作
var tomorrow = now.AddDays(1);
var nextWeek = now.AddDays(7);
var nextMonth = now.AddMonths(1);
var nextYear = now.AddYears(1);
```

> 💡 **提示**: `DateTime.Date` 會將時間部分設為 00:00:00

---

## 時間比較

### 基本比較操作

```csharp
// 比較當前時間與特定日期
DateTime targetDate = new DateTime(2020, 12, 12);
bool isAfter = DateTime.Now > targetDate;

Console.WriteLine($"當前時間是否晚於 2020-12-12: {isAfter}");

// 更多比較範例
DateTime date1 = new DateTime(2025, 3, 15);
DateTime date2 = new DateTime(2025, 3, 20);

Console.WriteLine($"date1 == date2: {date1 == date2}");
Console.WriteLine($"date1 < date2: {date1 < date2}");
Console.WriteLine($"date1.CompareTo(date2): {date1.CompareTo(date2)}"); // -1, 0, 1
```

---

## 字串解析

### 安全的字串轉換

```csharp
// 推薦使用 TryParse，避免例外狀況
string timeString = "2025-03-24T10:39:00";
if (DateTime.TryParse(timeString, out DateTime parsedTime))
{
    Console.WriteLine($"解析成功: {parsedTime}");
}
else
{
    Console.WriteLine("解析失敗");
}

// 指定格式解析
string customFormat = "2025/03/24 10:39";
if (DateTime.TryParseExact(customFormat, "yyyy/MM/dd HH:mm", null, DateTimeStyles.None, out DateTime exactTime))
{
    Console.WriteLine($"精確解析: {exactTime}");
}
```

> ⚠️ **注意**: 使用 `TryParse` 比 `Parse` 更安全，因為不會拋出例外

---

## 時間種類判斷

### 使用 DateTime.Kind 屬性

當時間物件在程式碼中經歷複雜的傳遞過程時，你可能需要確認它是 UTC 時間還是本地時間：

```csharp
// 建立不同種類的時間
var localTime = DateTime.Now;
var utcTime = DateTime.UtcNow;
var unspecifiedTime = new DateTime(2024, 5, 12, 21, 16, 1); // 未指定種類

Console.WriteLine($"本地時間: {localTime}");
Console.WriteLine($"UTC 時間: {utcTime}");
Console.WriteLine($"未指定時間: {unspecifiedTime}");

// 檢查時間種類
Console.WriteLine($"localTime.Kind: {localTime.Kind}");     // Local
Console.WriteLine($"utcTime.Kind: {utcTime.Kind}");         // Utc
Console.WriteLine($"unspecifiedTime.Kind: {unspecifiedTime.Kind}"); // Unspecified
```

### 實際應用情境

```csharp
public void ProcessOrderTime(DateTime orderTime)
{
    // 在服務層中，檢查接收到的時間類型
    switch (orderTime.Kind)
    {
        case DateTimeKind.Local:
            Console.WriteLine("這是本地時間，需要轉換為 UTC 進行儲存");
            var utcTime = orderTime.ToUniversalTime();
            break;
            
        case DateTimeKind.Utc:
            Console.WriteLine("這已經是 UTC 時間，可以直接使用");
            break;
            
        case DateTimeKind.Unspecified:
            Console.WriteLine("⚠️ 警告：時間種類未指定，建議明確指定");
            // 根據業務邏輯決定如何處理
            break;
    }
}
```

> 💡 **提示**: 使用 `DateTime.Kind` 可以避免時區混淆，特別是在處理來自不同來源的時間資料時

---

## 時間格式化

### 基本格式化方法

```csharp
DateTime now = DateTime.Now;

// 常用的格式化方式
string formatted1 = now.ToString("yyyy/MM/dd HH:mm:ss");
string formatted2 = now.ToString("yyyy-MM-dd");
string formatted3 = now.ToString("HH:mm:ss");

Console.WriteLine($"完整格式: {formatted1}");  // 2024/05/12 21:16:01
Console.WriteLine($"日期格式: {formatted2}");  // 2024-05-12
Console.WriteLine($"時間格式: {formatted3}");  // 21:16:01

// 實際應用範例
var orderUpdateTime = DateTime.Now;
var message = $"訂單更新時間: {orderUpdateTime.ToString("yyyy/MM/dd HH:mm:ss")}";
Console.WriteLine(message);
```

### 標準格式字串

```csharp
DateTime now = DateTime.Now;

Console.WriteLine("=== 標準格式字串 ===");
Console.WriteLine($"短日期 (d): {now:d}");           // 2024/5/12
Console.WriteLine($"長日期 (D): {now:D}");           // 2024年5月12日
Console.WriteLine($"完整日期時間 (F): {now:F}");     // 2024年5月12日 下午 09:16:01
Console.WriteLine($"一般日期時間 (G): {now:G}");     // 2024/5/12 下午 09:16:01
Console.WriteLine($"可排序格式 (s): {now:s}");       // 2024-05-12T21:16:01
Console.WriteLine($"ISO 8601 (o): {now:o}");        // 2024-05-12T21:16:01.1234567+08:00
```

---

## 文化特性設定

### 使用 CultureInfo 控制顯示格式

不同的文化設定會影響時間的顯示方式：

```csharp
DateTime date = DateTime.Now;

// 使用不同文化特性格式化
Console.WriteLine("=== 不同文化特性的時間顯示 ===");
Console.WriteLine($"系統預設: {date.ToString()}");
Console.WriteLine($"美國 (en-US): {date.ToString(new CultureInfo("en-US"))}");
Console.WriteLine($"台灣 (zh-TW): {date.ToString(new CultureInfo("zh-TW"))}");
Console.WriteLine($"香港 (zh-HK): {date.ToString(new CultureInfo("zh-HK"))}");
Console.WriteLine($"馬來西亞 (ms-MY): {date.ToString(new CultureInfo("ms-MY"))}");

// 輸出範例:
// 系統預設: 5/12/2024 11:06:46 PM
// 美國 (en-US): 5/12/2024 11:06:15 PM
// 台灣 (zh-TW): 2024/5/12 下午 11:06:15
// 香港 (zh-HK): 12/5/2024 下午 11:06:15
// 馬來西亞 (ms-MY): 12/05/2024 11:06:15 PTG
```

### 設定特定格式與文化

```csharp
DateTime date = DateTime.Now;
CultureInfo culture = new CultureInfo("zh-TW");

// 使用特定文化與格式
string customFormat = date.ToString("yyyy年MM月dd日 dddd", culture);
Console.WriteLine($"自訂中文格式: {customFormat}"); // 2024年05月12日 星期日

// 不變文化（建議用於資料儲存）
string invariantFormat = date.ToString("yyyy-MM-dd HH:mm:ss", CultureInfo.InvariantCulture);
Console.WriteLine($"不變文化格式: {invariantFormat}"); // 2024-05-12 21:16:01
```

> 🌍 **重要**: 在國際化應用中，使用 `CultureInfo.InvariantCulture` 進行資料儲存，避免因地區設定造成的問題

---

## 精確格式解析

### ParseExact vs Parse 的差異

```csharp
// 測試不同格式的字串
string dateString1 = "2024/05/12";
string dateString2 = "2024-05-12";

Console.WriteLine("=== ParseExact vs Parse 比較 ===");

// ParseExact：嚴格按照指定格式解析
try
{
    DateTime exactParsed1 = DateTime.ParseExact(dateString1, "yyyy/MM/dd", CultureInfo.InvariantCulture);
    Console.WriteLine($"ParseExact 成功: {exactParsed1}");
    
    // 這會拋出例外，因為格式不符
    DateTime exactParsed2 = DateTime.ParseExact(dateString2, "yyyy/MM/dd", CultureInfo.InvariantCulture);
    Console.WriteLine($"ParseExact 成功: {exactParsed2}");
}
catch (FormatException ex)
{
    Console.WriteLine($"❌ ParseExact 失敗: {ex.Message}");
}

// Parse：較寬鬆的解析，會嘗試多種格式
try
{
    DateTime parsed1 = DateTime.Parse(dateString1);
    DateTime parsed2 = DateTime.Parse(dateString2); // 這個會成功
    
    Console.WriteLine($"✅ Parse 成功 1: {parsed1}");
    Console.WriteLine($"✅ Parse 成功 2: {parsed2}");
}
catch (FormatException ex)
{
    Console.WriteLine($"Parse 失敗: {ex.Message}");
}
```

### 安全的精確解析

```csharp
public static class SafeDateParser
{
    /// <summary>
    /// 安全的精確日期解析
    /// </summary>
    public static DateTime? ParseExternalDate(string dateString, string format = "yyyy/MM/dd")
    {
        if (DateTime.TryParseExact(dateString, format, CultureInfo.InvariantCulture, 
            DateTimeStyles.None, out DateTime result))
        {
            return result;
        }
        return null;
    }
    
    /// <summary>
    /// 支援多種格式的解析
    /// </summary>
    public static DateTime? ParseMultipleFormats(string dateString)
    {
        string[] formats = {
            "yyyy/MM/dd",
            "yyyy-MM-dd",
            "dd/MM/yyyy",
            "MM/dd/yyyy",
            "yyyy.MM.dd"
        };
        
        foreach (string format in formats)
        {
            if (DateTime.TryParseExact(dateString, format, CultureInfo.InvariantCulture, 
                DateTimeStyles.None, out DateTime result))
            {
                return result;
            }
        }
        return null;
    }
}

// 使用範例
string[] testDates = { "2024/05/12", "2024-05-12", "12/05/2024" };

foreach (string dateStr in testDates)
{
    var parsed = SafeDateParser.ParseMultipleFormats(dateStr);
    if (parsed.HasValue)
    {
        Console.WriteLine($"✅ '{dateStr}' 解析成功: {parsed.Value:yyyy-MM-dd}");
    }
    else
    {
        Console.WriteLine($"❌ '{dateStr}' 解析失敗");
    }
}
```

### 最佳實務建議

| 方法 | 適用情境 | 優點 | 缺點 |
|------|----------|------|------|
| `Parse` | 使用者輸入，格式不固定 | 靈活，容錯性高 | 可能產生意外結果 |
| `ParseExact` | API 介面，格式固定 | 精確，可預測 | 嚴格，容錯性低 |
| `TryParse` | 不確定是否能解析成功 | 安全，不拋例外 | 需要檢查回傳值 |
| `TryParseExact` | 格式固定且需要安全性 | 精確且安全 | 程式碼較複雜 |

> ⚠️ **注意**: 在處理外部資料時，優先使用 `TryParseExact` 以確保資料品質和系統穩定性

---

## 時間計算

### 計算時間差

```csharp
DateTime orderTime = new DateTime(2025, 3, 20, 14, 30, 0);
DateTime currentTime = DateTime.Now;

// 計算天數差
TimeSpan difference = currentTime - orderTime;
double daysDifference = difference.TotalDays;

Console.WriteLine($"訂單建立時間: {orderTime}");
Console.WriteLine($"距今天數: {daysDifference:F2} 天");
Console.WriteLine($"距今小時: {difference.TotalHours:F2} 小時");
Console.WriteLine($"距今分鐘: {difference.TotalMinutes:F0} 分鐘");

// 其他有用的計算
Console.WriteLine($"完整天數: {difference.Days}");
Console.WriteLine($"剩餘小時: {difference.Hours}");
Console.WriteLine($"剩餘分鐘: {difference.Minutes}");
```

---

## 時間戳處理

### Unix 時間戳轉換

```csharp
// 方法一：使用 DateTime + DateTimeOffset
DateTime currentUtc = DateTime.UtcNow;
DateTime futureUtc = currentUtc.AddMinutes(15);

// 轉換為 Unix 時間戳
long unixSeconds = ((DateTimeOffset)futureUtc).ToUnixTimeSeconds();
long unixMilliseconds = ((DateTimeOffset)futureUtc).ToUnixTimeMilliseconds();

Console.WriteLine($"當前 UTC 時間: {currentUtc:yyyy-MM-dd HH:mm:ss}");
Console.WriteLine($"15 分鐘後: {futureUtc:yyyy-MM-dd HH:mm:ss}");
Console.WriteLine($"時間戳（秒）: {unixSeconds}");
Console.WriteLine($"時間戳（毫秒）: {unixMilliseconds}");

// 反向轉換：從時間戳轉回日期
DateTime fromUnix = DateTimeOffset.FromUnixTimeSeconds(unixSeconds).DateTime;
Console.WriteLine($"從時間戳還原: {fromUnix:yyyy-MM-dd HH:mm:ss}");
```

### 實用的時間戳工具方法

```csharp
public static class TimestampHelper
{
    public static long ToUnixTimestamp(this DateTime dateTime)
    {
        return ((DateTimeOffset)dateTime.ToUniversalTime()).ToUnixTimeSeconds();
    }
    
    public static DateTime FromUnixTimestamp(long timestamp)
    {
        return DateTimeOffset.FromUnixTimeSeconds(timestamp).DateTime;
    }
}
```

---

## 時區處理

### 使用 DateTimeOffset（推薦）

```csharp
// DateTimeOffset 包含時區資訊，更適合處理跨時區應用
var now = DateTimeOffset.Now;
var futureTime = now.AddMinutes(15);
long timestamp = futureTime.ToUnixTimeMilliseconds();

Console.WriteLine($"當前時間（含時區）: {now}");
Console.WriteLine($"15 分鐘後: {futureTime}");
Console.WriteLine($"時間戳（毫秒）: {timestamp}");

// 轉換到不同時區
var utcTime = now.ToUniversalTime();
var taipeiTime = now.ToOffset(TimeSpan.FromHours(8)); // UTC+8
var tokyoTime = now.ToOffset(TimeSpan.FromHours(9));  // UTC+9

Console.WriteLine($"UTC 時間: {utcTime}");
Console.WriteLine($"台北時間: {taipeiTime}");
Console.WriteLine($"東京時間: {tokyoTime}");
```

> 🌿 **優點**: `DateTimeOffset` 包含時區資訊，避免時區混淆問題

---

## 高精度時間

### 處理微秒級精度

```csharp
// JSON 序列化/反序列化範例
string json = "{\"Time\":\"2025-03-24T10:15:30.1234567\"}";
var options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
};

var obj = JsonSerializer.Deserialize<TimeModel>(json, options);
Console.WriteLine($"原始時間: {obj.Time.ToString("o")}"); // ISO 8601 格式
Console.WriteLine($"自訂格式: {obj.Time:yyyy-MM-dd HH:mm:ss.fffffff}");

public class TimeModel
{
    public DateTime Time { get; set; }
}
```

### 時間格式化選項

```csharp
DateTime now = DateTime.Now;

Console.WriteLine("=== 常用格式化 ===");
Console.WriteLine($"ISO 8601 (o): {now:o}");
Console.WriteLine($"標準日期 (d): {now:d}");
Console.WriteLine($"完整日期時間 (F): {now:F}");
Console.WriteLine($"一般日期時間 (G): {now:G}");
Console.WriteLine($"自訂格式: {now:yyyy年MM月dd日 HH時mm分ss秒}");
```

---

## 最佳實務

### ✅ 建議做法

1. **使用 `DateTimeOffset` 處理時區**
   ```csharp
   // 好的做法
   DateTimeOffset timestamp = DateTimeOffset.Now;
   ```

2. **使用 `TryParse` 進行安全解析**
   ```csharp
   // 好的做法
   if (DateTime.TryParse(input, out DateTime result)) { /* 使用 result */ }
   ```

3. **明確指定 UTC 或本地時間**
   ```csharp
   // 好的做法
   DateTime utcNow = DateTime.UtcNow;
   DateTime localNow = DateTime.Now;
   ```

4. **檢查 DateTime.Kind 避免時區混淆**
   ```csharp
   // 好的做法
   if (dateTime.Kind == DateTimeKind.Unspecified)
   {
       // 明確處理未指定種類的時間
   }
   ```

5. **使用 CultureInfo.InvariantCulture 進行資料儲存**
   ```csharp
   // 好的做法
   string dateForStorage = dateTime.ToString("yyyy-MM-dd HH:mm:ss", CultureInfo.InvariantCulture);
   ```

6. **使用 TryParseExact 處理外部資料**
   ```csharp
   // 好的做法
   if (DateTime.TryParseExact(input, "yyyy/MM/dd", CultureInfo.InvariantCulture, 
       DateTimeStyles.None, out DateTime result))
   {
       // 使用 result
   }
   ```

### ❌ 避免的做法

1. **不要混用 UTC 和本地時間**
   ```csharp
   // 不好的做法
   DateTime mixed = DateTime.Now.AddHours(DateTime.UtcNow.Hour);
   ```

2. **不要忽略時區問題**
   ```csharp
   // 不好的做法 - 在跨時區應用中
   DateTime serverTime = DateTime.Now; // 可能造成混淆
   ```

3. **不要忽略 DateTime.Kind**
   ```csharp
   // 不好的做法
   public void SaveTime(DateTime time)
   {
       // 沒有檢查 time.Kind，可能儲存錯誤的時間
       database.Save(time);
   }
   ```

4. **不要在國際化應用中使用系統文化設定**
   ```csharp
   // 不好的做法
   string dateString = dateTime.ToString(); // 會受系統語言影響
   ```

5. **不要使用 Parse 處理不可信的輸入**
   ```csharp
   // 不好的做法
   DateTime date = DateTime.Parse(userInput); // 可能拋出例外
   ```

---

## 關於 ISO 8601 格式中的 'T'

在 `2025-03-09T02:45` 這種格式中：
- **T** 是 ISO 8601 國際標準規定的**日期與時間分隔符號**
- 格式結構：`YYYY-MM-DDTHH:MM:SS`
- 目的是明確區分日期部分和時間部分
- 這是國際通用格式，確保不同系統間的相容性

### 範例對比
```
有 T 分隔：2025-03-09T14:30:00  ✅ 清楚易懂
無 T 分隔：2025-03-09 14:30:00  ❓ 可能造成歧義
```

---

## 常用擴充方法

```csharp
public static class DateTimeExtensions
{
    /// <summary>
    /// 判斷是否為今天
    /// </summary>
    public static bool IsToday(this DateTime date)
    {
        return date.Date == DateTime.Today;
    }
    
    /// <summary>
    /// 取得月份開始日期
    /// </summary>
    public static DateTime StartOfMonth(this DateTime date)
    {
        return new DateTime(date.Year, date.Month, 1);
    }
    
    /// <summary>
    /// 取得月份結束日期
    /// </summary>
    public static DateTime EndOfMonth(this DateTime date)
    {
        return date.StartOfMonth().AddMonths(1).AddDays(-1);
    }
    
    /// <summary>
    /// 格式化為友善顯示
    /// </summary>
    public static string ToFriendlyString(this DateTime date)
    {
        var diff = DateTime.Now - date;
        
        if (diff.TotalDays < 1) return "今天";
        if (diff.TotalDays < 2) return "昨天";
        if (diff.TotalDays < 7) return $"{(int)diff.TotalDays} 天前";
        
        return date.ToString("yyyy-MM-dd");
    }
}
```

---

*最後更新：2025-07-12*

