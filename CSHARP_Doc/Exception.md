# 🚨 C# 例外處理完整指南

## 📋 目錄
- [二、Try-Catch 使用時機與策略](#二try-catch-使用時機與策略)
  - [2.1 關鍵業務邏輯保護](#21-關鍵業務邏輯保護)
    - [2.1.1 購物車外部 API 呼叫範例](#211-購物車外部-api-呼叫範例)
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