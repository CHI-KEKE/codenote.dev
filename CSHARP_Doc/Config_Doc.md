# 🔧 C# Configuration 設定檔完整指南

## 📖 目錄

### 一、基礎設定
1. [套件安裝與環境準備](#一套件安裝與環境準備)
2. [設定檔格式與結構](#二設定檔格式與結構)
3. [基本讀取方式](#三基本讀取方式)

### 二、進階應用
1. [連線字串處理](#一連線字串處理)
2. [型別轉換與取值](#二型別轉換與取值)
3. [區段讀取技巧](#三區段讀取技巧)

### 三、實戰應用
1. [相依性注入整合](#一相依性注入整合)
2. [動態設定繫結](#二動態設定繫結)
3. [設定來源管理](#三設定來源管理)

---

## 🚀 一、基礎設定

### 一、套件安裝與環境準備

#### 📦 必要套件安裝

在開始使用 Configuration 功能前，需要安裝對應的 NuGet 套件：

```powershell
Install-Package Microsoft.Extensions.Configuration.Json
```

#### 📁 套件位置參考

安裝完成後，套件會位於以下路徑：
- `C:\Users\[UserName]\.nuget\packages\microsoft.extensions.configuration\6.0.1\`
- `C:\Users\[UserName]\.nuget\packages\microsoft.extensions.configuration.json\6.0.0\`

### 二、設定檔格式與結構

#### 🔧 基本 appsettings.json 範例

```json
{
  "ConnectionStrings": {
    "DefaultConnectionString": "Server=(localdb)\\mssqllocaldb;Database=EFGetStarted.ConsoleApp.NewDb;Trusted_Connection=True;"
  },
  "Player": {
    "AppId": "testApp",
    "Key": "12345678990"
  }
}
```

#### 🌐 進階設定範例（包含第三方服務）

```json
{
  "Stripe": {
    "ApiUrl": "https://api.stripe.com/v1/payment_methods",
    "": ""
  },
  "ConnectionStrings": {
    "WebStoreDB": ""
  }
}
```

### 三、基本讀取方式

#### 💻 基本讀取設定檔範例

```csharp
[TestMethod]
public void 讀取設定檔()
{
    var builder = new ConfigurationBuilder()
                  .SetBasePath(Directory.GetCurrentDirectory())
                  .AddJsonFile("appsettings.json");
    var config = builder.Build();

    Console.WriteLine($"AppId = {config["AppId"]}");
    Console.WriteLine($"AppId = {config["Player:AppId"]}");
    Console.WriteLine($"Key = {config["Player:Key"]}");
    Console.WriteLine($"Connection String = {config["ConnectionStrings:DefaultConnectionString"]}");
}
```

#### 🔍 重要概念說明

- **IConfigurationBuilder 物件**：用於建立 IConfigurationRoot 物件
- **SetBasePath 方法**：設定檔案的基本路徑
- **AddJsonFile 方法**：讀取設定檔的路徑（基本路徑 + AddJsonFile）
- **存取語法**：使用 `IConfigurationRoot[節點名稱]` 或 `IConfiguration[節點名稱]` 取得設定值
- **錯誤處理**：當區段不存在時，會得到 `null`

---

## 🎯 二、進階應用

### 一、連線字串處理

#### 🔗 專用連線字串讀取方法

```csharp
[TestMethod]
public void 讀取設定檔_GetConnectionString()
{
    var builder = new ConfigurationBuilder()
                  .SetBasePath(Directory.GetCurrentDirectory())
                  .AddJsonFile("appsettings.json");
    var config = builder.Build();
    var connectionString = config.GetConnectionString("DefaultConnection");

    // 整合 Entity Framework 範例
    // var dbContextOptions = new DbContextOptionsBuilder<LabEmployeeContext>()
    //                        .UseSqlServer(connectionString)
    //                        .Options;
}
```

#### 🎯 ASP.NET Core 專案整合範例

**appsettings.json**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=devVibe.db"
  }
}
```

**Program.cs**
```csharp
builder.Services.AddApplicationServices(builder.Configuration);
```

**ApplicationServiceExtensions.cs**
```csharp
namespace API.Extensions
{
    public static class ApplicationServiceExtensions
    {
        public static IServiceCollection AddApplicationServices(this IServiceCollection services, IConfiguration config)
        {
            services.AddDbContext<DataContext>(opt => {
                opt.UseSqlite(config.GetConnectionString("DefaultConnection"));
            });
            
            return services;
        }
    }
}
```

### 二、型別轉換與取值

#### 🔄 GetValue 方法使用

**設定檔範例**
```json
{
   "Authentication": {
      "ClientId": "MyClient",
      "ClientSecret": "MySecret",
      "AllowedHosts": [
         "localhost",
         "localhost:8080"
      ],
      "Attempts": 3
   }
}
```

**Program.cs 中使用**
```csharp
builder.Configuration.GetValue<string>("Authentication:ClientSecret");
```

**Controller 中使用**
```csharp
private readonly IConfiguration _configuration;

public HomeController(IConfiguration configuration)
{
    _configuration = configuration;
}
   
[HttpGet]
public IActionResult Index()
{
    return Ok(new {
        ClientId = _configuration.GetValue<string>("Authentication:ClientId"),
        ClientSecret = _configuration.GetValue<string>("Authentication:ClientSecret"),
        AllowedHosts = _configuration.GetSection("Authentication:AllowedHosts").Get<string[]>(),
        Attempts = _configuration.GetValue<int>("Authentication:Attempts"),
    });
}
```

### 三、區段讀取技巧

#### 📂 GetSection 方法應用

**複雜設定檔範例**
```json
{
   "Logging": {
      "LogLevel": {
         "Default": "Information",
         "Microsoft": "Warning",
         "Microsoft.Hosting.Lifetime": "Information"
      }
   },
   "AllowedHosts": "*",
   "RoundTheCodeSync": {
      "Title": "Our Sync Tool",
      "Interval": "00:30:00",
      "ConcurrentThreads": 10
   }
}
```

**HomeController 讀取範例**
```csharp
// 相依性注入 IConfiguration
private readonly IConfiguration _configuration;

// 讀取區段中的特定值
return Content(_config.GetSection("RoundTheCodeSync")
                     .GetChildren()
                     .FirstOrDefault(config => config.Key == "Title").Value
                + "\r\n" + _configuration.GetValue<string>("RoundTheCodeSync:Title"), 
                "text/plain");
```

---

## 🏆 三、實戰應用

### 一、相依性注入整合

#### 🔧 Singleton 模式設定

**Program.cs 設定**
```csharp
services.AddSingleton<IRoundTheCodeSync>((ServiceProvider) => {
    return Configuration.GetSection("RoundTheCodeSync").Get<RoundTheCodeSync>();
});
```

**介面定義**
```csharp
public interface IRoundTheCodeSync
{
    string Title { get; set; }
    TimeSpan Interval { get; set; }
    int ConcurrentThreads { get; set; }
}
```

**實作類別**
```csharp
public class RoundTheCodeSync : IRoundTheCodeSync
{
    public string Title { get; set; }
    public TimeSpan Interval { get; set; }
    public int ConcurrentThreads { get; set; }
}
```

**依賴注入使用**
```csharp
protected readonly IRoundTheCodeSync _roundTheCodeSync;

public IActionResult GetTitle()
{
    return Content(_roundTheCodeSync.Title);
}
```

### 二、動態設定繫結

#### 🔍 IConfigurationProvider 深度解析

**什麼是 IConfigurationProvider？**

當我們在 ASP.NET Core 或 .NET 應用程式中使用設定時，通常會使用 `IConfiguration` 或 `IConfigurationRoot` 來讀取 `appsettings.json` 或環境變數等資料。但其實背後是由一個個「設定來源」（Provider）組成的。

#### 🎯 TryGet 方法的核心概念

`TryGet(string key, out string value)` 是 `IConfigurationProvider` 的一個方法，用來嘗試從「單一個設定來源」讀取某個設定鍵的值。

**特性說明：**
- 回傳 `bool` 表示是否成功讀取
- 只從自己這個 Provider 查，不會從其他來源查（不像 `IConfiguration` 是綜合所有來源）

#### 📦 比喻說明

把整個設定系統想成一個「設定倉庫」，這個倉庫裡有很多櫃子（不同 Provider，例如 JSON、環境變數、命令列參數等）：

- 使用 `IConfiguration["MyKey"]`：「請給我這個 Key 的值，不管在哪個櫃子都找看看」
- 使用 `IConfigurationProvider.TryGet("MyKey", out var value)`：「我只想問某個櫃子，有沒有這個 Key？」

### 三、設定來源管理

#### 🔧 多重設定來源範例

```csharp
using Microsoft.Extensions.Configuration;
using System;

var builder = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: true)
    .AddEnvironmentVariables();

IConfigurationRoot configRoot = builder.Build();

// 列出所有 Provider（每個代表一種設定來源）
foreach (var provider in configRoot.Providers)
{
    if (provider.TryGet("MyAppSettings:MyKey", out var value))
    {
        Console.WriteLine($"找到設定值：{value}，來源：{provider.GetType().Name}");
    }
}
```

#### 🎯 常見應用情境

這種方式特別適合用於：
- 了解設定值的來源（appsettings.json vs 環境變數）
- 動態設定優先級管理
- 偵錯設定載入問題
- 實作複雜的設定覆蓋邏輯

---