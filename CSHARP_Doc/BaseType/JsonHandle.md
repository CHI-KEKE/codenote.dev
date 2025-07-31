# 📄 C# JSON 處理完整指南

## 📖 目錄
- [一、JSON 字串反序列化](#一json-字串反序列化)
  - [1. 範例程式碼](#1-範例程式碼)
- [二、JToken JObject 操作](#二jtoken-jobject-操作)
  - [1. 基本概念](#1-基本概念)
    - [1.1 JToken 介紹](#11-jtoken-介紹)
    - [1.2 JObject 介紹](#12-jobject-介紹)
  - [2. 實際應用範例](#2-實際應用範例)
    - [2.1 Loki 查詢資料提取](#21-loki-查詢資料提取)
    - [2.2 程式碼解析](#22-程式碼解析)
- [三、外部 API 指定反序列化屬性](#三外部-api-指定反序列化屬性)
  - [1. System.Text.Json 反序列化](#1-systemtextjson-反序列化)
    - [1.1 範例程式碼](#11-範例程式碼)
    - [1.2 實體類別定義](#12-實體類別定義)
    - [1.3 重要注意事項](#13-重要注意事項)
---

## 一、JSON 字串反序列化

#### 1. 範例程式碼

```csharp
void Main()
{
    string cc = "{\"PaymentServiceProvider\":\"TapPay\",\"AppVer\": null,\"Acquiring\":\"808\"}";
    
    var result = JsonConvert.DeserializeObject<PaymentServiceProviderProfileEntity>(cc);
    result.Dump();
}
```

## 二、JToken JObject 操作

### 1. 基本概念

#### 1.1 JToken 介紹

📋 **JToken 說明**

JToken 是 Newtonsoft.Json 套件中所有 JSON 節點的基類，它可以表示物件、陣列、字串、數字等各種 JSON 類型。

🔍 **特點**
- 通用的 JSON 節點基類
- 可以代表各種不同的 JSON 結構
- 提供靈活的 JSON 資料存取方式

#### 1.2 JObject 介紹

📋 **JObject 說明**

JObject 是 JToken 的子類別，專門用來處理 JSON 物件（即鍵值對結構）。

🎯 **主要功能**
- 專門處理 JSON 物件中的屬性
- 提供鍵值對的存取介面
- 支援動態屬性查詢

### 2. 實際應用範例

#### 2.1 Loki 查詢資料提取

```csharp
void Main()
{
    var result = LokiQueryExample.QueryLokiAndExtractData();
    foreach (var data in result)
    {
        Console.WriteLine(data);
    }
}

public class LokiQueryExample
{
    private static readonly string LogFilePath = @"C:\91APP\logg.txt";
    
    public static List<string> QueryLokiAndExtractData()
    {
        var result = new List<string>();

        try
        {
            var logData = File.ReadAllText(LogFilePath);
            var jsonData = JToken.Parse(logData);

            foreach (var item in jsonData)
            {
                if (item["_msg"] != null)
                {
                    var jsonResponse = JObject.Parse(item["_msg"].ToString());
                    var transactionId = jsonResponse["transaction_id"].ToString();
                    var errorCode = jsonResponse["extend_info"]["rawData"]["ErrorCode"].ToString();
                    result.Add($"{transactionId},{errorCode}");
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }

        return result;
    }
}
```

#### 2.2 程式碼解析

🔧 **關鍵步驟**

1. **讀取日誌檔案** - `File.ReadAllText(LogFilePath)`
2. **解析 JSON 資料** - `JToken.Parse(logData)`
3. **迭代處理** - 遍歷每個 JSON 項目
4. **屬性檢查** - 驗證 `_msg` 屬性是否存在
5. **深層解析** - 使用 `JObject.Parse()` 處理巢狀 JSON
6. **資料提取** - 取得 `transaction_id` 和 `ErrorCode`

💡 **最佳實作**
- 使用 try-catch 處理解析異常
- 檢查屬性是否存在避免 null 參考錯誤
- 適當的錯誤處理和日誌記錄

## 三、外部 API 指定反序列化屬性

### 1. System.Text.Json 反序列化

當與外部 API 進行資料交換時，JSON 屬性名稱可能與 C# 屬性命名慣例不同。透過使用 `JsonPropertyName` 屬性，我們可以指定 JSON 屬性與 C# 屬性之間的映射關係。

#### 1.1 範例程式碼

```csharp
void Main()
{
    var salesOrderThirdPartyPaymentInfo = "{\"out_trade_no\":\"TG240627M00002\",\"txcurrcd\":\"HKD\",\"pay_type\":\"802801\",\"order_type\":\"payment\",\"cancel\":\"0\"}";
    var info = System.Text.Json.JsonSerializer.Deserialize<QFPayThirdPartyPaymentInfoEntity>(salesOrderThirdPartyPaymentInfo);
    
    info.Dump();
}
```

#### 1.2 實體類別定義

```csharp
public class QFPayThirdPartyPaymentInfoEntity
{
    /// <summary>
    /// 訂單編號
    /// </summary>
    [JsonPropertyName("out_trade_no")]
    public string OutTradeNo { get; set; }

    /// <summary>
    /// 幣別
    /// </summary>
    [JsonPropertyName("txcurrcd")]
    public string Txcurrcd { get; set; }

    /// <summary>
    /// 付款方式 (QFPay 代號)
    /// </summary>
    [JsonPropertyName("pay_type")]
    public string PayType { get; set; }

    /// <summary>
    /// 訂單類型
    /// </summary>
    [JsonPropertyName("order_type")]
    public string OrderType { get; set; }

    /// <summary>
    /// 是否cancel
    /// </summary>
    [JsonPropertyName("cancel")]
    public string Cancel { get; set; }
}
```

#### 1.3 重要注意事項

⚠️ **套件一致性**

**重要原則：** Deserialize 與對應的 Attribute 要使用相同套件，例如本範例為 `System.Text.Json`

📋 **常見套件組合**

| JSON 函式庫 | 反序列化方法 | 屬性標註 |
|-------------|-------------|----------|
| `System.Text.Json` | `JsonSerializer.Deserialize<T>()` | `[JsonPropertyName]` |
| `Newtonsoft.Json` | `JsonConvert.DeserializeObject<T>()` | `[JsonProperty]` |

🔧 **實作要點**
- **屬性映射** - 使用 `JsonPropertyName` 將 JSON 屬性名稱映射到 C# 屬性
- **命名轉換** - 支援 snake_case 到 PascalCase 的自動轉換
- **型別安全** - 編譯時期確保屬性映射的正確性
- **效能優化** - `System.Text.Json` 提供更好的效能表現

💡 **最佳實作建議**
1. **統一套件使用** - 在同一專案中保持 JSON 處理套件的一致性
2. **明確屬性映射** - 為每個需要映射的屬性明確指定 `JsonPropertyName`
3. **文件化 API 契約** - 在註解中說明屬性的業務意義和來源
4. **錯誤處理** - 實作適當的例外處理機制處理反序列化失敗的情況