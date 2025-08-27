# HttpClient 請求處理指南

## 📖 目錄

- [1. Post-loop](#1-post-loop)
- [2. jsonPayload](#2-jsonpayload)
- [3. Get-loop](#3-get-loop)
- [4. 組織 QueryString](#4-組織-querystring)
- [5. 設定 Timeout](#5-設定-timeout)
---

## 1. Post-loop

```csharp
async Task Main()
{
    var httpClient = new HttpClient();
    httpClient.DefaultRequestHeaders.Clear();
    httpClient.DefaultRequestHeaders.Add("N1-SHOP-ID", "12820");
    httpClient.DefaultRequestHeaders.Add("ny-market", "MY");
	httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", Convert.ToBase64String(byteArray));
    for (int i = 0; i < 10; i++) // 測試先送 10 次
    {
        var result = await SendRequestAsync(httpClient);
        result.Dump();
        await Task.Delay(10);
    }
}

async Task<string> SendRequestAsync(HttpClient httpClient)
{
    var requestUri = "https://promotion-api-internal.qa.91dev.tw/api/promotion-rules/create";

    var jsonObj = new
    {
        shopId = 12820,
        typeDef = "DiscountReachPriceWithFreeGift",
        name = Guid.NewGuid().ToString(),
        startDateTime = "2024-01-16T20:00:00Z",
        endDateTime = "2024-01-17T23:59:30Z",
        hasPicture = true,
        description = "描述",
        // ...其餘欄位略（與前面相同）
    };

    var json = JsonSerializer.Serialize(jsonObj);
    var content = new StringContent(json, Encoding.UTF8, "application/json");
    var response = await httpClient.PostAsync(requestUri, content);
    return await response.Content.ReadAsStringAsync();
}

```

## 2. jsonPayload

```csharp
// payload
var jsonPayload = @"
{
    ""shopId"": 12820,
    ""typeDef"": ""DiscountReachPriceWithFreeGift"",
    ""name"": ""{{$guid}}"",
    ""startDateTime"": ""2024-01-16T20:00:00Z"",
    ""endDateTime"": ""2024-01-17T23:59:30Z"",
    ""hasPicture"": true,
    ""description"": ""描述"",
    ""customCampaignCode"": """",
    ""targetMemberTypeDef"": ""All"",
    ""targetMemberType"": ""All"",
    ""crmShopMemberCardIds"": [],
    ""isRegular"": false,
    ""promotionTargetRegionTypeDef"": ""All"",
    ""promotionTargetRegionType"": ""All"",
    ""includeTargetRegionIdList"": null,
    ""targetPlatformTypeList"": [
        { ""platform"": ""Web"", ""isEnabled"": true },
        { ""platform"": ""AppAndroid"", ""isEnabled"": true },
        { ""platform"": ""AppIOS"", ""isEnabled"": true },
        { ""platform"": ""LocationWizard"", ""isEnabled"": false }
    ],
    ""visibleMemberTypeDef"": ""All"",
    ""visiblePlatformTypeList"": [
        { ""platform"": ""Web"", ""isEnabled"": true },
        { ""platform"": ""AppAndroid"", ""isEnabled"": true },
        { ""platform"": ""AppIOS"", ""isEnabled"": true },
        { ""platform"": ""LocationWizard"", ""isEnabled"": false }
    ],
    ""targetTypeDef"": ""Shop"",
    ""targetType"": ""Shop"",
    ""excludeTargetTypeDef"": ""None"",
    ""excludeTargetType"": ""None"",
    ""discountScopesType"": null,
    ""groupDiscountRuleList"": null,
    ""discountRuleList"": null,
    ""isCyclable"": false,
    ""accumulated"": false,
    ""isMultiLevel"": true,
    ""canUseECoupon"": true,
    ""calculateTypeDef"": ""Compatible"",
    ""salesDayOfWeek"": 0,
    ""payProfileTypeDef"": ""string"",
    ""promotionEngineBinCodeId"": 0,
    ""registerStartDateTime"": ""2023-03-10T11:50:16.112Z"",
    ""registerEndDateTime"": ""2023-03-10T11:50:16.112Z"",
    ""terms"": null,
    ""agreeContent"": null,
    ""displaySetting"": """",
    ""supplierId"": 2,
    ""userName"": 3,
    ""giftRuleList"": [
        {
            ""reachCondition"": 1,
            ""type"": ""Gift"",
            ""qty"": 1,
            ""collectionId"": """",
            ""isRepeatable"": false,
            ""giftTypeIds"": [0]
        }
    ],
    ""discountRuleList"": [
        { ""condition"": 399, ""discount"": 0.8 }
    ],
    ""code"": ""string"",
    ""includeTargetIdList"": [57264],
    ""excludeTargetIdList"": null,
    ""payTypes"": [],
    ""shippingTypes"": [""Home"", ""Oversea""]
}";
```

## 3. Get-loop

```csharp
async Task Main()
{
    var httpClient = new HttpClient();
    httpClient.DefaultRequestHeaders.Add("N1-SHOP-ID", "12820");
    httpClient.DefaultRequestHeaders.Add("ny-market", "MY");

    for (int i = 0; i < 10; i++) // 可自行調整次數
    {
        var result = await SendGetRequestAsync(httpClient);
        result.Dump();
        await Task.Delay(10);
    }
}

async Task<string> SendGetRequestAsync(HttpClient httpClient)
{
    var requestUri = "https://your-api-url.com/api/some-endpoint"; // ✅ 替換成你的 GET API

    var response = await httpClient.GetAsync(requestUri);
    response.EnsureSuccessStatusCode(); // 如果不是 200 會丟例外

    var content = await response.Content.ReadAsStringAsync();
    return content;
}
```

## 4. 組織 QueryString

```csharp
public static string AppendQueryStringToUrl(string url, IReadOnlyDictionary<string, string> query)
{
    if (string.IsNullOrWhiteSpace(url) || query == null || query.Count == 0)
        return url;

    var uriBuilder = new UriBuilder(url);

    var queryParams = HttpUtility.ParseQueryString(uriBuilder.Query);

    foreach (var kvp in query)
    {
        if (string.IsNullOrWhiteSpace(kvp.Value) == false)
        {
            queryParams[kvp.Key] = kvp.Value;
        }
    }

    // 重建 query，並強制用 EscapeDataString 編碼（避免 +）
    var encodedQuery = string.Join("&", queryParams.AllKeys
        .Select(key => $"{Uri.EscapeDataString(key)}={Uri.EscapeDataString(queryParams[key])}"));

    uriBuilder.Query = encodedQuery;

    return uriBuilder.ToString();
}

## 5. 設定 Timeout

```csharp
builder.Services.AddHttpClient("FastApi", client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.Timeout = TimeSpan.FromSeconds(5); // 設定 5 秒逾時
});
```

| 使用方式                 | 設定 Timeout 的位置                                          |
| -------------------- | ------------------------------------------------------- |
| `new HttpClient()`   | `client.Timeout = TimeSpan.FromSeconds(5);`             |
| `IHttpClientFactory` | `AddHttpClient("name", client => client.Timeout = ...)` |
| 每次建立動態設定             | `factory.CreateClient().Timeout = ...`（較不推薦）            |