# C# Random 隨機數完整實務指南 🎲

> 從抽名字到密碼學，掌握各種隨機數的正確用法！

## 目錄
1. [Random 基礎應用](#random-基礎應用)
   - 1.1 [簡單抽籤範例](#簡單抽籤範例)
   - 1.2 [常見使用場景](#常見使用場景)
   - 1.3 [基本方法介紹](#基本方法介紹)

2. [偽隨機數原理](#偽隨機數原理)
   - 2.1 [什麼是偽隨機數？](#什麼是偽隨機數)
   - 2.2 [種子（Seed）概念](#種子seed概念)
   - 2.3 [PRNG 運作原理](#prng-運作原理)

3. [種子使用策略](#種子使用策略)
   - 3.1 [固定種子的用途](#固定種子的用途)
   - 3.2 [動態種子的選擇](#動態種子的選擇)
   - 3.3 [實際應用場景](#實際應用場景)

4. [安全隨機數](#安全隨機數)
   - 4.1 [Random vs RandomNumberGenerator](#random-vs-randomnumbergenerator)
   - 4.2 [密碼學級隨機數](#密碼學級隨機數)
   - 4.3 [使用場景選擇](#使用場景選擇)

---

## Random 基礎應用

### 簡單抽籤範例

> 🧪 **抽名字實作**：最常見的 Random 應用場景

```csharp
// 基本抽名字程式
string[] names = new string[] { "Bill", "Allen", "Lily", "Jacky", "Willu", "Jill" };
var random = new Random(DateTime.Now.Millisecond);
var name = names.ElementAt(random.Next(0, names.Length));
Console.WriteLine($"抽到你: {name}");
```

**更簡潔的寫法**
```csharp
string[] names = { "Bill", "Allen", "Lily", "Jacky", "Willu", "Jill" };
var random = new Random();
var name = names[random.Next(names.Length)];  // 更直接的陣列存取
Console.WriteLine($"抽到你: {name}");
```

### 常見使用場景

**數字範圍隨機**
```csharp
var random = new Random();

// 擲骰子 (1-6)
int dice = random.Next(1, 7);
Console.WriteLine($"擲骰子: {dice}");

// 百分比機率
int percentage = random.Next(0, 101);
Console.WriteLine($"機率: {percentage}%");

// 浮點數隨機 (0.0 - 1.0)
double ratio = random.NextDouble();
Console.WriteLine($"比例: {ratio:F2}");
```

**陣列洗牌**
```csharp
static void Shuffle<T>(T[] array)
{
    var random = new Random();
    for (int i = array.Length - 1; i > 0; i--)
    {
        int j = random.Next(i + 1);
        (array[i], array[j]) = (array[j], array[i]);  // Tuple 交換
    }
}

// 使用範例
string[] cards = { "♠A", "♥K", "♦Q", "♣J" };
Shuffle(cards);
Console.WriteLine(string.Join(", ", cards));
```

### 基本方法介紹

| 方法 | 用途 | 範例 |
|------|------|------|
| `Next()` | 非負整數 | `random.Next()` |
| `Next(max)` | 0 到 max-1 | `random.Next(10)` // 0-9 |
| `Next(min, max)` | min 到 max-1 | `random.Next(1, 7)` // 1-6 |
| `NextDouble()` | 0.0 到 1.0 | `random.NextDouble()` |
| `NextBytes(byte[])` | 填充位元組陣列 | `random.NextBytes(buffer)` |

---

## 偽隨機數原理

### 什麼是偽隨機數？

> 🎲 **重要概念**：電腦的隨機數不是「真的亂」！

電腦產生的所謂「亂數」其實是靠**公式算出來的**，所以叫做 **偽隨機數（Pseudo-Random Numbers, PRNG）**。

**特性說明**
- ✅ **看起來很亂**：通過統計測試
- ❌ **可預測**：知道公式和種子就能算出
- ✅ **可重現**：相同條件產生相同序列
- ⚡ **效能高**：純數學運算

### 種子（Seed）概念

> 🌱 **種子就是「起點」**：PRNG 的原理是每次都從上一個數字推算下一個

```csharp
// 同樣的種子 → 同樣的序列
var rand1 = new Random(42);
var rand2 = new Random(42);

Console.WriteLine(rand1.Next()); // 1608637542
Console.WriteLine(rand2.Next()); // 1608637542 (相同!)

Console.WriteLine(rand1.Next()); // 602680029
Console.WriteLine(rand2.Next()); // 602680029 (相同!)
```

**種子來源比較**
```csharp
// 固定種子 - 結果可重現
var rand1 = new Random(123);

// 時間種子 - 每次不同
var rand2 = new Random(DateTime.Now.Millisecond);

// 無參數 - 內部使用時間種子
var rand3 = new Random();
```

### PRNG 運作原理

> ⚙️ **.NET System.Random 使用線性同餘產生器（LCG）**

**數學公式**
```
Xₙ₊₁ = (a × Xₙ + c) mod m

其中：
- Xₙ = 目前的數字
- Xₙ₊₁ = 下一個數字  
- a, c, m = 固定參數
- X₀ = 種子（起始值）
```

**實際運作示意**
```csharp
// 概念性展示（實際參數不同）
class SimplePRNG
{
    private uint seed;
    
    public SimplePRNG(uint seed)
    {
        this.seed = seed;
    }
    
    public uint Next()
    {
        seed = (1103515245u * seed + 12345u) % (1u << 31);
        return seed;
    }
}
```

---

## 種子使用策略

### 固定種子的用途

> ✅ **情境：想要結果可重現**

**單元測試**
```csharp
[Test]
public void TestRandomBehavior()
{
    var random = new Random(42);  // 固定種子確保測試穩定
    var result = GenerateRandomData(random);
    
    Assert.AreEqual(expectedValue, result);
}
```

**遊戲地圖生成**
```csharp
public class MapGenerator
{
    public Map GenerateMap(int seed)
    {
        var random = new Random(seed);
        // 相同種子總是生成相同地圖
        return CreateRandomMap(random);
    }
}

// 玩家可以分享地圖種子
var map1 = generator.GenerateMap(12345);
var map2 = generator.GenerateMap(12345);  // 完全相同的地圖
```

**科學模擬**
```csharp
public class MonteCarloSimulation
{
    public double RunSimulation(int iterations, int seed)
    {
        var random = new Random(seed);
        // 確保實驗可重現
        return PerformCalculation(random, iterations);
    }
}
```

### 動態種子的選擇

> ❌ **情境：要每次都不一樣**

**常見動態種子來源**
```csharp
// 毫秒時間（最常見）
var rand1 = new Random(DateTime.Now.Millisecond);

// 完整時間戳記
var rand2 = new Random((int)DateTime.Now.Ticks);

// Guid 雜湊值
var rand3 = new Random(Guid.NewGuid().GetHashCode());

// 無參數（推薦）- Random 內部處理
var rand4 = new Random();
```

### 實際應用場景

**遊戲開發範例**
```csharp
public class GameManager
{
    private Random gameRandom;
    private Random mapRandom;
    
    public GameManager(int? mapSeed = null)
    {
        // 遊戲事件用隨機種子
        gameRandom = new Random();
        
        // 地圖生成用固定種子（可選）
        mapRandom = mapSeed.HasValue ? 
            new Random(mapSeed.Value) : 
            new Random();
    }
    
    public void DropItem()
    {
        // 每次都不同的掉寶
        if (gameRandom.NextDouble() < 0.1)  // 10% 機率
        {
            Console.WriteLine("獲得稀有道具！");
        }
    }
}
```

---

## 安全隨機數

### Random vs RandomNumberGenerator

> 🔓 **安全等級比較**：不是所有隨機數都一樣安全！

| 特性 | Random | RandomNumberGenerator |
|------|--------|----------------------|
| **用途** | 一般用途 | 密碼學用途 |
| **可預測性** | 可預測 | 不可預測 |
| **效能** | 高 | 較低 |
| **安全性** | 低 | 高 |
| **熵來源** | 算法 | 系統熵池 |

### 密碼學級隨機數

**RandomNumberGenerator 使用方式**
```csharp
using System.Security.Cryptography;

// 產生安全的隨機位元組
byte[] secureBytes = new byte[16];
using var rng = RandomNumberGenerator.Create();
rng.GetBytes(secureBytes);

// 轉換為整數
int secureInt = BitConverter.ToInt32(secureBytes, 0);
Console.WriteLine($"安全隨機數: {secureInt}");
```

**產生安全的隨機字串**
```csharp
public static string GenerateSecureToken(int length)
{
    byte[] randomBytes = new byte[length];
    using var rng = RandomNumberGenerator.Create();
    rng.GetBytes(randomBytes);
    
    return Convert.ToBase64String(randomBytes)
                  .Replace("+", "")
                  .Replace("/", "")
                  .Replace("=", "")
                  .Substring(0, length);
}

// 使用範例
string token = GenerateSecureToken(32);
Console.WriteLine($"安全 Token: {token}");
```

**產生安全的數字範圍**
```csharp
public static int GetSecureRandomInRange(int minValue, int maxValue)
{
    if (minValue >= maxValue)
        throw new ArgumentException("minValue must be less than maxValue");
    
    uint range = (uint)(maxValue - minValue);
    byte[] randomBytes = new byte[4];
    
    using var rng = RandomNumberGenerator.Create();
    rng.GetBytes(randomBytes);
    
    uint randomValue = BitConverter.ToUInt32(randomBytes, 0);
    return (int)(randomValue % range) + minValue;
}
```

### 使用場景選擇

> 📌 **什麼時候要用「更安全的亂數」？**

| 用途 | 適合方案 | 理由 |
|------|----------|------|
| 遊戲道具掉落 | `Random` | 可重現、效能高 |
| 隨機洗牌 | `Random` | 足夠隨機 |
| 抽獎活動 | `Random` 或 `RandomNumberGenerator` | 看公平性要求 |
| **加密金鑰** | `RandomNumberGenerator` | PRNG 太可預測，有被破解風險 |
| **密碼重設 Token** | `RandomNumberGenerator` | 安全性至關重要 |
| **OTP 驗證碼** | `RandomNumberGenerator` | 防止預測攻擊 |
| 數學模擬 | `Random` + 固定種子 | 方便可重現結果 |

---

## 實用工具與技巧

### 隨機數工具類

```csharp
public static class RandomHelper
{
    private static readonly Random _random = new Random();
    private static readonly object _lock = new object();
    
    // 執行緒安全的隨機數
    public static int Next(int minValue, int maxValue)
    {
        lock (_lock)
        {
            return _random.Next(minValue, maxValue);
        }
    }
    
    // 隨機選擇
    public static T Choose<T>(params T[] options)
    {
        if (options == null || options.Length == 0)
            throw new ArgumentException("Options cannot be null or empty");
            
        lock (_lock)
        {
            return options[_random.Next(options.Length)];
        }
    }
    
    // 加權隨機選擇
    public static T WeightedChoice<T>(IList<(T item, int weight)> weightedItems)
    {
        int totalWeight = weightedItems.Sum(x => x.weight);
        
        lock (_lock)
        {
            int randomValue = _random.Next(totalWeight);
            int currentWeight = 0;
            
            foreach (var (item, weight) in weightedItems)
            {
                currentWeight += weight;
                if (randomValue < currentWeight)
                    return item;
            }
            
            return weightedItems.Last().item;
        }
    }
}

// 使用範例
string winner = RandomHelper.Choose("Alice", "Bob", "Charlie");

var items = new List<(string, int)>
{
    ("普通道具", 70),
    ("稀有道具", 25),
    ("傳說道具", 5)
};
string drop = RandomHelper.WeightedChoice(items);
```

### 完整小抄

```csharp
// 📝 【Random 完整小抄】

// 基本用法
var rand1 = new Random(123);    // 固定種子 → 每次結果一樣
var rand2 = new Random();       // 用時間當種子 → 每次結果可能不同

// 常用方法
var dice = rand1.Next(1, 7);           // 1-6 的整數
var percentage = rand1.Next(101);      // 0-100 的整數
var ratio = rand1.NextDouble();        // 0.0-1.0 的浮點數

// 陣列隨機選擇
string[] options = { "A", "B", "C" };
string choice = options[rand1.Next(options.Length)];

// 安全隨機數
using var rng = RandomNumberGenerator.Create();
byte[] bytes = new byte[4];
rng.GetBytes(bytes);
int secureVal = BitConverter.ToInt32(bytes, 0);
```

---

## 總結與最佳實踐

### 🎯 **選擇指南**

```
一般遊戲、模擬 → Random
需要重現結果 → Random(固定種子)
密碼、Token → RandomNumberGenerator
```