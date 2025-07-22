# C# Dictionary 完整使用指南

## 目錄
1. [Dictionary 基礎介紹](#dictionary-基礎介紹)
2. [建立與初始化](#建立與初始化)
3. [基本操作](#基本操作)
4. [進階查詢技巧](#進階查詢技巧)
5. [實戰範例：以 Dictionary 作為 Mapping 來源](#實戰範例以-dictionary-作為-mapping-來源)
6. [效能考量與最佳實務](#效能考量與最佳實務)
7. [常見陷阱與解決方案](#常見陷阱與解決方案)

---

## Dictionary 基礎介紹

### 什麼是 Dictionary？
Dictionary<TKey, TValue> 是 C# 中的鍵值對集合，提供快速的查找功能。每個鍵都是唯一的，對應一個值。

### 特點
- **快速查找**：平均時間複雜度 O(1)
- **唯一鍵值**：每個鍵只能出現一次
- **強型別**：編譯時期型別檢查
- **可變集合**：可以動態新增、修改、刪除元素

---

## 建立與初始化

### 基本建立方式

```csharp
// 空的 Dictionary
var dict = new Dictionary<string, int>();

// 指定初始容量
var dictWithCapacity = new Dictionary<string, int>(100);

// 使用集合初始化器
var colorDict = new Dictionary<string, string>
{
    { "red", "紅色" },
    { "blue", "藍色" },
    { "green", "綠色" }
};

// 使用索引初始化器 (C# 6.0+)
var modernDict = new Dictionary<string, string>
{
    ["apple"] = "蘋果",
    ["banana"] = "香蕉",
    ["orange"] = "橘子"
};
```

### 複雜型別的 Dictionary

```csharp
// Dictionary 的值為 List
var categoryProducts = new Dictionary<string, List<string>>
{
    { "水果", new List<string> { "蘋果", "香蕉", "橘子" } },
    { "蔬菜", new List<string> { "白菜", "紅蘿蔔", "洋蔥" } }
};

// Dictionary 的值為自定義物件
var userProfiles = new Dictionary<int, UserProfile>
{
    { 1, new UserProfile { Name = "張三", Email = "zhang@example.com" } },
    { 2, new UserProfile { Name = "李四", Email = "li@example.com" } }
};
```

---

## 基本操作

### 新增元素

```csharp
var dict = new Dictionary<string, int>();

// 使用 Add 方法
dict.Add("apple", 10);

// 使用索引器
dict["banana"] = 5;

// 使用 TryAdd (如果鍵不存在才新增)
dict.TryAdd("orange", 8);
```

### 取得元素

```csharp
var dict = new Dictionary<string, int>
{
    ["apple"] = 10,
    ["banana"] = 5
};

// 直接存取（可能拋出 KeyNotFoundException）
int appleCount = dict["apple"];

// 安全存取
if (dict.TryGetValue("apple", out int count))
{
    Console.WriteLine($"蘋果數量：{count}");
}

// 使用 ContainsKey 檢查
if (dict.ContainsKey("apple"))
{
    int value = dict["apple"];
}
```

### 修改元素

```csharp
var dict = new Dictionary<string, int> { ["apple"] = 10 };

// 直接修改
dict["apple"] = 15;

// 條件修改
if (dict.ContainsKey("apple"))
{
    dict["apple"] += 5;
}
```

### 刪除元素

```csharp
var dict = new Dictionary<string, int>
{
    ["apple"] = 10,
    ["banana"] = 5,
    ["orange"] = 8
};

// 刪除指定鍵
bool removed = dict.Remove("apple");

// 刪除所有元素
dict.Clear();

// 條件刪除
dict.Remove("banana", out int removedValue);
```

---

## 進階查詢技巧

### 遍歷 Dictionary

```csharp
var dict = new Dictionary<string, int>
{
    ["apple"] = 10,
    ["banana"] = 5,
    ["orange"] = 8
};

// 遍歷鍵值對
foreach (var kvp in dict)
{
    Console.WriteLine($"鍵：{kvp.Key}，值：{kvp.Value}");
}

// 僅遍歷鍵
foreach (string key in dict.Keys)
{
    Console.WriteLine($"鍵：{key}");
}

// 僅遍歷值
foreach (int value in dict.Values)
{
    Console.WriteLine($"值：{value}");
}
```

### LINQ 查詢

```csharp
var dict = new Dictionary<string, int>
{
    ["apple"] = 10,
    ["banana"] = 5,
    ["orange"] = 8,
    ["grape"] = 12
};

// 篩選值大於 7 的項目
var filtered = dict.Where(kvp => kvp.Value > 7)
                  .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);

// 找到第一個符合條件的項目
var firstMatch = dict.FirstOrDefault(kvp => kvp.Value > 10);

// 轉換為其他集合
var keyList = dict.Keys.ToList();
var valueArray = dict.Values.ToArray();

// 排序
var sortedByKey = dict.OrderBy(kvp => kvp.Key).ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
var sortedByValue = dict.OrderByDescending(kvp => kvp.Value).ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
```

---

## 實戰範例：以 Dictionary 作為 Mapping 來源

### 場景說明
在促銷系統中，需要根據啟用的平台類型，判斷對應的通路類型。使用 Dictionary 作為映射表，可以快速找到匹配的通路。

### 資料模型定義

```csharp
public enum PromotionEnginePlatformTypeEnum
{
    /// <summary>
    /// 官網
    /// </summary>
    Web = 1,

    /// <summary>
    /// IOS
    /// </summary>
    AppIOS = 2,

    /// <summary>
    /// Android
    /// </summary>
    AppAndroid = 4,

    /// <summary>
    /// 門市小幫手
    /// </summary>
    LocationWizard = 8,

    /// <summary>
    /// 線下資料交換
    /// </summary>
    InStore = 16
}

public enum PromoCodeChannelTypeEnum
{
    /// <summary>
    /// WebAndApp
    /// </summary>
    WebAndApp,

    /// <summary>
    /// APP 獨享
    /// </summary>
    App,

    /// <summary>
    /// APP/門市
    /// </summary>
    AppAndInStore,

    /// <summary>
    /// 僅限門市
    /// </summary>
    InStore,

    /// <summary>
    /// APP/官網/門市
    /// </summary>
    All
}

public class PromotionEnginePlatformTypeEntity
{
    /// <summary>
    /// 通路
    /// </summary>
    public string Platform { get; set; }

    /// <summary>
    /// 是否啟用
    /// </summary>
    public bool IsEnabled { get; set; }
}
```

### Dictionary Mapping 實作

```csharp
void Main()
{
    // 建立通路類型與平台類型的映射表
    var channelTypeDictionary = new Dictionary<string, List<string>>()
    {
        // WebAndApp 通路：支援 Android、iOS、Web
        { 
            nameof(PromoCodeChannelTypeEnum.WebAndApp), 
            new List<string> 
            { 
                nameof(PromotionEnginePlatformTypeEnum.AppAndroid), 
                nameof(PromotionEnginePlatformTypeEnum.AppIOS), 
                nameof(PromotionEnginePlatformTypeEnum.Web) 
            } 
        },
        
        // AppAndInStore 通路：支援 Android、iOS、門市、LocationWizard
        { 
            nameof(PromoCodeChannelTypeEnum.AppAndInStore), 
            new List<string> 
            { 
                nameof(PromotionEnginePlatformTypeEnum.AppAndroid), 
                nameof(PromotionEnginePlatformTypeEnum.AppIOS), 
                nameof(PromotionEnginePlatformTypeEnum.InStore), 
                nameof(PromotionEnginePlatformTypeEnum.LocationWizard) 
            } 
        },
        
        // App 通路：僅支援 Android、iOS
        { 
            nameof(PromoCodeChannelTypeEnum.App), 
            new List<string> 
            { 
                nameof(PromotionEnginePlatformTypeEnum.AppAndroid), 
                nameof(PromotionEnginePlatformTypeEnum.AppIOS) 
            } 
        },
        
        // InStore 通路：僅支援門市相關
        { 
            nameof(PromoCodeChannelTypeEnum.InStore), 
            new List<string> 
            { 
                nameof(PromotionEnginePlatformTypeEnum.InStore), 
                nameof(PromotionEnginePlatformTypeEnum.LocationWizard) 
            } 
        },
        
        // All 通路：支援所有平台
        { 
            nameof(PromoCodeChannelTypeEnum.All), 
            new List<string> 
            { 
                nameof(PromotionEnginePlatformTypeEnum.AppAndroid), 
                nameof(PromotionEnginePlatformTypeEnum.AppIOS), 
                nameof(PromotionEnginePlatformTypeEnum.InStore), 
                nameof(PromotionEnginePlatformTypeEnum.LocationWizard), 
                nameof(PromotionEnginePlatformTypeEnum.Web) 
            } 
        }
    };

    // 目標平台類型清單（模擬使用者啟用的平台）
    var targetPlatformTypeList = new List<PromotionEnginePlatformTypeEntity>
    {
        new PromotionEnginePlatformTypeEntity
        {
            IsEnabled = true,
            Platform = nameof(PromotionEnginePlatformTypeEnum.AppIOS)
        },
        new PromotionEnginePlatformTypeEntity
        {
            IsEnabled = true,
            Platform = nameof(PromotionEnginePlatformTypeEnum.AppAndroid)
        },
        new PromotionEnginePlatformTypeEntity
        {
            IsEnabled = true,
            Platform = nameof(PromotionEnginePlatformTypeEnum.Web)
        }
        // 其他平台已註解，表示未啟用
    };
    
    // 取得啟用的平台清單並排序
    var enabledPlatforms = targetPlatformTypeList
        .Where(platform => platform.IsEnabled)
        .Select(platform => platform.Platform)
        .OrderBy(platform => platform)
        .ToList();

    // 在 Dictionary 中尋找匹配的通路類型
    var matchedChannel = channelTypeDictionary
        .FirstOrDefault(kv => kv.Value.OrderBy(v => v).SequenceEqual(enabledPlatforms));
    
    // 判斷是否找到匹配項目，未找到則預設為 App
    var result = !matchedChannel.Equals(default(KeyValuePair<string, List<string>>)) 
        ? matchedChannel.Key 
        : nameof(PromoCodeChannelTypeEnum.App);
    
    Console.WriteLine($"匹配的通路類型：{result}");
}
```

## 效能考量與最佳實務

### 執行緒安全考量

```csharp
// Dictionary 不是執行緒安全的
// 需要執行緒安全時使用 ConcurrentDictionary
using System.Collections.Concurrent;

var concurrentDict = new ConcurrentDictionary<string, int>();

// 或使用鎖定機制
private readonly object _lock = new object();
private readonly Dictionary<string, int> _dict = new Dictionary<string, int>();

public void SafeAdd(string key, int value)
{
    lock (_lock)
    {
        _dict[key] = value;
    }
}
```

---

## 常見陷阱與解決方案

### 1. KeyNotFoundException 處理

```csharp
var dict = new Dictionary<string, int> { ["apple"] = 10 };

// ❌ 錯誤：直接存取可能拋出例外
try
{
    int value = dict["banana"]; // 拋出 KeyNotFoundException
}
catch (KeyNotFoundException)
{
    // 處理例外
}

// ✅ 正確：使用 TryGetValue
if (dict.TryGetValue("banana", out int value))
{
    Console.WriteLine(value);
}
else
{
    Console.WriteLine("鍵不存在");
}

// ✅ 正確：先檢查鍵是否存在
if (dict.ContainsKey("banana"))
{
    int value = dict["banana"];
}
```

### 2. 修改過程中遍歷

```csharp
var dict = new Dictionary<string, int>
{
    ["a"] = 1,
    ["b"] = 2,
    ["c"] = 3
};

// ❌ 錯誤：遍歷過程中修改 Dictionary
foreach (var kvp in dict)
{
    if (kvp.Value < 2)
    {
        dict.Remove(kvp.Key); // 拋出 InvalidOperationException
    }
}

// ✅ 正確：先收集要修改的鍵
var keysToRemove = dict.Where(kvp => kvp.Value < 2).Select(kvp => kvp.Key).ToList();
foreach (var key in keysToRemove)
{
    dict.Remove(key);
}

// ✅ 正確：使用 LINQ 建立新的 Dictionary
var filteredDict = dict.Where(kvp => kvp.Value >= 2).ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
```

### 3. 參考型別作為值的陷阱

```csharp
var dict = new Dictionary<string, List<int>>();

// ❌ 錯誤：可能會拋出 KeyNotFoundException
dict["key1"].Add(1);

// ✅ 正確：確保 List 存在
if (!dict.ContainsKey("key1"))
{
    dict["key1"] = new List<int>();
}
dict["key1"].Add(1);

// ✅ 更好：使用 TryGetValue
if (dict.TryGetValue("key1", out var list))
{
    list.Add(1);
}
else
{
    dict["key1"] = new List<int> { 1 };
}

// ✅ 最佳：使用輔助方法
public static void AddToListInDictionary<TKey, TValue>(
    Dictionary<TKey, List<TValue>> dict, 
    TKey key, 
    TValue value)
{
    if (dict.TryGetValue(key, out var list))
    {
        list.Add(value);
    }
    else
    {
        dict[key] = new List<TValue> { value };
    }
}
```

### 4. null 值處理

```csharp
var dict = new Dictionary<string, string>();

// null 鍵會拋出 ArgumentNullException
// dict[null] = "value"; // ❌

// null 值是允許的
dict["key"] = null; // ✅

// 檢查 null 值
if (dict.TryGetValue("key", out string value) && value != null)
{
    Console.WriteLine(value);
}
```

---