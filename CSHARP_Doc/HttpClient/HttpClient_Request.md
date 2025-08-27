# HttpClient 請求處理指南

## 📖 目錄

- [1. Post-loop](#1-Post-loop)
---

## 1. Post-loop

```csharp
async Task Main()
{
    var httpClient = new HttpClient();
    httpClient.DefaultRequestHeaders.Add("N1-SHOP-ID", "12820");
    httpClient.DefaultRequestHeaders.Add("ny-market", "MY");

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