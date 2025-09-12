# Autofac 相依性注入指南

## 目錄
- [1. Autofac.Integration.Mvc 使用](#1-autofacintegrationmvc-使用)
  - [1.1 Autofac.Integration.Mvc 套件的本質](#11-autofacintegrationmvc-套件的本質)
  - [1.2 Application_Start](#12-application_start)
- [2. Validator 是靜態的不要用建構子注入!!](#2-validator-是靜態的不要用建構子注入)
- [3. 找 Domain](#3-找-domain)

---

## 1. Autofac.Integration.Mvc 使用

### 1.1 Autofac.Integration.Mvc 套件的本質

- **把 Autofac 的 DI 能力整合到 ASP.NET MVC 專案中**
- **讓 MVC Controller、Filter、Action 之類的元件都能被注入（inject）服務**

### 🔧 **核心功能**

| 功能 | 說明 |
|------|------|
| **Controller 注入** | 自動解析 Controller 的建構子相依性 |
| **Filter 注入** | 支援 ActionFilter、AuthorizationFilter 等的相依性注入 |
| **生命週期管理** | 提供 InstancePerRequest 等 MVC 特有的生命週期 |
| **Web 類型支援** | 內建支援 HttpContext、HttpRequest 等 Web 相關類型 |

### 1.2 Application_Start

```csharp
AutofacConfig.Initialize();
```

表示我要在專案一啟動時，設定好 Autofac 的 DI 容器（Container）

**那 `AutofacConfig.Initialize` 又做了什麼？**

### 🏗️ **完整的 AutofacConfig.Initialize 實作**

```csharp
public static class AutofacConfig
{
    public static void Initialize()
    {
        // 建立 DI 容器建構器（開始建一個裝各種零件（服務）的工廠）
        var builder = new ContainerBuilder();

        // 註冊你自己的服務類別（例如資料庫連線）
        // 我要註冊一個類別叫 TrackContext, 每次 HTTP 請求來，都建立一個新的物件給它用（InstancePerRequest）
        builder.RegisterType<TrackContext>().AsSelf().InstancePerRequest();

        // 註冊所有 Controller
        builder.RegisterControllers(typeof(MvcApplication).Assembly);

        // 加入 MVC 必備的支援（像 HttpContext）
        /*
        Autofac 提供的輔助模組，讓你可以注入：
        - HttpContextBase
        - HttpRequestBase
        - HttpResponseBase
        等等這些 MVC 常用的類別
        */
        builder.RegisterModule<AutofacWebTypesModule>();

        // 其他自訂元件
        builder.RegisterType<DetectOfficialAttribute>().AsSelf();

        // 建立 DI 容器
        var container = builder.Build();

        // 告訴 MVC：從現在開始都用 Autofac 來找服務
        DependencyResolver.SetResolver(new AutofacDependencyResolver(container));
    }
}
```

### 📋 **設定步驟詳解**

**步驟 1：建立容器建構器**
```csharp
var builder = new ContainerBuilder();
```
- 🏭 建立一個「服務工廠」的藍圖
- 📝 準備註冊各種服務和元件

**步驟 2：註冊自訂服務**
```csharp
builder.RegisterType<TrackContext>().AsSelf().InstancePerRequest();
```
- 📊 註冊資料庫上下文
- 🔄 每個 HTTP 請求建立一個新實例
- ♻️ 請求結束時自動釋放資源

**步驟 3：註冊所有 Controller**
```csharp
builder.RegisterControllers(typeof(MvcApplication).Assembly);
```
- 🎯 自動掃描並註冊所有 MVC Controller
- 🔧 讓 Controller 支援建構子注入

**步驟 4：加入 Web 類型支援**
```csharp
builder.RegisterModule<AutofacWebTypesModule>();
```
- 🌐 註冊 Web 相關的內建類型
- 💡 讓你可以在建構子中注入 `HttpContextBase` 等

**步驟 5：註冊自訂屬性**
```csharp
builder.RegisterType<DetectOfficialAttribute>().AsSelf();
```
- 🏷️ 註冊自訂的 Action Filter
- 🔌 讓 Filter 也能使用相依性注入

**步驟 6：建立容器並設定解析器**
```csharp
var container = builder.Build();
DependencyResolver.SetResolver(new AutofacDependencyResolver(container));
```
- 🏗️ 建立最終的 DI 容器
- 🔀 替換 MVC 預設的相依性解析器

### 🎯 **實際使用範例**

**Controller 中的相依性注入：**
```csharp
public class HomeController : Controller
{
    private readonly TrackContext _trackContext;
    private readonly IEmailService _emailService;
    
    public HomeController(TrackContext trackContext, IEmailService emailService)
    {
        _trackContext = trackContext;
        _emailService = emailService;
    }
    
    public ActionResult Index()
    {
        // 使用注入的服務
        var data = _trackContext.GetTrackingData();
        return View(data);
    }
}
```

**Action Filter 中的相依性注入：**
```csharp
public class DetectOfficialAttribute : ActionFilterAttribute
{
    private readonly IUserService _userService;
    
    public DetectOfficialAttribute(IUserService userService)
    {
        _userService = userService;
    }
    
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        // 使用注入的服務
        var isOfficial = _userService.IsOfficialUser();
        // 執行相關邏輯
    }
}
```

---

## 2. Validator 是靜態的不要用建構子注入!!

### ⚠️ **重要警告：會撈到舊資料**

```csharp
// ❌ 錯誤做法 - 不要在 Validator 中使用建構子注入
public class UserValidator : AbstractValidator<User>
{
    private readonly IUserRepository _userRepository;
    
    // ❌ 這樣做會有問題！
    public UserValidator(IUserRepository userRepository)
    {
        _userRepository = userRepository; // 可能撈到舊的快取資料
        
        RuleFor(x => x.Email)
            .Must(BeUniqueEmail)
            .WithMessage("Email 已存在");
    }
    
    private bool BeUniqueEmail(string email)
    {
        return !_userRepository.EmailExists(email); // 可能是舊資料！
    }
}
```

### 🚨 **問題原因**

1. **Validator 通常是單例模式**：建構子只會執行一次
2. **資料可能被快取**：第一次查詢的結果可能被保存
3. **後續驗證使用舊資料**：無法取得最新的資料庫狀態
4. **競爭條件**：多執行緒環境下可能出現資料不一致

### ✅ **正確做法**

**方法一：使用 Service Locator Pattern**
```csharp
public class UserValidator : AbstractValidator<User>
{
    public UserValidator()
    {
        RuleFor(x => x.Email)
            .Must(BeUniqueEmail)
            .WithMessage("Email 已存在");
    }
    
    private bool BeUniqueEmail(string email)
    {
        // ✅ 每次驗證時都重新取得服務
        var userRepository = DependencyResolver.Current.GetService<IUserRepository>();
        return !userRepository.EmailExists(email);
    }
}
```

**方法二：使用 FluentValidation 的內建支援**
```csharp
public class UserValidator : AbstractValidator<User>
{
    public UserValidator()
    {
        RuleFor(x => x.Email)
            .MustAsync(async (email, cancellation) =>
            {
                // ✅ 使用非同步方式，每次都重新查詢
                var userRepository = DependencyResolver.Current.GetService<IUserRepository>();
                return !await userRepository.EmailExistsAsync(email);
            })
            .WithMessage("Email 已存在");
    }
}
```

**方法三：將驗證邏輯移到 Service 層**
```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<ValidationResult> ValidateUserAsync(User user)
    {
        var result = new ValidationResult();
        
        // ✅ 在 Service 層進行即時驗證
        if (await _userRepository.EmailExistsAsync(user.Email))
        {
            result.Errors.Add(new ValidationFailure("Email", "Email 已存在"));
        }
        
        return result;
    }
}
```

---

## 3. 找 Domain

### 📁 **檔案位置**
```
C:\91APP\nineyi.scm.apiv2\CrossLayer\Modules\BL\ServiceModule.cs
```

### 🔧 **Domain 設定範例**

```csharp
builder.RegisterType<TaggingServiceHttpClient>()
    .As<ITaggingServiceHttpClient>()
    .WithParameter((info, _) => info.Name == "httpClient", 
                  (_, ctx) => ctx.Resolve<IHttpClientFactoryService>()
                              .Get(new Uri(config.GetAppSetting("TagService.Domain"))));
```

### 📋 **程式碼詳解**

**註冊類型：**
```csharp
builder.RegisterType<TaggingServiceHttpClient>()
```
- 🏷️ 註冊 `TaggingServiceHttpClient` 實作類別

**指定介面：**
```csharp
.As<ITaggingServiceHttpClient>()
```
- 🔌 當需要 `ITaggingServiceHttpClient` 時，提供 `TaggingServiceHttpClient` 實例

**參數注入：**
```csharp
.WithParameter((info, _) => info.Name == "httpClient", 
               (_, ctx) => ctx.Resolve<IHttpClientFactoryService>()
                           .Get(new Uri(config.GetAppSetting("TagService.Domain"))))
```

### 🎯 **參數注入詳解**

**條件判斷：**
```csharp
(info, _) => info.Name == "httpClient"
```
- 🎯 當建構子參數名稱是 "httpClient" 時才套用

**值提供：**
```csharp
(_, ctx) => ctx.Resolve<IHttpClientFactoryService>()
            .Get(new Uri(config.GetAppSetting("TagService.Domain")))
```
- 🏭 從容器中解析 `IHttpClientFactoryService`
- 🌐 從設定檔讀取 "TagService.Domain" 
- 📡 建立指向該 Domain 的 HttpClient

### 🔄 **完整流程**

1. **讀取設定**：從 config 取得 TagService 的 Domain 位址
2. **建立 URI**：將 Domain 字串轉為 Uri 物件
3. **取得 HttpClient**：透過 Factory 建立對應的 HttpClient
4. **注入參數**：將 HttpClient 注入到 TaggingServiceHttpClient 的建構子

### 💡 **設定檔範例**

```xml
<!-- web.config 或 app.config -->
<appSettings>
  <add key="TagService.Domain" value="https://tag-service.example.com" />
</appSettings>
```

### 🏗️ **建構子範例**

```csharp
public class TaggingServiceHttpClient : ITaggingServiceHttpClient
{
    private readonly HttpClient _httpClient;
    
    public TaggingServiceHttpClient(HttpClient httpClient)
    {
        _httpClient = httpClient; // 這個 httpClient 就是透過 WithParameter 注入的
    }
    
    public async Task<TagResponse> GetTagsAsync()
    {
        var response = await _httpClient.GetAsync("/api/tags");
        // 處理回應...
    }
}
```

> **🌟 重點提醒**
> 
> - **生命週期管理**：使用 `InstancePerRequest` 確保每個請求都有獨立的實例
> - **Validator 陷阱**：避免在靜態 Validator 中使用建構子注入
> - **參數注入**：使用 `WithParameter` 進行複雜的參數配置
> - **設定檔管理**：將 Domain 等設定外部化到 config 檔案中