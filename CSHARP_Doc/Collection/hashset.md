# C# HashSet 完整實戰指南 🚀

> 掌握 HashSet 的精髓，讓您的程式碼效能飛起來！

## 目錄
1. [HashSet 基礎篇](#hashset-基礎篇)
   - 1.1 [什麼是 HashSet？](#什麼是-hashset)
   - 1.2 [HashSet vs List 效能大比拼](#hashset-vs-list-效能大比拼)
   - 1.3 [HashSet 的特性](#hashset-的特性)

2. [HashSet 核心操作篇](#hashset-核心操作篇)
   - 2.1 [建立與初始化](#建立與初始化)
   - 2.2 [新增與移除元素](#新增與移除元素)
   - 2.3 [查詢與檢查](#查詢與檢查)

3. [HashSet 集合運算篇](#hashset-集合運算篇)
   - 3.1 [SetEquals - 集合相等性比較](#setequals---集合相等性比較)
   - 3.2 [交集運算 (IntersectWith)](#交集運算-intersectwith)
   - 3.3 [聯集運算 (UnionWith)](#聯集運算-unionwith)
   - 3.4 [差集運算 (ExceptWith)](#差集運算-exceptwith)

4. [實務應用篇](#實務應用篇)
   - 4.1 [銷售通路比較實例](#銷售通路比較實例)
   - 4.2 [權限管理系統](#權限管理系統)
   - 4.3 [資料去重與篩選](#資料去重與篩選)

5. [進階技巧篇](#進階技巧篇)
   - 5.1 [自訂相等比較器](#自訂相等比較器)
   - 5.2 [HashSet 與 LINQ 結合](#hashset-與-linq-結合)
   - 5.3 [效能優化秘訣](#效能優化秘訣)

6. [常見陷阱與最佳實踐](#常見陷阱與最佳實踐)
   - 6.1 [常見錯誤](#常見錯誤)
   - 6.2 [最佳實踐指南](#最佳實踐指南)

---

## HashSet 基礎篇

### 什麼是 HashSet？

HashSet<T> 是 .NET 中的一個泛型集合類別，它使用雜湊表來儲存元素，具有以下特色：

🎯 **核心特性**
- **唯一性**: 不允許重複元素
- **無序性**: 不保證元素的順序
- **高效性**: O(1) 的平均查詢、新增、刪除時間複雜度

```csharp
// 基本使用範例
var numbers = new HashSet<int> { 1, 2, 3, 3, 4 }; // 重複的 3 只會保留一個
Console.WriteLine(numbers.Count); // 輸出: 4

foreach (var number in numbers)
{
    Console.WriteLine(number); // 順序可能是: 1, 2, 3, 4 (但不保證)
}
```

### HashSet vs List 效能大比拼

| 操作 | HashSet | List | 勝出者 |
|-----|---------|------|--------|
| 查詢元素 | O(1) | O(n) | 🏆 HashSet |
| 新增元素 | O(1) | O(1) | 🤝 平手 |
| 移除元素 | O(1) | O(n) | 🏆 HashSet |
| 檢查重複 | O(1) | O(n) | 🏆 HashSet |
| 保持順序 | ❌ | ✅ | 🏆 List |
| 允許重複 | ❌ | ✅ | 🏆 List |

### HashSet 的特性

```csharp
public class HashSetFeatures
{
    public static void DemonstrateFeatures()
    {
        var fruits = new HashSet<string>();
        
        // 1. 唯一性 - 重複元素會被忽略
        fruits.Add("蘋果");
        fruits.Add("香蕉");
        fruits.Add("蘋果"); // 這個不會被加入
        Console.WriteLine($"水果數量: {fruits.Count}"); // 輸出: 2
        
        // 2. 高效查詢
        bool hasApple = fruits.Contains("蘋果"); // O(1) 時間複雜度
        Console.WriteLine($"有蘋果嗎? {hasApple}"); // 輸出: True
        
        // 3. 高效移除
        bool removed = fruits.Remove("香蕉"); // O(1) 時間複雜度
        Console.WriteLine($"成功移除香蕉? {removed}"); // 輸出: True
    }
}
```

---

## HashSet 核心操作篇

### 建立與初始化

```csharp
public class HashSetCreation
{
    public static void CreationExamples()
    {
        // 1. 空的 HashSet
        var emptySet = new HashSet<int>();
        
        // 2. 使用集合初始化器
        var numbersSet = new HashSet<int> { 1, 2, 3, 4, 5 };
        
        // 3. 從其他集合建立
        var list = new List<int> { 1, 2, 2, 3, 3, 4 };
        var fromList = new HashSet<int>(list); // 自動去重
        Console.WriteLine(fromList.Count); // 輸出: 4
        
        // 4. 使用自訂比較器
        var caseInsensitive = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
        caseInsensitive.Add("Hello");
        caseInsensitive.Add("HELLO"); // 會被忽略，因為大小寫不敏感
        Console.WriteLine(caseInsensitive.Count); // 輸出: 1
        
        // 5. 從 LINQ 查詢建立
        var evenNumbers = Enumerable.Range(1, 10)
                                   .Where(x => x % 2 == 0)
                                   .ToHashSet();
    }
}
```

### 新增與移除元素

```csharp
public class HashSetModification
{
    public static void ModificationExamples()
    {
        var colors = new HashSet<string>();
        
        // 新增單個元素
        bool added1 = colors.Add("紅色");   // 回傳 true (成功新增)
        bool added2 = colors.Add("紅色");   // 回傳 false (已存在)
        
        // 批量新增
        var moreColors = new[] { "藍色", "綠色", "黃色" };
        foreach (var color in moreColors)
        {
            colors.Add(color);
        }
        
        // 使用 UnionWith 批量新增
        colors.UnionWith(new[] { "紫色", "橘色" });
        
        // 移除元素
        bool removed = colors.Remove("藍色"); // 回傳 true (成功移除)
        
        // 條件移除
        colors.RemoveWhere(color => color.Contains("色"));
        
        // 清空所有元素
        colors.Clear();
        
        Console.WriteLine($"最終顏色數量: {colors.Count}"); // 輸出: 0
    }
}
```

### 查詢與檢查

```csharp
public class HashSetQuerying
{
    public static void QueryingExamples()
    {
        var numbers = new HashSet<int> { 1, 3, 5, 7, 9 };
        
        // 檢查是否包含特定元素
        bool contains5 = numbers.Contains(5); // true
        bool contains2 = numbers.Contains(2); // false
        
        // 檢查是否為空
        bool isEmpty = numbers.Count == 0; // false
        
        // 檢查大小
        int count = numbers.Count; // 5
        
        // 轉換為陣列或列表
        int[] array = numbers.ToArray();
        List<int> list = numbers.ToList();
        
        // 複製到現有陣列
        int[] destination = new int[10];
        numbers.CopyTo(destination, 2); // 從索引 2 開始複製
    }
}
```

---

## HashSet 集合運算篇

### SetEquals - 集合相等性比較

這是您範例中遇到的核心概念！讓我們深入了解：

```csharp
public class SetEqualsExample
{
    public static void DemonstrateSetEquals()
    {
        // 定義離線銷售通路
        HashSet<SalesChannelEnum> offlineSalesChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.LocationWizard,
            SalesChannelEnum.InStore
        };
        
        // 測試案例 1: 完全相同的集合
        var rule1 = new SalesRule
        {
            MatchedSalesChannels = new List<SalesChannelEnum> 
            { 
                SalesChannelEnum.LocationWizard, 
                SalesChannelEnum.InStore 
            }
        };
        
        bool isEqual1 = rule1.MatchedSalesChannels.ToHashSet().SetEquals(offlineSalesChannels);
        Console.WriteLine($"案例1 - 相等嗎? {isEqual1}"); // 輸出: True
        
        // 測試案例 2: 順序不同但內容相同
        var rule2 = new SalesRule
        {
            MatchedSalesChannels = new List<SalesChannelEnum> 
            { 
                SalesChannelEnum.InStore,           // 順序不同
                SalesChannelEnum.LocationWizard 
            }
        };
        
        bool isEqual2 = rule2.MatchedSalesChannels.ToHashSet().SetEquals(offlineSalesChannels);
        Console.WriteLine($"案例2 - 相等嗎? {isEqual2}"); // 輸出: True (順序不影響)
        
        // 測試案例 3: 包含額外元素
        var rule3 = new SalesRule
        {
            MatchedSalesChannels = new List<SalesChannelEnum> 
            { 
                SalesChannelEnum.LocationWizard,
                SalesChannelEnum.InStore,
                SalesChannelEnum.Web              // 額外元素
            }
        };
        
        bool isEqual3 = rule3.MatchedSalesChannels.ToHashSet().SetEquals(offlineSalesChannels);
        Console.WriteLine($"案例3 - 相等嗎? {isEqual3}"); // 輸出: False
        
        // 測試案例 4: 空集合
        var rule4 = new SalesRule(); // 空的 MatchedSalesChannels
        
        bool isEqual4 = rule4.MatchedSalesChannels.ToHashSet().SetEquals(offlineSalesChannels);
        Console.WriteLine($"案例4 - 相等嗎? {isEqual4}"); // 輸出: False
    }
}

public class SalesRule
{
    public IList<SalesChannelEnum> MatchedSalesChannels { get; set; } = new List<SalesChannelEnum>();
}

[Flags]
public enum SalesChannelEnum
{
    Web = 1,
    AppIOS = 2,
    AppAndroid = 4,
    LocationWizard = 8,
    InStore = 0x10  // 16 in decimal
}
```

**SetEquals 的核心概念：**

🎯 **兩個集合相等的條件**
1. 包含相同的元素
2. 元素數量相同
3. 順序不重要
4. 重複元素會被忽略

### 交集運算 (IntersectWith)

找出兩個集合的共同元素：

```csharp
public class IntersectionExample
{
    public static void DemonstrateIntersection()
    {
        var onlineChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.Web,
            SalesChannelEnum.AppIOS,
            SalesChannelEnum.AppAndroid
        };
        
        var mobileChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.AppIOS,
            SalesChannelEnum.AppAndroid,
            SalesChannelEnum.LocationWizard
        };
        
        // 找出共同的行動通路
        var commonMobileChannels = new HashSet<SalesChannelEnum>(onlineChannels);
        commonMobileChannels.IntersectWith(mobileChannels);
        
        Console.WriteLine("共同的行動通路:");
        foreach (var channel in commonMobileChannels)
        {
            Console.WriteLine($"- {channel}");
        }
        // 輸出: AppIOS, AppAndroid
    }
}
```

### 聯集運算 (UnionWith)

合併兩個集合的所有元素：

```csharp
public class UnionExample
{
    public static void DemonstrateUnion()
    {
        var basicChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.Web,
            SalesChannelEnum.InStore
        };
        
        var premiumChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.AppIOS,
            SalesChannelEnum.AppAndroid,
            SalesChannelEnum.LocationWizard
        };
        
        // 合併所有通路
        var allChannels = new HashSet<SalesChannelEnum>(basicChannels);
        allChannels.UnionWith(premiumChannels);
        
        Console.WriteLine($"總共有 {allChannels.Count} 個銷售通路:");
        foreach (var channel in allChannels)
        {
            Console.WriteLine($"- {channel}");
        }
    }
}
```

### 差集運算 (ExceptWith)

從集合中移除另一個集合的元素：

```csharp
public class ExceptExample
{
    public static void DemonstrateExcept()
    {
        var allChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.Web,
            SalesChannelEnum.AppIOS,
            SalesChannelEnum.AppAndroid,
            SalesChannelEnum.LocationWizard,
            SalesChannelEnum.InStore
        };
        
        var onlineChannels = new HashSet<SalesChannelEnum>
        {
            SalesChannelEnum.Web,
            SalesChannelEnum.AppIOS,
            SalesChannelEnum.AppAndroid
        };
        
        // 找出非線上通路
        var offlineChannels = new HashSet<SalesChannelEnum>(allChannels);
        offlineChannels.ExceptWith(onlineChannels);
        
        Console.WriteLine("離線通路:");
        foreach (var channel in offlineChannels)
        {
            Console.WriteLine($"- {channel}");
        }
        // 輸出: LocationWizard, InStore
    }
}
```

---

## 實務應用篇

### 銷售通路比較實例

基於您提供的程式碼，讓我們建立一個完整的銷售通路管理系統：

```csharp
public class SalesChannelManager
{
    private static readonly HashSet<SalesChannelEnum> OfflineSalesChannels = new HashSet<SalesChannelEnum>
    {
        SalesChannelEnum.LocationWizard,
        SalesChannelEnum.InStore
    };
    
    private static readonly HashSet<SalesChannelEnum> OnlineSalesChannels = new HashSet<SalesChannelEnum>
    {
        SalesChannelEnum.Web,
        SalesChannelEnum.AppIOS,
        SalesChannelEnum.AppAndroid
    };
    
    public static void AnalyzeSalesRule(SalesRule rule)
    {
        var ruleChannels = rule.MatchedSalesChannels.ToHashSet();
        
        // 🔍 檢查是否為純離線通路
        if (ruleChannels.SetEquals(OfflineSalesChannels))
        {
            Console.WriteLine("✅ 這是純離線銷售規則");
            return;
        }
        
        // 🔍 檢查是否為純線上通路
        if (ruleChannels.SetEquals(OnlineSalesChannels))
        {
            Console.WriteLine("🌐 這是純線上銷售規則");
            return;
        }
        
        // 🔍 檢查是否包含離線通路
        if (ruleChannels.Overlaps(OfflineSalesChannels))
        {
            var offlineInRule = new HashSet<SalesChannelEnum>(ruleChannels);
            offlineInRule.IntersectWith(OfflineSalesChannels);
            
            Console.WriteLine($"🏪 包含離線通路: {string.Join(", ", offlineInRule)}");
        }
        
        // 🔍 檢查是否包含線上通路
        if (ruleChannels.Overlaps(OnlineSalesChannels))
        {
            var onlineInRule = new HashSet<SalesChannelEnum>(ruleChannels);
            onlineInRule.IntersectWith(OnlineSalesChannels);
            
            Console.WriteLine($"💻 包含線上通路: {string.Join(", ", onlineInRule)}");
        }
        
        // 🔍 找出未涵蓋的通路
        var allChannels = new HashSet<SalesChannelEnum>(Enum.GetValues<SalesChannelEnum>());
        allChannels.ExceptWith(ruleChannels);
        
        if (allChannels.Any())
        {
            Console.WriteLine($"⚠️  未涵蓋的通路: {string.Join(", ", allChannels)}");
        }
    }
    
    public static void Main()
    {
        // 您的原始範例
        var rule = new SalesRule
        {
            MatchedSalesChannels = new List<SalesChannelEnum> 
            { 
                SalesChannelEnum.Web, 
                SalesChannelEnum.AppAndroid, 
                SalesChannelEnum.AppIOS 
            }
        };
        
        var brule = new SalesRule(); // 空規則
        
        Console.WriteLine("=== 分析 rule ===");
        AnalyzeSalesRule(rule);
        
        Console.WriteLine("\n=== 分析 brule ===");
        AnalyzeSalesRule(brule);
        
        // 檢查您原本的邏輯問題
        if (brule.MatchedSalesChannels.ToHashSet().SetEquals(OfflineSalesChannels))
        {
            Console.WriteLine("❌ brule 符合離線通路條件"); // 這不會執行
        }
        else
        {
            Console.WriteLine("✅ brule 不符合離線通路條件"); // 這會執行
        }
    }
}
```

### 權限管理系統

```csharp
[Flags]
public enum Permission
{
    Read = 1,
    Write = 2,
    Delete = 4,
    Admin = 8,
    Execute = 16
}

public class PermissionManager
{
    private readonly HashSet<Permission> _userPermissions;
    
    public PermissionManager(IEnumerable<Permission> permissions)
    {
        _userPermissions = permissions.ToHashSet();
    }
    
    public bool HasPermission(Permission permission)
    {
        return _userPermissions.Contains(permission);
    }
    
    public bool HasAllPermissions(IEnumerable<Permission> requiredPermissions)
    {
        return requiredPermissions.ToHashSet().IsSubsetOf(_userPermissions);
    }
    
    public bool HasAnyPermission(IEnumerable<Permission> permissions)
    {
        return _userPermissions.Overlaps(permissions);
    }
    
    public void GrantPermissions(IEnumerable<Permission> permissions)
    {
        _userPermissions.UnionWith(permissions);
    }
    
    public void RevokePermissions(IEnumerable<Permission> permissions)
    {
        _userPermissions.ExceptWith(permissions);
    }
    
    public IReadOnlySet<Permission> GetPermissions()
    {
        return _userPermissions;
    }
}

public class PermissionExample
{
    public static void DemonstratePermissions()
    {
        var user = new PermissionManager(new[] { Permission.Read, Permission.Write });
        
        // 檢查單一權限
        Console.WriteLine($"有讀取權限? {user.HasPermission(Permission.Read)}"); // True
        Console.WriteLine($"有管理員權限? {user.HasPermission(Permission.Admin)}"); // False
        
        // 檢查是否擁有所有必要權限
        var requiredForEdit = new[] { Permission.Read, Permission.Write };
        Console.WriteLine($"可以編輯? {user.HasAllPermissions(requiredForEdit)}"); // True
        
        var requiredForAdmin = new[] { Permission.Admin, Permission.Delete };
        Console.WriteLine($"可以管理? {user.HasAllPermissions(requiredForAdmin)}"); // False
        
        // 授予新權限
        user.GrantPermissions(new[] { Permission.Delete });
        Console.WriteLine($"授予刪除權限後，有刪除權限? {user.HasPermission(Permission.Delete)}"); // True
    }
}
```

### 資料去重與篩選

```csharp
public class DataDeduplication
{
    public class Customer
    {
        public int Id { get; set; }
        public string Email { get; set; }
        public string Name { get; set; }
        
        public override bool Equals(object obj)
        {
            return obj is Customer customer && Email == customer.Email;
        }
        
        public override int GetHashCode()
        {
            return Email?.GetHashCode() ?? 0;
        }
    }
    
    public static void DemonstrateDeduplication()
    {
        var customers = new List<Customer>
        {
            new Customer { Id = 1, Email = "john@example.com", Name = "John Doe" },
            new Customer { Id = 2, Email = "jane@example.com", Name = "Jane Smith" },
            new Customer { Id = 3, Email = "john@example.com", Name = "John Doe" }, // 重複
            new Customer { Id = 4, Email = "bob@example.com", Name = "Bob Johnson" },
            new Customer { Id = 5, Email = "jane@example.com", Name = "Jane Smith" }  // 重複
        };
        
        // 使用 HashSet 自動去重
        var uniqueCustomers = customers.ToHashSet();
        
        Console.WriteLine($"原始客戶數量: {customers.Count}");        // 5
        Console.WriteLine($"去重後客戶數量: {uniqueCustomers.Count}"); // 3
        
        // 找出重複的客戶
        var seen = new HashSet<Customer>();
        var duplicates = new HashSet<Customer>();
        
        foreach (var customer in customers)
        {
            if (!seen.Add(customer))
            {
                duplicates.Add(customer);
            }
        }
        
        Console.WriteLine($"重複客戶數量: {duplicates.Count}");
    }
}
```

---

## 進階技巧篇

### 自訂相等比較器

```csharp
public class CaseInsensitiveStringComparer : IEqualityComparer<string>
{
    public bool Equals(string x, string y)
    {
        return string.Equals(x, y, StringComparison.OrdinalIgnoreCase);
    }
    
    public int GetHashCode(string obj)
    {
        return obj?.ToLowerInvariant().GetHashCode() ?? 0;
    }
}

public class ProductComparer : IEqualityComparer<Product>
{
    public bool Equals(Product x, Product y)
    {
        if (x == null || y == null) return x == y;
        return x.SKU == y.SKU;
    }
    
    public int GetHashCode(Product obj)
    {
        return obj?.SKU?.GetHashCode() ?? 0;
    }
}

public class Product
{
    public string SKU { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class CustomComparerExample
{
    public static void DemonstrateCustomComparers()
    {
        // 不區分大小寫的字串 HashSet
        var caseInsensitiveSet = new HashSet<string>(new CaseInsensitiveStringComparer());
        caseInsensitiveSet.Add("Hello");
        caseInsensitiveSet.Add("HELLO"); // 會被忽略
        caseInsensitiveSet.Add("hello"); // 會被忽略
        
        Console.WriteLine($"不區分大小寫集合大小: {caseInsensitiveSet.Count}"); // 1
        
        // 使用自訂產品比較器
        var products = new HashSet<Product>(new ProductComparer())
        {
            new Product { SKU = "ABC123", Name = "產品A", Price = 100 },
            new Product { SKU = "ABC123", Name = "產品A更新", Price = 120 }, // 相同 SKU，會被忽略
            new Product { SKU = "DEF456", Name = "產品B", Price = 200 }
        };
        
        Console.WriteLine($"產品集合大小: {products.Count}"); // 2
    }
}
```

### HashSet 與 LINQ 結合

```csharp
public class HashSetLinqExample
{
    public static void DemonstrateLinqIntegration()
    {
        var numbers = Enumerable.Range(1, 100).ToHashSet();
        
        // 使用 LINQ 查詢 HashSet
        var evenNumbers = numbers.Where(n => n % 2 == 0).ToHashSet();
        var primeNumbers = numbers.Where(IsPrime).ToHashSet();
        
        // 找出既是偶數又是質數的數字
        var evenPrimes = evenNumbers.Intersect(primeNumbers).ToHashSet();
        Console.WriteLine($"偶質數: {string.Join(", ", evenPrimes)}"); // 2
        
        // 使用 HashSet 優化 LINQ 查詢
        var targetIds = new HashSet<int> { 1, 3, 5, 7, 9 };
        var customers = GetCustomers();
        
        // O(1) 查詢而非 O(n)
        var filteredCustomers = customers.Where(c => targetIds.Contains(c.Id)).ToList();
        
        // 分組並轉換為 HashSet
        var customersByCity = customers
            .GroupBy(c => c.City)
            .ToDictionary(g => g.Key, g => g.Select(c => c.Name).ToHashSet());
    }
    
    private static bool IsPrime(int number)
    {
        if (number < 2) return false;
        for (int i = 2; i <= Math.Sqrt(number); i++)
        {
            if (number % i == 0) return false;
        }
        return true;
    }
    
    private static List<Customer> GetCustomers()
    {
        return new List<Customer>
        {
            new Customer { Id = 1, Name = "Alice", City = "台北" },
            new Customer { Id = 2, Name = "Bob", City = "台中" },
            new Customer { Id = 3, Name = "Charlie", City = "台北" }
        };
    }
    
    public class Customer
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string City { get; set; }
    }
}
```

### 效能優化秘訣

```csharp
public class PerformanceOptimization
{
    public static void DemonstrateOptimizations()
    {
        // 1. 預設容量以避免重新配置
        var largeSet = new HashSet<int>(10000);
        
        // 2. 使用 HashSet 進行快速查詢
        var validIds = new HashSet<int>(Enumerable.Range(1, 1000));
        var items = GetLargeItemList();
        
        // ❌ 慢：每次都要搜尋整個列表 O(n²)
        var slowFiltered = items.Where(item => 
            Enumerable.Range(1, 1000).Contains(item.Id)).ToList();
        
        // ✅ 快：HashSet 查詢 O(1)
        var fastFiltered = items.Where(item => validIds.Contains(item.Id)).ToList();
        
        // 3. 批量操作比逐一操作更有效率
        var source1 = new HashSet<int> { 1, 2, 3 };
        var source2 = new HashSet<int> { 3, 4, 5 };
        
        // ❌ 效率較低
        foreach (var item in source2)
        {
            source1.Add(item);
        }
        
        // ✅ 更有效率
        source1.UnionWith(source2);
        
        // 4. 使用適當的初始容量
        var estimatedSize = 5000;
        var optimizedSet = new HashSet<string>(estimatedSize);
    }
    
    private static List<Item> GetLargeItemList()
    {
        return Enumerable.Range(1, 10000)
            .Select(i => new Item { Id = i, Name = $"Item {i}" })
            .ToList();
    }
    
    public class Item
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}
```

---

## 常見陷阱與最佳實踐

### 常見錯誤

```csharp
public class CommonMistakes
{
    public static void DemonstrateMistakes()
    {
        // ❌ 錯誤 1: 修改 HashSet 時進行迭代
        var numbers = new HashSet<int> { 1, 2, 3, 4, 5 };
        
        // 這會拋出 InvalidOperationException
        try
        {
            foreach (var number in numbers)
            {
                if (number % 2 == 0)
                {
                    numbers.Remove(number); // ❌ 在迭代時修改集合
                }
            }
        }
        catch (InvalidOperationException ex)
        {
            Console.WriteLine($"錯誤: {ex.Message}");
        }
        
        // ✅ 正確做法：使用 RemoveWhere 或先收集要移除的項目
        numbers.RemoveWhere(n => n % 2 == 0);
        
        // 或者
        var toRemove = numbers.Where(n => n % 2 == 0).ToList();
        foreach (var item in toRemove)
        {
            numbers.Remove(item);
        }
        
        // ❌ 錯誤 2: 假設 HashSet 有順序
        var fruits = new HashSet<string> { "蘋果", "香蕉", "橘子" };
        // 不要依賴元素的順序！
        
        // ❌ 錯誤 3: 在可變物件中使用可變屬性作為雜湊依據
        var badSet = new HashSet<MutablePerson>();
        var person = new MutablePerson { Name = "John" };
        badSet.Add(person);
        
        person.Name = "Jane"; // 改變用於雜湊的屬性
        bool contains = badSet.Contains(person); // 可能找不到！
        
        // ✅ 正確做法：使用不可變物件或不會改變的屬性
        var goodSet = new HashSet<ImmutablePerson>();
    }
    
    public class MutablePerson
    {
        public string Name { get; set; }
        
        public override bool Equals(object obj)
        {
            return obj is MutablePerson person && Name == person.Name;
        }
        
        public override int GetHashCode()
        {
            return Name?.GetHashCode() ?? 0;
        }
    }
    
    public class ImmutablePerson
    {
        public string Name { get; }
        
        public ImmutablePerson(string name)
        {
            Name = name;
        }
        
        public override bool Equals(object obj)
        {
            return obj is ImmutablePerson person && Name == person.Name;
        }
        
        public override int GetHashCode()
        {
            return Name?.GetHashCode() ?? 0;
        }
    }
}
```

### 最佳實踐指南

```csharp
public class BestPractices
{
    // ✅ 1. 使用唯讀介面公開 HashSet
    private readonly HashSet<string> _internalSet = new();
    
    public IReadOnlySet<string> Items => _internalSet;
    
    // ✅ 2. 提供安全的修改方法
    public bool AddItem(string item)
    {
        return _internalSet.Add(item);
    }
    
    public bool RemoveItem(string item)
    {
        return _internalSet.Remove(item);
    }
    
    // ✅ 3. 實作適當的相等性比較
    public class GoodProduct : IEquatable<GoodProduct>
    {
        public string SKU { get; }
        public string Name { get; set; } // 不用於相等性比較
        
        public GoodProduct(string sku, string name)
        {
            SKU = sku ?? throw new ArgumentNullException(nameof(sku));
            Name = name;
        }
        
        public bool Equals(GoodProduct other)
        {
            return other != null && SKU == other.SKU;
        }
        
        public override bool Equals(object obj)
        {
            return Equals(obj as GoodProduct);
        }
        
        public override int GetHashCode()
        {
            return SKU.GetHashCode();
        }
    }
    
    // ✅ 4. 使用工廠方法建立特殊用途的 HashSet
    public static HashSet<string> CreateCaseInsensitiveSet()
    {
        return new HashSet<string>(StringComparer.OrdinalIgnoreCase);
    }
    
    public static HashSet<T> CreateFromEnum<T>() where T : Enum
    {
        return Enum.GetValues<T>().ToHashSet();
    }
    
    // ✅ 5. 提供方便的擴充方法
    public static class HashSetExtensions
    {
        public static HashSet<T> ToHashSet<T>(this IEnumerable<T> source, IEqualityComparer<T> comparer)
        {
            return new HashSet<T>(source, comparer);
        }
        
        public static bool AddRange<T>(this HashSet<T> hashSet, IEnumerable<T> items)
        {
            bool added = false;
            foreach (var item in items)
            {
                if (hashSet.Add(item))
                    added = true;
            }
            return added;
        }
        
        public static HashSet<T> Clone<T>(this HashSet<T> original)
        {
            return new HashSet<T>(original, original.Comparer);
        }
    }
}
```