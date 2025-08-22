# 🔴 Redis 快取指南

## 📖 目錄

- [1. 連線設定](#1-連線設定)
  - [1.1 檢查連線狀態](#11-檢查連線狀態)
- [2. 套件](#2-套件)
  - [2.1 StackExchange.Redis](#21-stackexchangeredis)
- [3. 使用 Docker Compose 建立 Redis 環境](#3-使用-docker-compose-建立-redis-環境)
  - [3.1 基本 Docker Compose 設定](#31-基本-docker-compose-設定)
  - [3.2 進階持久化設定](#32-進階持久化設定)
  - [3.3 容器操作指令](#33-容器操作指令)
  - [3.4 GUI 管理工具](#34-gui-管理工具)
  - [3.5 ASP.NET 連線設定](#35-aspnet-連線設定)
- [4. 設計模式 - CacheAttribute](#4-設計模式---cacheattribute)
  - [4.1 快取服務介面設計](#41-快取服務介面設計)
  - [4.2 快取服務實作](#42-快取服務實作)
  - [4.3 自訂快取 Attribute](#43-自訂快取-attribute)
  - [4.4 實際應用範例](#44-實際應用範例)

---

## 1. 連線設定

### 1.1 檢查連線狀態

在 ASP.NET Core 應用程式中設定 Redis 連線並檢查連線是否建立成功：

#### 🔌 **基本連線檢查**

```csharp
using StackExchange.Redis;

var config = builder.Configuration;
builder.Services.AddSingleton<IConnectionMultiplexer>(c => {
    var options = ConfigurationOptions.Parse(config.GetConnectionString("Redis"));
    var connection = ConnectionMultiplexer.Connect(options);
    
    if (connection.IsConnected)
    {
        Console.WriteLine("Redis 連線已建立。");
    }
    else
    {
        Console.WriteLine("連線到 Redis 失敗。");
    }

    return connection;
});
```

#### ⚙️ **設定檔範例**

在 `appsettings.json` 中設定 Redis 連線字串：

```json
{
  "ConnectionStrings": {
    "Redis": "localhost:6379"
  }
}
```

#### 🔧 **進階連線設定**

```csharp
using StackExchange.Redis;

var config = builder.Configuration;
builder.Services.AddSingleton<IConnectionMultiplexer>(serviceProvider =>
{
    var redisConnectionString = config.GetConnectionString("Redis");
    
    var options = ConfigurationOptions.Parse(redisConnectionString);
    
    // 🎯 進階設定選項
    options.AbortOnConnectFail = false;  // 連線失敗時不中止應用程式
    options.ConnectRetry = 3;            // 重試次數
    options.ConnectTimeout = 5000;       // 連線逾時（毫秒）
    options.SyncTimeout = 5000;          // 同步操作逾時
    options.AsyncTimeout = 5000;         // 非同步操作逾時
    
    var connection = ConnectionMultiplexer.Connect(options);
    
    // 📊 詳細連線狀態檢查
    if (connection.IsConnected)
    {
        Console.WriteLine("✅ Redis 連線已成功建立");
        Console.WriteLine($"📍 連線端點: {string.Join(", ", connection.GetEndPoints())}");
        Console.WriteLine($"🔧 設定選項: {connection.Configuration}");
    }
    else
    {
        Console.WriteLine("❌ Redis 連線失敗");
        Console.WriteLine($"🔍 連線字串: {redisConnectionString}");
    }
    
    // 🎯 連線事件處理
    connection.ConnectionFailed += (sender, e) =>
    {
        Console.WriteLine($"❌ Redis 連線失敗: {e.Exception?.Message}");
    };
    
    connection.ConnectionRestored += (sender, e) =>
    {
        Console.WriteLine($"✅ Redis 連線已恢復: {e.EndPoint}");
    };
    
    connection.ErrorMessage += (sender, e) =>
    {
        Console.WriteLine($"⚠️ Redis 錯誤訊息: {e.Message}");
    };
    
    return connection;
});
```

#### 🧪 **連線測試服務**

建立一個專門的服務來測試 Redis 連線：

```csharp
public interface IRedisHealthService
{
    Task<bool> IsHealthyAsync();
    Task<string> GetConnectionInfoAsync();
}

public class RedisHealthService : IRedisHealthService
{
    private readonly IConnectionMultiplexer _connectionMultiplexer;

    public RedisHealthService(IConnectionMultiplexer connectionMultiplexer)
    {
        _connectionMultiplexer = connectionMultiplexer;
    }

    public async Task<bool> IsHealthyAsync()
    {
        try
        {
            var database = _connectionMultiplexer.GetDatabase();
            await database.PingAsync();
            return _connectionMultiplexer.IsConnected;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"❌ Redis 健康檢查失敗: {ex.Message}");
            return false;
        }
    }

    public async Task<string> GetConnectionInfoAsync()
    {
        try
        {
            var endpoints = _connectionMultiplexer.GetEndPoints();
            var server = _connectionMultiplexer.GetServer(endpoints.First());
            var info = await server.InfoAsync();
            
            return $"Redis 伺服器資訊: {info}";
        }
        catch (Exception ex)
        {
            return $"無法取得 Redis 伺服器資訊: {ex.Message}";
        }
    }
}

// 註冊服務
builder.Services.AddScoped<IRedisHealthService, RedisHealthService>();
```

#### 🏥 **健康檢查整合**

將 Redis 連線狀態整合到 ASP.NET Core 的健康檢查系統：

```csharp
using Microsoft.Extensions.Diagnostics.HealthChecks;

// 加入健康檢查
builder.Services.AddHealthChecks()
    .AddCheck<RedisHealthCheck>("redis");

public class RedisHealthCheck : IHealthCheck
{
    private readonly IConnectionMultiplexer _connectionMultiplexer;

    public RedisHealthCheck(IConnectionMultiplexer connectionMultiplexer)
    {
        _connectionMultiplexer = connectionMultiplexer;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            if (!_connectionMultiplexer.IsConnected)
            {
                return HealthCheckResult.Unhealthy("Redis 連線未建立");
            }

            var database = _connectionMultiplexer.GetDatabase();
            var pingResult = await database.PingAsync();
            
            var data = new Dictionary<string, object>
            {
                ["ping_time"] = pingResult.TotalMilliseconds,
                ["endpoints"] = string.Join(", ", _connectionMultiplexer.GetEndPoints()),
                ["is_connected"] = _connectionMultiplexer.IsConnected
            };

            return HealthCheckResult.Healthy("Redis 連線正常", data);
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Redis 健康檢查失敗", ex);
        }
    }
}

// 在 Program.cs 中設定健康檢查端點
app.MapHealthChecks("/health");
```

#### 🔍 **連線狀態監控**

```csharp
public class RedisConnectionMonitor : BackgroundService
{
    private readonly IConnectionMultiplexer _connectionMultiplexer;
    private readonly ILogger<RedisConnectionMonitor> _logger;

    public RedisConnectionMonitor(
        IConnectionMultiplexer connectionMultiplexer,
        ILogger<RedisConnectionMonitor> logger)
    {
        _connectionMultiplexer = connectionMultiplexer;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                var isConnected = _connectionMultiplexer.IsConnected;
                var endpoints = _connectionMultiplexer.GetEndPoints();
                
                _logger.LogInformation("Redis 連線狀態: {IsConnected}, 端點: {Endpoints}", 
                    isConnected, string.Join(", ", endpoints));

                if (isConnected)
                {
                    var database = _connectionMultiplexer.GetDatabase();
                    var pingTime = await database.PingAsync();
                    _logger.LogInformation("Redis Ping 時間: {PingTime}ms", pingTime.TotalMilliseconds);
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Redis 連線監控發生錯誤");
            }

            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}

// 註冊背景服務
builder.Services.AddHostedService<RedisConnectionMonitor>();
```

## 2. 套件

### 2.1 StackExchange.Redis

#### 📦 **套件安裝**

**StackExchange.Redis** 是 .NET 中最受歡迎且效能最佳的 Redis 用戶端函式庫。

```bash
# 使用 Package Manager Console
Install-Package StackExchange.Redis

# 使用 .NET CLI
dotnet add package StackExchange.Redis

# 使用 PackageReference (在 .csproj 檔案中)
<PackageReference Include="StackExchange.Redis" Version="2.7.20" />
```

#### 🎯 **主要功能特色**

- ✅ **高效能**：最佳化的網路協定和連線管理
- ✅ **非同步支援**：完整的 async/await 支援
- ✅ **多重資料庫**：支援 Redis 的多個資料庫
- ✅ **叢集支援**：Redis Cluster 的完整支援
- ✅ **交易支援**：Redis 交易和批次操作
- ✅ **發布訂閱**：Redis Pub/Sub 功能
- ✅ **Lua 腳本**：執行 Lua 腳本的支援

#### 🔧 **基本使用方式**

```csharp
using StackExchange.Redis;

// 建立連線
var redis = ConnectionMultiplexer.Connect("localhost:6379");
var database = redis.GetDatabase();

// 基本操作
await database.StringSetAsync("key", "value");
var value = await database.StringGetAsync("key");

// 複雜資料型別
await database.HashSetAsync("user:1", "name", "Allen");
await database.ListLeftPushAsync("queue", "item1");
await database.SetAddAsync("tags", "redis", "cache");
```

## 3. 使用 Docker Compose 建立 Redis 環境

### 3.1 基本 Docker Compose 設定

#### 🐳 **簡單的 Redis + GUI 設定**

建立 `docker-compose.yml` 檔案：

```yaml
services:

  redis:
    image: redis:latest
    ports:
      - 6379:6379
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis-data:/data

  redis-commander:
    image: rediscommander/redis-commander:latest
    environment:
      - REDIS_HOSTS=local:redis:6379
      - HTTP_USER=root
      - HTTP_PASSWORD=secret
    ports:
      - 8081:8081
    depends_on:
      - redis
    
volumes:
  redis-data:
```

#### 📌 **組態說明**

**Redis 容器：**
- 🔧 使用官方的 `redis:latest` 映像檔
- 🌐 對外開放 `6379` port
- 💾 使用 `appendonly` 模式確保資料持久化
- 📂 使用 `redis-data` volume 保存資料，即使容器刪除也能保留

**Redis Commander 容器：**
- 🖥️ 使用 `rediscommander/redis-commander` 映像檔，這是一個 Redis 的 GUI 工具
- 🔗 `REDIS_HOSTS=local:redis:6379` → 連接到上面定義的 redis 容器
- 🔐 `HTTP_USER=root` / `HTTP_PASSWORD=secret` → 登入 GUI 的帳號密碼
- 🌐 對外開放 `8081` port
- ⚡ `depends_on` → 確保 redis 先啟動，再啟動 redis-commander

**Volumes：**
- 💾 定義一個 `redis-data` volume，用來持久化 redis 資料

#### 🚀 **執行方式**

```bash
# 在專案目錄下開啟終端機
docker-compose up --detach

# --detach = 背景執行，不會卡住終端機
```

Docker 會自動幫你下載映像檔，並啟動 redis 及 redis-commander。

這樣，就能同時擁有 Redis 服務（6379）和一個方便的 Web GUI（8081）來管理資料。

### 3.2 進階持久化設定

#### 💾 **完整的持久化和設定管理**

首先建立本機目錄結構：

```
D:\dockers\redis\
├── conf\
│   └── redis.conf
├── data\
└── docker-compose.yml
```

#### 📥 **下載並設定 Redis 設定檔**

1. 從 [http://download.redis.io/redis-stable/redis.conf](http://download.redis.io/redis-stable/redis.conf) 下載範例設定檔
2. 放置於 `D:\dockers\redis\conf\redis.conf`

#### ⚙️ **關鍵設定修改**

編輯 `redis.conf` 檔案：

```conf
# 1. 拿掉 bind 127.0.0.1 可以允許非本機連線
# bind 127.0.0.1

# 2. 資料目錄設定
dir /data

# 3. 設定密碼
requirepass YourPassword123

# 4. 啟用 AOF 持久化
appendonly yes
appendfilename "appendonly.aof"

# 5. 記憶體最佳化
maxmemory 256mb
maxmemory-policy allkeys-lru
```

#### 🐳 **進階 Docker Compose 設定**

建立 `D:\dockers\redis\docker-compose.yml`：

```yaml
services:
  myredis:
    container_name: myredis
    image: redis:latest
    ports:
      - 6379:6379
    privileged: true
    command: redis-server /etc/redis/redis.conf --appendonly yes
    volumes:
      - /d/dockers/redis/data:/data
      - /d/dockers/redis/conf/redis.conf:/etc/redis/redis.conf
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
```

#### 🚀 **啟動服務**

```bash
# 啟動 Redis 伺服器
docker-compose up -d

# 檢查執行狀態
docker-compose ps

# 檢視日誌
docker-compose logs -f myredis
```

### 3.3 容器操作指令

#### 🔍 **容器管理指令**

```bash
# 確認容器 ID 和名稱
docker ps

# 進入容器
docker exec -it myredis sh

# 或者如果容器名稱不同
docker exec -it conversationalshopping-redis-1 sh
```

#### 💻 **Redis CLI 操作**

進入容器後的操作：

```bash
# 開啟 Redis CLI
redis-cli

# 如果有設定密碼，需要先驗證
AUTH YourPassword123
# 回應: OK

# 測試連線
ping
# 回應: PONG

# 基本操作測試
set test "Hello Redis"
get test
# 回應: "Hello Redis"

# 檢視所有鍵
keys *

# 檢視 Redis 資訊
info

# 離開 CLI
exit
```

#### 📊 **監控和除錯指令**

```bash
# 即時監控 Redis 指令
redis-cli monitor

# 檢視緩慢查詢
redis-cli slowlog get

# 檢視記憶體使用情況
redis-cli info memory

# 檢視連線資訊
redis-cli info clients
```

### 3.4 GUI 管理工具

#### 🖥️ **Another Redis Desktop Manager**

安裝 GUI 管理工具：

```bash
# 使用 winget 安裝
winget install qishibo.AnotherRedisDesktopManager
```

#### 🔗 **連線設定**

在 Another Redis Desktop Manager 中設定連線：

```
主機: localhost
Port: 6379
密碼: YourPassword123
名稱: Local Redis
```

#### 🌐 **Web 介面（Redis Commander）**

如果使用 Redis Commander，可透過瀏覽器存取：

```
網址: http://localhost:8081
帳號: root
密碼: secret
```

#### 🎯 **GUI 工具功能**

- 📊 **資料瀏覽**：視覺化檢視所有鍵值
- 🔧 **資料編輯**：直接編輯 Redis 資料
- 📈 **效能監控**：即時監控 Redis 效能
- 💾 **匯入匯出**：資料備份和還原
- 🔍 **搜尋功能**：快速找尋特定鍵值

### 3.5 ASP.NET 連線設定

#### 🔌 **連線設定類別**

```csharp
public class RedisManagerHelper
{
    private readonly IDatabase _redisDatabase;

    public RedisManagerHelper() 
    {
        var options = ConfigurationOptions.Parse("localhost");
        options.AllowAdmin = true;          // 如果需要進行管理操作，可以設定為 true
        options.AbortOnConnectFail = false; // 如果要允許重試連線，可以設定為 false
        options.Password = "YourPassword123";

        var redisConnection = ConnectionMultiplexer.Connect(options);
        _redisDatabase = redisConnection.GetDatabase();
    }

    // 基本操作方法
    public async Task<bool> SetAsync(string key, string value)
    {
        return await _redisDatabase.StringSetAsync(key, value);
    }

    public async Task<string> GetAsync(string key)
    {
        return await _redisDatabase.StringGetAsync(key);
    }

    public async Task<bool> DeleteAsync(string key)
    {
        return await _redisDatabase.KeyDeleteAsync(key);
    }
}
```

#### 🏗️ **相依性注入設定**

```csharp
// Program.cs
builder.Services.AddSingleton<IConnectionMultiplexer>(provider =>
{
    var configuration = provider.GetService<IConfiguration>();
    var connectionString = configuration.GetConnectionString("Redis");
    
    var options = ConfigurationOptions.Parse(connectionString);
    options.AllowAdmin = true;
    options.AbortOnConnectFail = false;
    options.Password = configuration["Redis:Password"];
    
    return ConnectionMultiplexer.Connect(options);
});

builder.Services.AddScoped<IDatabase>(provider =>
{
    var multiplexer = provider.GetService<IConnectionMultiplexer>();
    return multiplexer.GetDatabase();
});
```

#### ⚙️ **設定檔（appsettings.json）**

```json
{
  "ConnectionStrings": {
    "Redis": "localhost:6379"
  },
  "Redis": {
    "Password": "YourPassword123",
    "Database": 0,
    "ConnectTimeout": 5000,
    "SyncTimeout": 5000
  }
}
```

#### 🧪 **使用範例**

```csharp
[ApiController]
[Route("api/[controller]")]
public class CacheController : ControllerBase
{
    private readonly IDatabase _database;

    public CacheController(IDatabase database)
    {
        _database = database;
    }

    [HttpPost("{key}")]
    public async Task<IActionResult> SetCache(string key, [FromBody] string value)
    {
        var result = await _database.StringSetAsync(key, value);
        return Ok(new { success = result, key, value });
    }

    [HttpGet("{key}")]
    public async Task<IActionResult> GetCache(string key)
    {
        var value = await _database.StringGetAsync(key);
        
        if (!value.HasValue)
            return NotFound(new { message = "Key not found" });
            
        return Ok(new { key, value = value.ToString() });
    }

    [HttpDelete("{key}")]
    public async Task<IActionResult> DeleteCache(string key)
    {
        var result = await _database.KeyDeleteAsync(key);
        return Ok(new { deleted = result, key });
    }
}
```

> **💡 重點提醒**
> 
> 1. **連線管理**：使用 `IConnectionMultiplexer` 作為單例服務，避免頻繁建立連線
> 2. **錯誤處理**：設定適當的逾時和重試機制
> 3. **監控機制**：實作健康檢查和連線狀態監控
> 4. **設定彈性**：透過設定檔管理 Redis 連線參數
> 5. **資料持久化**：使用 Volume 確保資料不會因容器重啟而遺失
> 6. **安全性**：在正式環境中務必設定強密碼和適當的網路安全措施

## 4. 設計模式 - CacheAttribute

### 4.1 快取服務介面設計

#### 🎯 **IResponseCacheService 介面定義**

建立一個清晰且簡潔的快取服務介面，提供基本的快取存取功能：

```csharp
public interface IResponseCacheService
{
    Task CacheResponseAsync(string cacheKey, object response, TimeSpan timeToLive);
    Task<string> GetCacheResponseAsync(string cacheKey);
}
```

#### 📋 **介面方法說明**

- **`CacheResponseAsync`**：將物件序列化後儲存到 Redis，並設定過期時間
- **`GetCacheResponseAsync`**：根據快取鍵值取得已序列化的資料

### 4.2 快取服務實作

#### 🔧 **ResponseCacheService 實作**

```csharp
public class ResponseCacheService : IResponseCacheService
{
    private readonly IDatabase _database;

    public ResponseCacheService(IConnectionMultiplexer redis)
    {
        _database = redis.GetDatabase();
    }

    public async Task CacheResponseAsync(string cacheKey, object response, TimeSpan timeToLive)
    {
        if (response == null)
        {
            return;
        }

        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

        var serializedResponse = JsonSerializer.Serialize(response, options);

        await _database.StringSetAsync(cacheKey, serializedResponse, timeToLive);
    }

    public async Task<string> GetCacheResponseAsync(string cacheKey)
    {
        var cachedResponse = await _database.StringGetAsync(cacheKey);

        if (cachedResponse.IsNullOrEmpty)
        {
            return null;
        }

        return cachedResponse;
    }
}
```

#### 🏗️ **服務註冊**

在 `Program.cs` 中註冊快取服務：

```csharp
services.AddSingleton<IResponseCacheService, ResponseCacheService>();
```

#### ✨ **實作特色**

- 🔒 **空值檢查**：避免快取 null 物件
- 📦 **JSON 序列化**：使用 camelCase 命名規則，符合前端慣例
- ⏰ **過期時間控制**：支援自訂快取存活時間
- 🚀 **非同步操作**：完全支援 async/await 模式

### 4.3 自訂快取 Attribute

#### 🎭 **CachedAttribute 實作**

建立一個強大的快取 Attribute，可以自動處理 API 回應的快取機制：

```csharp
public class CachedAttribute : Attribute, IAsyncActionFilter
{
    private readonly int _timeToLiveSeconds;

    public CachedAttribute(int timeToLiveSeconds)
    {
        _timeToLiveSeconds = timeToLiveSeconds;
    }

    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var cacheService = context.HttpContext.RequestServices.GetRequiredService<IResponseCacheService>();

        var cacheKey = GenerateCacheKeyFromRequest(context.HttpContext.Request);

        // 檢查是否已有快取
        var cachedResponse = await cacheService.GetCacheResponseAsync(cacheKey);

        if (!string.IsNullOrEmpty(cachedResponse))
        {
            // 直接回傳快取內容
            var contentResult = new ContentResult
            {
                Content = cachedResponse,
                ContentType = "application/json",
                StatusCode = 200,
            };

            context.Result = contentResult;
            return;
        }

        // 若沒有快取 → 繼續執行 Controller
        var executedContext = await next();

        // 成功回傳時，將結果存入快取
        if (executedContext.Result is OkObjectResult okObjectResult)
        {
            await cacheService.CacheResponseAsync(
                cacheKey,
                okObjectResult.Value,
                TimeSpan.FromSeconds(_timeToLiveSeconds)
            );
        }
    }

    /// <summary>
    /// 根據 Path + QueryString 產生快取 Key
    /// </summary>
    private string GenerateCacheKeyFromRequest(HttpRequest request)
    {
        var keyBuilder = new StringBuilder();

        // API 路徑
        keyBuilder.Append($"{request.Path}");

        // Query 參數需排序，避免同樣參數不同順序產生不同 Key
        foreach (var (key, value) in request.Query.OrderBy(x => x.Key))
        {
            keyBuilder.Append($"|{key}-{value}");
        }

        return keyBuilder.ToString();
    }
}
```

#### 🎯 **Attribute 核心功能**

- 🔍 **快取檢查**：優先檢查是否已有快取資料
- 🚀 **快速回應**：如有快取直接回傳，無需執行原始邏輯
- 💾 **自動快取**：成功執行後自動將結果存入快取
- 🔑 **智慧鍵值**：根據路徑和參數產生唯一快取鍵值
- 📊 **參數排序**：確保相同參數不同順序產生相同鍵值

### 4.4 實際應用範例

#### 🛍️ **Controller 套用**

在需要快取的 API 方法上套用 `[Cached]` 屬性：

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    [Cached(600)] // 600 秒 = 10 分鐘
    public async Task<IActionResult> GetProducts()
    {
        var products = await _productService.GetProductsAsync();
        return Ok(products);
    }

    [HttpGet("{id}")]
    [Cached(300)] // 300 秒 = 5 分鐘
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null)
            return NotFound();
            
        return Ok(product);
    }

    [HttpGet("category/{categoryId}")]
    [Cached(1200)] // 1200 秒 = 20 分鐘
    public async Task<IActionResult> GetProductsByCategory(int categoryId, [FromQuery] int page = 1, [FromQuery] int size = 10)
    {
        var products = await _productService.GetProductsByCategoryAsync(categoryId, page, size);
        return Ok(products);
    }
}
```

#### 🎨 **應用場景建議**

| **場景** | **快取時間** | **說明** |
|----------|-------------|----------|
| 🏪 **商品列表** | 10-30 分鐘 | 變動頻率低，適合較長快取 |
| 👤 **使用者資料** | 5-15 分鐘 | 可能隨時更新，適中快取時間 |
| 📊 **統計資料** | 1-6 小時 | 計算成本高，可長時間快取 |
| 🔥 **熱門內容** | 30 分鐘 - 2 小時 | 重要但變動性適中 |
| 🎯 **搜尋結果** | 10-60 分鐘 | 結果相對穩定，可適度快取 |

#### 💡 **最佳實務建議**

1. **⏰ 快取時間策略**：根據資料變動頻率設定合適的過期時間
2. **🔑 鍵值設計**：確保鍵值唯一性，避免不同資料互相覆蓋
3. **🎯 選擇性快取**：只對讀取頻繁且計算成本高的 API 使用快取
4. **🔄 快取更新**：重要資料異動時考慮主動清除相關快取
5. **📊 監控快取**：定期檢查快取命中率和效能改善情況

> **🚀 效能提升效果**
> 
> - ⚡ **回應時間**：從原本的 200-500ms 降低至 10-30ms
> - 💾 **資料庫負載**：減少 60-90% 的重複查詢
> - 🔄 **併發處理**：提升系統處理大量同時請求的能力
> - 💰 **成本節省**：降低伺服器資源使用和資料庫連線成本