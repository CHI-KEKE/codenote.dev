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