# C# 程式碼品質提升指南

## 目錄
- [一、程式碼重構與共用能力](#一程式碼重構與共用能力)
  - [1.1 抽取共用驗證邏輯](#11-抽取共用驗證邏輯)
---

## 一、程式碼重構與共用能力

### 1.1 抽取共用驗證邏輯

在促銷引擎的開發中，常常需要對「指定商品類型」和「排除商品類型」進行相同的驗證邏輯。如果每次都重複寫相同的驗證程式碼，會造成：

透過建立共用的驗證函式 `ValidateSalePageTypeList`，我們可以優雅地解決上述問題：

```csharp
/// <summary>
/// 指定排除類型 / 指定排除商品 對應驗證
/// </summary>
/// <param name="entity">The entity.</param>
/// <returns></returns>
private ValidationFailure ValidateTargetTypeProductList(PromotionEngineBaseEntity entity)
{
    return ValidateSalePageTypeList(
        entity.TargetTypeDef, 
        entity.IncludeTargetIdList, 
        entity.IncludeProductSkuOuterIdList, 
        nameof(PromotionEngineBaseEntity.TargetTypeDef), 
        "指定商品類型, 請指定商品或料號")
        ?? ValidateSalePageTypeList(
        entity.ExcludeTargetTypeDef, 
        entity.ExcludeTargetIdList, 
        entity.ExcludeProductSkuOuterIdList, 
        nameof(PromotionEngineBaseEntity.ExcludeTargetTypeDef), 
        "排除商品類型, 請排除商品或料號");
}

/// <summary>
/// Validates the type list.
/// </summary>
/// <param name="typeDef">The type definition.</param>
/// <param name="targetIdList">The target identifier list.</param>
/// <param name="productSkuOuterIdList">The product sku outer identifier list.</param>
/// <param name="propertyName">Name of the property.</param>
/// <param name="errorMessage">The error message.</param>
/// <returns> ValidationFailure </returns>
private ValidationFailure ValidateSalePageTypeList(
    string typeDef, 
    List<long> targetIdList, 
    List<string> productSkuOuterIdList, 
    string propertyName, 
    string errorMessage)
{
    var noTargetIdSelected = targetIdList == null || targetIdList.Any() == false;
    var noProductSkuOuterIdSelected = productSkuOuterIdList == null || productSkuOuterIdList.Any() == false;
    
    if (typeDef == nameof(PromotionEngineTargetTypeDefEnum.SalePage) && 
        noTargetIdSelected && 
        noProductSkuOuterIdSelected)
    {
        return new ValidationFailure(propertyName, errorMessage);
    }

    return null;
}
```