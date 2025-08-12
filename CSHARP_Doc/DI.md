# 💉 C# 依賴注入完整指南

## 📖 目錄
- [一、注入手法](#一注入手法)
  - [1. Constructor Injection 建構式注入](#1-constructor-injection-建構式注入)
    - [1.1 基本概念](#11-基本概念)
    - [1.2 實作範例](#12-實作範例)
    - [1.3 容器註冊](#13-容器註冊)
  - [2. 手動依賴注入](#2-手動依賴注入)
    - [2.1 不使用容器的做法](#21-不使用容器的做法)
    - [2.2 完整範例](#22-完整範例)
    - [2.3 手動注入特性](#23-手動注入特性)
  - [3. 方法注入](#3-方法注入)
    - [3.1 適用時機](#31-適用時機)
    - [3.2 實作範例](#32-實作範例)
    - [3.3 注意事項](#33-注意事項)
  - [4. 屬性注入](#4-屬性注入)
    - [4.1 基本概念](#41-基本概念)
    - [4.2 實作範例](#42-實作範例)
    - [4.3 問題點分析](#43-問題點分析)
  - [5. 注入物件時的預處理](#5-注入物件時的預處理)
    - [5.1 基本委派設定](#51-基本委派設定)
    - [5.2 取得其他服務](#52-取得其他服務)
    - [5.3 實際應用場景](#53-實際應用場景)
  - [6. 動態服務解析 (Autofac)](#6-動態服務解析-autofac)
    - [6.1 Named Registration 機制](#61-named-registration-機制)
    - [6.2 動態解析實作](#62-動態解析實作)
    - [6.3 應用優勢](#63-應用優勢)
  - [7. IServiceProvider 注入問題與解法](#7-iserviceprovider-注入問題與解法)
    - [7.1 問題案例](#71-問題案例)
    - [7.2 解法 A：Keyed Services (.NET 8+)](#72-解法-a-keyed-services-net-8)
    - [7.3 解法 B：策略模式 + 字典](#73-解法-b-策略模式--字典)
    - [7.4 解法 C：Factory 介面封裝](#74-解法-c-factory-介面封裝)
  - [8. 循環依賴問題與解法](#8-循環依賴問題與解法)
    - [8.1 問題說明](#81-問題說明)
    - [8.2 產生原因](#82-產生原因)
    - [8.3 解決方法：抽出更高階類別](#83-解決方法抽出更高階類別)
    - [8.4 實作範例](#84-實作範例)
- [二、概念](#二概念)
  - [1. 組合根（Composition Root）](#1-組合根composition-root)
    - [1.1 核心職責](#11-核心職責)
    - [1.2 最佳實踐](#12-最佳實踐)
    - [1.3 常見問題](#13-常見問題)
    - [1.4 組合根分類管理](#14-組合根分類管理)
- [三、生命週期](#三生命週期)
  - [1. Singleton 使用時機](#1-singleton-使用時機)
    - [1.1 適用場景](#11-適用場景)
    - [1.2 範例說明](#12-範例說明)
  - [2. Scoped 使用時機](#2-scoped-使用時機)
    - [2.1 適用場景](#21-適用場景)
    - [2.2 範例說明](#22-範例說明)
  - [3. 依賴關係週期對齊](#3-依賴關係週期對齊)
    - [3.1 問題分析](#31-問題分析)
    - [3.2 實際案例](#32-實際案例)
    - [3.3 合法範例](#33-合法範例)
  - [4. BuildServiceProvider 問題](#4-buildserviceprovider-問題)
    - [4.1 核心概念](#41-核心概念)
    - [4.2 問題分析](#42-問題分析)
    - [4.3 正確做法](#43-正確做法)

---

## 一、注入手法

### 1. Constructor Injection 建構式注入

#### 1.1 基本概念

🎯 **核心優勢**

Constructor Injection 能保證物件初始化後就具備所有必要依賴，不會用到時才出現「尚未注入就使用」的錯誤。

#### 1.2 實作範例

```csharp
public class Car
{
    private readonly IEngine _engine;

    public Car(IEngine engine)
    {
        _engine = engine;
    }

    public void Start()
    {
        _engine.Run();
    }
}
```

#### 1.3 容器註冊

```csharp
services.AddTransient<IEngine, V8Engine>();
services.AddTransient<Car>();
```

### 2. 手動依賴注入

#### 2.1 不使用容器的做法

📋 **適用場景**

當不想使用 DI 容器，或需要精細控制物件建立過程時，可以採用手動依賴注入。

#### 2.2 完整範例

```csharp
public interface IEngine
{
    void Run();
}

public class V8Engine : IEngine
{
    public void Run()
    {
        Console.WriteLine("V8 engine is running!");
    }
}

public class Car
{
    private readonly IEngine _engine;

    public Car(IEngine engine)
    {
        _engine = engine;
    }

    public void Drive()
    {
        _engine.Run();
        Console.WriteLine("Car is driving!");
    }
}

class Program
{
    static void Main()
    {
        // 手動建立依賴
        IEngine engine = new V8Engine();

        // 手動將依賴注入給使用者
        Car car = new Car(engine);

        // 使用物件
        car.Drive();
    }
}
```

#### 2.3 手動注入特性

**優點：**
- 完全控制物件建立過程
- 不依賴外部容器
- 適合簡單場景

**缺點：**
- 需要手動管理依賴關係
- 程式碼較為冗長
- 難以應付複雜的依賴網路

### 3. 方法注入

#### 3.1 適用時機

⚡ **使用場景**

方法注入適用於以下情況：
- 呼叫方法時需要注入不同的依賴對象
- 該方法在不同地方被呼叫的依賴對象不一樣
- 第一次呼叫和第二次呼叫時的依賴對象不一樣

#### 3.2 實作範例

```csharp
public class Wizard
{
    // 施放指定的法術卷軸
    public void Enchant(ISpellScroll scroll)
    {
        scroll.CastSpell(); // 詠唱
    }
}

// 法術卷軸介面
public interface ISpellScroll
{
    void CastSpell();
}
```

##### 📋 設計說明

1. **我們有一個法師類別**
2. **他會施法**
3. **施法需要由呼叫端傳入卷軸類型**

#### 3.3 注意事項

⚠️ **重要提醒**

由於該方法的呼叫端需要準備依賴對象給方法當作參數使用，整個過程是在方法被呼叫的時候才動態處理的，所以呼叫端還是需要想辦法弄到該依賴對象，也就是再從上一層注入或是工廠製造之類的。

**維護考量：**
- 會增加類別或介面之間的依賴關係
- 讓注入的位置散落在各地
- 維護時需要多注意

**適用場合：**
- 一些輔助工具，例如擴充方法或是 Helper
- 單純操作邏輯的場合

🚨 **絕對不要的做法**

絕對不要把方法注入的依賴對象留在物件內部給其他方法使用，例如 A 方法注入了某個物件，保留給之後呼叫 B 方法的時候用。

**問題分析：**
- 容易造成時序耦合（Temporal Coupling）的問題
- 使用者不照想好的順序呼叫，方法就會直接失敗
- 違反了封裝精神
- 容易產生預期外的副作用

✅ **建議解決方案**

如果真的有需要針對依賴對象做初始化，還是考慮用建構式注入。

### 4. 屬性注入

#### 4.1 基本概念

📋 **核心特性**

屬性注入是從公開的屬性丟進依賴對象，讓呼叫端決定「要不要」依賴的場合。

#### 4.2 實作範例

```csharp
void Main()
{
    var wizard = new Wizard();
    wizard.Spell = new GiantCollosolCoolball();
}

public class Wizard
{
    private ISpell _spell;
    public ISpell Spell
    {
        get
        {
            if (this._spell is null)
            {
                this._spell = new Fireball();
            }

            return this._spell;
        }
        set
        {
            this._spell = value;
        }
    }
}

public interface ISpell { }
public class Fireball : ISpell{}
public class GiantCollosolCoolball : ISpell{}
```

##### 🎯 設計考量

**預設值策略：**
- 由於使用者並不會知道物件內部的狀態，屬性注入沒有提供預設值就很容易壞掉
- 給預設值時又不可避免地產生耦合（例如法師為了預設是火球術，所以和火球術產生了耦合）

**適用時機：**
- 預設值使用內建的、通常會存在的類別，但允許外部隨時替換
- 讓呼叫端決定「要不要」依賴的場合

**重要提醒：**
「屬性注入」的時機比「建構式注入」來得晚，而且外界不一定會設定該屬性。如欲採用「屬性注入」，類別本身通常要能夠取得相依物件的預設實作。

#### 4.3 問題點分析

##### 1️⃣ 可能為 Null，導致執行時錯誤

🧠 **問題說明**

屬性注入不像建構子注入那樣「保證一定有值」，如果忘了設定，程式執行時可能會 Null Reference 錯誤。

💥 **範例**

```csharp
public ILogger Logger { get; set; }

public void DoSomething()
{
    Logger.Log("開始執行任務"); // 如果 Logger 是 null，這裡就會拋出例外
}
```

✅ **解法**
- 用 Null-conditional 運算子 (`Logger?.Log(...)`)
- 或搭配 Null Object Pattern（設一個預設的 Logger）

##### 2️⃣ 依賴關係不清楚、不顯性

🧠 **問題說明**

屬性注入讓依賴項「藏起來」，不像建構子注入那樣一眼就知道一個類別依賴什麼。

📌 **範例**

```csharp
public class MyService
{
    public ICacheService Cache { get; set; }
    public ILogger Logger { get; set; }
}
```

你根本不知道這兩個有沒有被設定進來，也不知道哪些是必要的。

✅ **解法**
- 嚴格定義哪些屬性是「必填」，哪些是「選配」
- 搭配文件或注釋說明

##### 3️⃣ DI 容器無法強制注入該屬性

🧠 **問題說明**

大多數 DI 容器（像 ASP.NET Core 的 IServiceProvider）預設不支援自動屬性注入，除非你自己加上額外邏輯。

✅ **解法**
- 改用建構子注入
- 或使用支援屬性注入的容器（如 Autofac）
- 或在註冊後手動設定屬性

##### 4️⃣ 單元測試容易漏掉設定屬性

🧠 **問題說明**

測試時很容易忘了設定那些注入的屬性，導致 Null 錯誤，增加維護成本。

✅ **解法**
- 寫測試時要明確設定屬性（Mock 或 Fake）
- 或在建構子中提供預設值，減少出錯

##### 6️⃣ 生命週期管理容易出錯

🧠 **問題說明**

若類別本身是 Singleton，但透過屬性注入注入了一個 Scoped 的依賴，會造成生命週期衝突，甚至記憶體洩漏。

✅ **解法**
- 避免在 Singleton 中用屬性注入 Scoped/Transient
- 建議用建構子注入 + 明確的生命週期管理

##### 7️⃣ 順序依賴問題

🧠 **問題說明**

如果有人在注入屬性前就呼叫了方法，可能會出錯。

📌 **範例**

```csharp
var service = new MyService();
service.DoSomething(); // Logger 還沒設，就執行了
```

✅ **解法**
- 建立初始化方法（如 `Init()`）
- 在方法中檢查是否已初始化

##### 🎯 總結建議

**主要問題：**
- 今天沒有一個統一管理的地方，可能每個使用者都自己注入不同的依賴服務，比較能預測行為
- 建構的時候就帶著可能使用上比較安全一點

**最佳實踐：**
- 優先考慮建構式注入
- 屬性注入僅用於可選的依賴
- 提供適當的預設值和錯誤處理

### 5. 注入物件時的預處理

#### 5.1 基本委派設定

🔧 **簡單預處理**

當我們需要在注入物件時進行一些預處理，可以使用委派的方式：

```csharp
services.AddScoped<ITestService>(sp =>
{
    var token = @"TestServiceToken";
    return new TestService(token);
});
```

#### 5.2 取得其他服務

🔗 **整合其他依賴**

藉由 ServiceProvider 的 `GetRequiredService` 方法來取得其他註冊的實體，並利用這個實體來完成注入所需的材料：

```csharp
// 委派的參數是 ServiceProvider
services.AddScoped<ITestService>(sp =>
{
    var tokenService = sp.GetRequiredService<ITokenService>();
    var token = tokenService.Get();
    return new TestService(token);
});
```

#### 5.3 實際應用場景

📋 **常見使用情境**

- **設定初始化參數**：需要特定設定值或環境變數
- **複雜建構邏輯**：需要多個依賴協同建立物件
- **條件性建立**：根據不同條件建立不同實現
- **第三方函式庫整合**：需要特殊初始化步驟的外部函式庫

### 6. 動態服務解析 (Autofac)

#### 6.1 Named Registration 機制

🏷️ **具名註冊範例**

Autofac 的 Named Registration 允許我們為同一個介面註冊多個不同的實作，並透過名稱來區分：

```csharp
// PaymentServiceProvider 註冊
builder.RegisterType<AtomeService>().Named<IDirectPaymentService>("Atome");
builder.RegisterType<PXPayPlusService>().Named<IDirectPaymentService>("PXPayPlus");
builder.RegisterType<PlusPayService>().Named<IDirectPaymentService>("PlusPay");
builder.RegisterType<OpenWalletService>().Named<IDirectPaymentService>(nameof(PayProfileTypeDefEnum.OpenWallet));
builder.RegisterType<FamilyMartOnlinePayService>().Named<IDirectPaymentService>(nameof(PayProfileTypeDefEnum.FamilyMartOnlinePay));
builder.RegisterType<RazerMerchantServicesService>().Named<IDirectPaymentService>(nameof(PayProfileTypeDefEnum.RazerPay));
builder.RegisterType<LinePayDirectService>().Named<IDirectPaymentService>(nameof(PayProfileTypeDefEnum.LinePay));
builder.RegisterType<StoreCreditPaymentService>()
    .Named<IDirectPaymentService>(nameof(PayProfileTypeDefEnum.StoreCredit))
    .As<IStoreCreditCancelPaymentService>()
    .AsSelf();

// Provider 層級註冊
builder.RegisterType<DirectPaymentProvider>().Named<IPaymentServiceProviderBase>("Direct");
builder.RegisterType<PaymentMiddlewareProvider>().Named<IPaymentServiceProviderBase>("PaymentMiddleware");
builder.RegisterType<Nine1PaymentProvider>().Named<IPaymentServiceProviderBase>(nameof(PaymentServiceProviderTypeEnum.Nine1Payment));
```

##### 🔧 **PayChannelService 解析案例**

以下是另一個實際應用案例，展示如何透過 Named Registration 來解析支付通道服務：

```csharp
// 業務邏輯中的使用方式
var (payChannelService, paymentPath) = this.ResolvePayChannelService(context);

payChannelService = _payChannelServiceResolver.Resolve(acquiring);

/// <summary>
/// Resolve <see cref="IPayChannelService"/>
/// </summary>
/// <param name="payChannel">PayChannel</param>
/// <returns><see cref="IPayChannelService"/></returns>
public IPayChannelService Resolve(string payChannel)
{
    return this._lifetimeScope.ResolveNamed<IPayChannelService>(payChannel);
}
```

#### 6.2 動態解析實作

⚡ **GetService 方法實作**

透過 `ResolveNamed` 方法，我們可以根據執行時的條件動態選擇對應的服務：

```csharp
/// <summary>
/// 取得 PSP Service
/// </summary>
/// <param name="shopId">商店序號</param>
/// <param name="payProfileType">付款方式</param>
/// <returns>PSP Service</returns>
public IPaymentServiceProviderBase GetService(long shopId, string payProfileType)
{
    // 取得 PSP 設定值
    var paymentProviderSetting = this.GetPaymentProviderSetting(shopId, payProfileType);

    if (Enum.TryParse<PaymentServiceProviderTypeEnum>(paymentProviderSetting, out PaymentServiceProviderTypeEnum providerType) == false)
    {
        throw new NotSupportedException("PaymentServiceProviderTypeEnum");
    }
    
    IPaymentServiceProviderBase targetService = this._lifetimeScope.ResolveNamed<IPaymentServiceProviderBase>(providerType.ToString());

    return targetService;
}
```

##### 🔄 **基於具體類型的動態解析**

除了 Autofac 的 Named Registration，我們也可以使用 ASP.NET Core 原生 DI 容器搭配具體類型來實現動態服務解析：

```csharp
// Program.cs 中註冊具體實作服務
builder.Services.AddScoped<RewardReachPriceWithCouponRuleService>();

// RewardReachPriceWithCouponRuleService 實際上有實作 Interface
public class RewardReachPriceWithCouponRuleService : RewardReachPriceRuleBaseService, IPromotionEngineRuleService
{
    // 實作內容
}

// 動態解析方法
private IPromotionEngineRuleService GetRuleService(string typeDef) => typeDef switch
{
    nameof(PromotionEngineTypeDefEnum.RewardReachPriceWithCoupon) => 
        this._serviceProvider.GetRequiredService<RewardReachPriceWithCouponRuleService>(),
    // 其他類型的對應...
    _ => throw new NotSupportedException($"不支援的促銷規則類型: {typeDef}")
};

// 應用端使用方式
// 反序列化 Rule
var ruleService = this.GetRuleService(promotion.PromotionEngine_TypeDef);
var ruleEntity = ruleService.ParsePromotionEngineRuleObject(promotion.PromotionEngine_Rule);
```

**此方法的特點：**
- **原生支援**：使用 ASP.NET Core 內建 DI 容器，無需額外相依性
- **類型安全**：透過具體類型註冊，享有編譯時期類型檢查
- **Pattern Matching**：利用 C# switch expression 提供清晰的對應邏輯
- **介面統一**：所有服務都實作相同介面，確保API一致性

#### 6.3 應用優勢

🎯 **程式碼清潔度**

**避免複雜的 switch/if 判斷：**
- 傳統做法需要大量條件判斷來決定使用哪個付款服務
- Named Registration 將選擇邏輯委託給容器處理
- 程式碼更加簡潔且易於維護

**增強可維護性：**
- 新增付款方式只需註冊新的 Named service
- 不需要修改現有的條件判斷邏輯
- 符合開放封閉原則（Open/Closed Principle）

**實際業務場景：**
- **付款系統**：根據不同付款方式動態選擇對應的處理器
- **多租戶架構**：根據租戶 ID 選擇不同的服務實作
- **通知系統**：根據通知類型選擇 Email、SMS 或推播服務
- **資料來源切換**：根據環境或設定動態選擇不同的資料提供者

**設計模式整合：**
- 結合策略模式（Strategy Pattern）實現動態行為選擇
- 工廠模式（Factory Pattern）的現代化實作
- 依賴反轉原則（Dependency Inversion Principle）的實際應用

##### 📚 **進階參考資料**

關於動態解析實作的更多技術細節和進階用法，可以參考：

🔗 [如何在 Microsoft.Extensions.DependencyInjection 中為同一介面註冊多個實作](https://dotblogs.azurewebsites.net/yc421206/2021/05/21/how_to_register_the_same_interface_for_multiple_implement_microsoft_extensions_dependencyInjection)

### 7. IServiceProvider 注入問題與解法

#### 7.1 問題案例

⚠️ **生命週期衝突場景**

有時候我們會遇到這樣的情況：Validator 是 Singleton 類型的，但我們需要注入 Scoped 類型的服務，無法直接在建構子上相依。這時候往往會想到注入 `IServiceProvider` 來解決，但這其實不是最佳做法。

**常見問題模式：**
```csharp
// ❌ 不推薦的做法
public class MySingletonValidator
{
    private readonly IServiceProvider _serviceProvider;
    
    public MySingletonValidator(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider; // 直接依賴容器
    }
    
    public void Validate(string type)
    {
        var service = _serviceProvider.GetRequiredService<IScopedService>(); // Service Locator 反模式
    }
}
```

#### 7.2 解法 A：Keyed Services (.NET 8+)

🎯 **最推薦的現代化解法**

用 DI 直接幫你管理「Key → 服務」的對照表，你的類別只依賴「抽象」與「key」，不需要拿容器。

##### 🔧 服務註冊

```csharp
// 針對不同規則，用 key 註冊對應的實作
services.AddKeyedScoped<IPromotionEngineRuleService, DiscountReachGroupsPieceRuleService>(
    nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece));

services.AddKeyedScoped<IPromotionEngineRuleService, CommonRuleService>("__common");
```

##### 📋 使用方式 A-1：直接用 IKeyedServiceProvider

```csharp
public class PromotionRuleResolver(IKeyedServiceProvider keyed)
{
    private const string CommonKey = "__common";

    private IPromotionEngineRuleService GetRuleService(string typeDef) => typeDef switch
    {
        nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece)
            => keyed.GetRequiredKeyedService<IPromotionEngineRuleService>(
                   nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece)),
        _   => keyed.GetRequiredKeyedService<IPromotionEngineRuleService>(CommonKey)
    };
}
```

**優點：** 依賴的是專用的 keyed provider，範圍很小，不是萬能的 IServiceProvider。

##### 🏭 使用方式 A-2：用「工廠委派」讓依賴更明確

```csharp
public class PromotionRuleResolver(Func<object, IPromotionEngineRuleService> getByKey)
{
    private const string CommonKey = "__common";

    private IPromotionEngineRuleService GetRuleService(string typeDef) => typeDef switch
    {
        nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece)
            => getByKey(nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece)),
        _   => getByKey(CommonKey)
    };
}
```

**優點：** 你的類別只依賴一個「取服務的函式」，更容易替身（mock）測試。

#### 7.3 解法 B：策略模式 + 字典

🗂️ **任何版本可用的經典解法**

每個規則實作自己宣告「我是哪個 typeDef」，在組裝時把它們做成字典。

##### 🎯 介面與實作設計

```csharp
public interface IPromotionEngineRuleService
{
    string TypeDef { get; }  // 例如 "DiscountReachGroupsPiece"
    Task ApplyAsync(...);
}

public class DiscountReachGroupsPieceRuleService : IPromotionEngineRuleService
{
    public string TypeDef => nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece);
    public Task ApplyAsync(...) { /* ... */ }
}

public class CommonRuleService : IPromotionEngineRuleService
{
    public string TypeDef => "__common";
    public Task ApplyAsync(...) { /* ... */ }
}
```

##### 🔧 組裝與使用

```csharp
public class PromotionRuleResolver
{
    private readonly IReadOnlyDictionary<string, IPromotionEngineRuleService> _map;
    private readonly IPromotionEngineRuleService _common;

    public PromotionRuleResolver(IEnumerable<IPromotionEngineRuleService> services)
    {
        _map = services.ToDictionary(s => s.TypeDef, StringComparer.Ordinal);
        _common = services.Single(s => s.TypeDef == "__common");
    }

    private IPromotionEngineRuleService GetRuleService(string typeDef)
        => _map.TryGetValue(typeDef, out var svc) ? svc : _common;
}
```

**優點：** 
- 零 Service Locator
- 依賴全顯性
- 超好測試

**💡 小技巧：** 如果建立服務很重，可把字典值改成 `Lazy<IPromotionEngineRuleService>`。

#### 7.4 解法 C：Factory 介面封裝

🏭 **漸進式改善方案**

把「選服務」的邏輯搬到專屬工廠，之後要從 `_serviceProvider` 換到 A/B 的寫法，只改工廠就好。

##### 🎯 Factory 介面設計

```csharp
public interface IPromotionRuleFactory
{
    IPromotionEngineRuleService Create(string typeDef);
}

public class PromotionRuleFactory : IPromotionRuleFactory
{
    private readonly IKeyedServiceProvider _keyed; // 或 Func<object, IPromotionEngineRuleService>
    
    public PromotionRuleFactory(IKeyedServiceProvider keyed) => _keyed = keyed;

    public IPromotionEngineRuleService Create(string typeDef) => typeDef switch
    {
        nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece)
            => _keyed.GetRequiredKeyedService<IPromotionEngineRuleService>(
                   nameof(PromotionEngineTypeDefEnum.DiscountReachGroupsPiece)),
        _ => _keyed.GetRequiredKeyedService<IPromotionEngineRuleService>("__common")
    };
}
```

##### 📋 使用端程式碼

```csharp
// 使用端
public class PromotionRuleResolver(IPromotionRuleFactory factory)
{
    private IPromotionEngineRuleService GetRuleService(string typeDef) => factory.Create(typeDef);
}
```

**優點：**
- 小改就好，風險較低
- 封裝容器依賴，未來容易替換
- 依賴更加明確和可測試

### 8. 循環依賴問題與解法

#### 8.1 問題說明

🔄 **雞生蛋、蛋生雞的困境**

循環依賴是指 Class A 需要 Class B 才能工作，而 Class B 又需要 Class A 才能工作。DI 容器在建立物件時，會卡在「先要 A 還是先要 B？」這種雞生蛋、蛋生雞的情況，結果就會拋出例外。

```csharp
// ❌ 循環依賴範例 - 這會導致 DI 容器失敗
Class A ← depends on → Class B
    ↑                     ↓
    └─── depends on ←─────┘
```

**典型錯誤訊息：**
- `A circular dependency was detected`
- `Cannot resolve service for type 'ClassA' because it has a circular dependency`

#### 8.2 產生原因

🧠 **設計問題根源**

循環依賴通常反映了以下設計問題：

**責任切不清：**
- 兩個類別互相管彼此太多事
- 職責邊界模糊，導致過度耦合
- 每個類別都想要控制或了解對方的行為

**流程被拆得太碎：**
- 拆分過度細粒度，失去了整體性
- 彼此都要知道對方的細節才能運作
- 缺乏統一的協調機制

#### 8.3 解決方法：抽出更高階類別

🏗️ **Orchestrator 模式**

解決循環依賴的核心思想是引入一個更高階的類別（通常稱為 Orchestrator 或 Coordinator），它的角色是：

**統一協調：**
- 不讓 A 和 B 直接知道對方的存在
- 把「A 做什麼 → B 做什麼」這個流程集中管理
- A 和 B 都只對這個新類別報告或由新類別呼叫

**依賴重新組織：**
- A 不再依賴 B
- B 也不再依賴 A  
- 兩者改成依賴同一個「更高階」類別
- 循環依賴自然解除

#### 8.4 實作範例

##### ❌ **壞的範例：循環依賴**

```csharp
public class ClassA
{
    private readonly ClassB _b;
    
    public ClassA(ClassB b) => _b = b;

    public void DoA()
    {
        Console.WriteLine("A 做了事情");
        _b.DoB(); // A 依賴 B
    }
}

public class ClassB
{
    private readonly ClassA _a;
    
    public ClassB(ClassA a) => _a = a;

    public void DoB()
    {
        Console.WriteLine("B 做了事情");
        _a.DoA(); // B 又依賴 A -> 循環依賴！
    }
}
```

**問題分析：**
- ClassA 建構子需要 ClassB
- ClassB 建構子需要 ClassA
- DI 容器無法決定先建立哪一個

##### ✅ **好的範例：抽出更高階類別**

```csharp
public class Orchestrator
{
    private readonly ClassA _a;
    private readonly ClassB _b;

    public Orchestrator(ClassA a, ClassB b)
    {
        _a = a;
        _b = b;
    }

    public void Run()
    {
        _a.DoA();
        _b.DoB();
        // 統一管理 A 和 B 的協作流程
    }
}

public class ClassA
{
    public void DoA()
    {
        Console.WriteLine("A 做了事情");
        // 不再直接呼叫 B
    }
}

public class ClassB
{
    public void DoB()
    {
        Console.WriteLine("B 做了事情");
        // 不再直接呼叫 A
    }
}
```

**改善效果：**
- ✅ ClassA 和 ClassB 不再互相依賴
- ✅ 職責清楚：A 專心做 A 的事，B 專心做 B 的事
- ✅ Orchestrator 負責協調整體流程
- ✅ DI 容器可以順利建立所有物件

**DI 註冊：**
```csharp
services.AddScoped<ClassA>();
services.AddScoped<ClassB>();
services.AddScoped<Orchestrator>();
```

**使用方式：**
```csharp
public class SomeController
{
    private readonly Orchestrator _orchestrator;
    
    public SomeController(Orchestrator orchestrator)
    {
        _orchestrator = orchestrator;
    }
    
    public void SomeAction()
    {
        _orchestrator.Run(); // 透過 Orchestrator 協調 A 和 B
    }
}
```

##### 🎯 **設計原則總結**

**避免循環依賴的關鍵：**
- **單一職責**：每個類別專注於自己的核心功能
- **依賴方向**：依賴應該是單向的，從具體到抽象
- **協調分離**：將協調邏輯抽出到專門的協調者
- **介面隔離**：使用介面減少直接類別依賴

---

## 二、概念

### 1. 組合根（Composition Root）

#### 1.1 核心職責

🏗️ **統一管理依賴關係**

由於組合根必須分配各個類別前往自己負責的崗位，因此它可以說是和所有模組都有直接依賴的關係（並且也應該只有組合根可以知道整體的物件關聯）。

#### 1.2 最佳實踐

📍 **統一設定位置**

我們應該要在應用程式啟動的部份，找個風水寶地去統一管理我們的組合根和依賴關係，而不能讓注入的部分散亂在各地。

**現代 DI 容器解決方案：**
- Unity 的 `UnityConfig`
- .NET Core 的 `ConfigureServices`
- 其他框架的對應設定機制

#### 1.3 常見問題

⚠️ **避免散亂注入**

只要注意別在其他地方偷偷搞注入、挖坑給別人跳就好。

📌 **實際案例警告**

註：不要以為真的遇不到……同事維護的專案就有遇到前同事直接在單元測試案例裡把 DI 容器叫出來註冊依賴注入的，都不知道從哪裡開始吐槽囧

**問題分析：**
- 如果在別的地方去亂對依賴關係動手動腳，很可能就會踩到一些坑
- 總之，盡量別在組合根以外的地方使用 DI 容器

#### 1.4 組合根分類管理

🗂️ **模組化組織**

當專案規模變大時，可以將不同類型的服務註冊分類管理，避免 `ConfigureServices` 變得過於冗長且難以維護。

##### 📋 分類註冊範例

**Service 類別的擴充方法：**

```csharp
// ProjectN.DIExtensions

/// <summary>
/// Service 相關註冊
/// </summary>
public static class ServiceDIExtensions
{
    public static IServiceCollection AddServices(this IServiceCollection services)
    {
        services.AddScoped<ICardService, CardService>();

        return services;
    }
}
```

**主要啟動設定：**

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddServices(); // 呼叫 ServiceDIExtensions 進行註冊

    services.AddRepositories();

    services.AddControllers();
}
```

##### 🎯 分類管理優勢

**組織結構清晰：**
- 按功能領域分組（Services、Repositories、Infrastructure）
- 每個擴充類別專責特定類型的註冊
- 主要設定檔保持簡潔易讀

**維護性提升：**
- 新增服務時只需修改對應的擴充類別
- 減少 merge conflict 的機會
- 便於團隊分工協作

**可讀性增強：**
- 一目了然各個模組的註冊狀況
- 便於追蹤和除錯依賴關係
- 符合單一職責原則

---

## 三、生命週期

### 1. Singleton 使用時機

#### 1.1 適用場景

🔄 **資源共用考量**

對 `HttpClient` 這類能共用同個實例節省資源的，我們就可以考慮使用 Singleton。

#### 1.2 範例說明

**適合 Singleton 的服務：**
- HTTP 客戶端
- 快取服務
- 設定服務
- 日誌服務

### 2. Scoped 使用時機

#### 2.1 適用場景

🎯 **請求範圍內共用**

一般來說最常用的會是 Scoped，例如功能服務或登入者資訊，在同一次呼叫中保持同一個即可。

#### 2.2 範例說明

**適合 Scoped 的服務：**
- 業務邏輯服務
- 資料庫上下文
- 使用者身份資訊
- 請求特定的狀態管理

### 3. 依賴關係週期對齊

#### 3.1 問題分析

🚨 **生命週期衝突**

Singleton 不應該依賴 Scoped 或 Transient，否則可能會造成非預期的錯誤。

#### 3.2 實際案例

💥 **問題情境描述**

想像你有一個 Singleton 類別 `MyService`，在應用程式啟動時建立，永遠都存在記憶體中。如果它依賴一個 Scoped 類別 `MyDependency`，後者只會在每次 HTTP Request 被建立一次，請想想下面這個情境：

**步驟分解：**
1. **第一次使用 Singleton 時**，它會被注入一個來自某個 HTTP Request 的 Scoped 實例
2. **該 Request 完成後**，該 Scoped 實例應該會被釋放（Disposed）。但 Singleton 還持有對該物件的參考
3. **當下一次呼叫 Singleton** 嘗試使用已被釋放的物件時，就會產生：
   - `ObjectDisposedException`
   - 記憶體洩漏
   - 跨 Request 資料污染

📌 **核心問題**

換句話說：Singleton 是所有 Request 共用的，但 Scoped 僅存在於單一 Request，因此不能保證 Scoped 在 Singleton 使用它時還是有效的。

#### 3.3 合法範例

✅ **正確的生命週期組合**

合法範例（生命週期一致或越長）：
- `Scoped` 依賴 `Transient` ✅（Request 中建立多個）
- `Singleton` 依賴 `Singleton` ✅（全域共用）

### 4. BuildServiceProvider 問題

#### 4.1 核心概念

📋 **重要名詞說明**

| 名詞                       | 解釋                                                                       |
| ------------------------ | ------------------------------------------------------------------------ |
| `ServiceProvider`        | 是 ASP.NET Core 內部用來管理「誰要注入誰」的容器。就像一間公司的人事系統，知道哪個部門（Service）需要什麼人員（相依物件）。 |
| `BuildServiceProvider()` | 是一個方法，會「自己建立一個新的 ServiceProvider」。就像是你在原公司外面又偷偷開了一間人事公司。                 |
| `Singleton`              | 表示這個服務在應用程式整個生命週期中只會建立一次的實例。                                             |

#### 4.2 問題分析

🚨 **雙重容器問題**

當你用 `BuildServiceProvider()` 自己偷偷蓋一個新的 DI 容器時，你就會出現 **兩個人事系統（ServiceProvider）同時存在** 的狀況：

✅ **原本 ASP.NET Core 框架幫你建立、全站共用的 ServiceProvider**（預設）

❌ **你手動 BuildServiceProvider() 建立出來的另一個 ServiceProvider**（額外）

##### 💥 實際問題場景

🎯 **假設你有一個 Singleton 服務 MySingleton**

- 在原本的 ServiceProvider 中，它只會被建立一次（符合 Singleton 的精神）
- 但你手動用 `BuildServiceProvider()`，就會在「新的容器」中再建立一次 **完全不同的 Singleton 實例**

這樣就違反了 Singleton 的初衷（應該要全站唯一一份），也會導致：

| 問題場景 | 結果 |
|---------|------|
| **寫入快取** | 你以為寫入了 Singleton 中的記憶體，但實際上寫的是另一份 |
| **共用設定** | 設定在原本 Singleton 裡的參數，另一個 ServiceProvider 看不到 |
| **記錄狀態** | 多個 ServiceProvider 各自管理自己的生命週期與物件狀態，會出現同步錯亂 |

#### 4.3 正確做法

✅ **委派注入現有 ServiceProvider**

在需要使用 ServiceProvider 的地方，應該用委派（delegate）注入現有的 IServiceProvider，不要自己呼叫 `BuildServiceProvider()`。

```csharp
services.AddScoped<IMyService>(provider =>
{
    var dep = provider.GetRequiredService<IDependency>();
    return new MyService(dep);
});
```

##### 🎯 最佳實踐原則

1. **絕對不要手動呼叫 BuildServiceProvider()**
2. **使用框架提供的 IServiceProvider**
3. **讓 DI 容器自動管理物件生命週期**
4. **在組合根統一設定所有依賴關係**