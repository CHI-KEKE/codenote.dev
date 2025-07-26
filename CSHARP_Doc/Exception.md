# 🚨 C# 例外處理完整指南

## 📋 目錄
- [一、Try-Catch 使用時機與策略](#一try-catch-使用時機與策略)
  - [1.1 關鍵業務邏輯保護](#11-關鍵業務邏輯保護)
    - [1.1.1 購物車外部 API 呼叫範例](#111-購物車外部-api-呼叫範例)
- [二、例外處理常見問題](#二例外處理常見問題)
  - [2.1 重新 throw Exception 導致最內層的錯誤看不到](#21-重新-throw-exception-導致最內層的錯誤看不到)
    - [2.1.1 問題現象與解決方案](#211-問題現象與解決方案)
    - [2.1.2 例外鏈解析工具](#212-例外鏈解析工具)
---

## 一、Try-Catch 使用時機與策略

### 1.1 關鍵業務邏輯保護

#### 1.1.1 購物車外部 API 呼叫範例

當購物車需要呼叫外部 API 時，我們必須確保外部服務的問題不會影響購物車的核心功能。

> **💡 設計思維**
> 購物車是電商系統的核心功能，任何導致購物車無法使用的錯誤都是重大問題。因此，外部 API 的錯誤應該被妥善處理，而不是讓整個購物車系統崩潰。

```csharp
/// <summary>
/// 券中心 API 取得券相關資訊 (折數、券面文字、券種 ...)
/// </summary>
/// <param name="rewardEcouponEntities">The reward ecoupon entities.</param>
/// <returns> CouponInfoResponseEntity </returns>
private async Task<List<CouponInfoResponseEntity>> GetCouponInfos(IEnumerable<long> rewardEcouponEntities)
{
    //// 本功能僅用於顯示預計回饋用, 討論後若券站台發生錯誤謹記 Error log
    var couponInfos = new List<CouponInfoResponseEntity>();

    try
    {
        //// 建立 query string，包含 lang 參數
        var queryParams = new Dictionary<string, string>();
        if (string.IsNullOrEmpty(this._requestDataRetriever.Lang) == false)
        {
            queryParams.Add("lang", this._requestDataRetriever.Lang);
        }

        //// 組合 API Request
        var uri = new Uri(QueryHelpers.AddQueryString("api/coupons/coupon-info", queryParams), UriKind.Relative);

        //// 處理 API Response
        var couponInfoReponseEntities = await this._domainApiHelper.PostAsJsonAsync<IEnumerable<CouponInfoResponseEntity>, CouponInfoRequestEntity>(
                uri,
                new CouponInfoRequestEntity
                {
                    CouponIdList = rewardEcouponEntities
                },
                clientErr => throw new CouponGetException(clientErr),
                serverErr => throw new CouponGetException(CouponGetExceptionTypeEnum.UnknownError),
                DomainApiHelper.HttpclientCouponDomain);

        if (couponInfoReponseEntities?.Code == "Success" && couponInfoReponseEntities.Data?.Any() == true)
        {
            couponInfos = couponInfoReponseEntities.Data.ToList();
        }
    }
    catch (Exception ex)
    {
        this._logger.LogError(ex, $"處理 api/coupons/coupon-info 取得優惠券券面訊息是發生錯誤 : {ex?.Message}");
    }

    return couponInfos;
}
```

---

## 二、例外處理常見問題

### 2.1 重新 throw Exception 導致最內層的錯誤看不到

#### 2.1.1 問題現象與解決方案

以下範例展示了這個常見問題以及解決方案：

```csharp
void Main()
{
    try
    {
        Test();
    }
    catch (Exception ex)
    {
        ParseThatFuckingException(ex).Dump();
    }
}

public static void Test()
{
    try
    {
        test2();
    }
    catch (Exception ex)
    {
        throw new Exception(" 阿冠好腦呵呵，但 InnerCall 裡面好像說了什麼喔呵呵呵你看不到", ex);
    }
}

public static void test2()
{
    throw new Exception("內部");
}
```

**🚨 問題分析：**
- 當 `test2()` 拋出例外時，外層的 `Test()` 方法捕捉並重新包裝例外
- 如果只看外層例外訊息，很難知道真正的錯誤原因
- 內層的詳細錯誤資訊被埋在 `InnerException` 中

#### 2.1.2 例外鏈解析工具

為了解決這個問題，我們可以建立一個遞迴解析例外鏈的工具：

```csharp
public static string ParseThatFuckingException(Exception ex)
{
    var fullMessage = new StringBuilder();
    var level = 0;
    var currentLevelExceoton = ex;
    
    while (currentLevelExceoton != null)
    {
        var indent = new string(' ', level * 4);
        fullMessage.AppendLine($"{indent}錯誤訊息 : {currentLevelExceoton.Message}");
        
        if (string.IsNullOrEmpty(currentLevelExceoton.StackTrace) == false)
        {
            var stackTraces = currentLevelExceoton.StackTrace.Split(new[] { Environment.NewLine }, StringSplitOptions.RemoveEmptyEntries);
            foreach (var line in stackTraces)
            {
                fullMessage.AppendLine($"{indent} {line.Trim()}");
            }
        }
        
        currentLevelExceoton = currentLevelExceoton.InnerException;
        level++;
        
        if (currentLevelExceoton != null)
        {
            fullMessage.AppendLine($"{indent}內部例外 =>");
        }
    }
    
    return fullMessage.ToString();
}
```

##### 📊 輸出範例

```
錯誤訊息 : 阿冠好腦呵呵，但 InnerCall 裡面好像說了什麼喔呵呵呵你看不到
    at UserQuery.Test() in C:\Users\Example\LINQPad\Test.linq:line 15
    at UserQuery.Main() in C:\Users\Example\LINQPad\Test.linq:line 5
內部例外 =>
    錯誤訊息 : 內部
        at UserQuery.test2() in C:\Users\Example\LINQPad\Test.linq:line 21
        at UserQuery.Test() in C:\Users\Example\LINQPad\Test.linq:line 11
```