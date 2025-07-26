# 🎯 C# Delegate 委派完整指南

## 📖 目錄

- [二、應用案例](#二應用案例)
  - [2.1 簡單的驗證邏輯](#21-簡單的驗證邏輯)
    - [2.1.1 使用 Func 委派](#211-使用-func-委派)
    - [2.1.2 使用 Predicate 委派](#212-使用-predicate-委派)
    - [2.1.3 實際測試結果](#213-實際測試結果)
---


## 二、應用案例

### 2.1 簡單的驗證邏輯

當我們需要執行簡單的驗證邏輯，但又不想為此獨立寫一個函式時，委派是一個很好的選擇。

#### 2.1.1 使用 Func 委派

```csharp
void Main()
{
    // 🔍 測試字串長度差異
    "Ａ".Length.Dump();  // 全形字元 A
    "A".Length.Dump();   // 半形字元 A
    
    // 🎯 使用 Func 委派建立產品 SKU 驗證邏輯
    Func<string, bool> _isValidProductSkuOuterId = productSkuOuterId => 
        System.Text.RegularExpressions.Regex.IsMatch(productSkuOuterId, @"^[a-zA-Z0-9ａ-ｚＡ-Ｚ０-９]+$");
    
    // 📝 測試不同的輸入
    _isValidProductSkuOuterId("Ａ０").Dump();   // true - 全形字元
    _isValidProductSkuOuterId("a9ｚ").Dump();   // true - 混合半形全形
    _isValidProductSkuOuterId(",").Dump();      // false - 包含特殊字元
}
```

#### 2.1.2 使用 Predicate 委派

```csharp
void Main()
{
    // 🎯 使用 Predicate 委派（專門用於 bool 回傳值）
    Predicate<string> isValidOuterId = x => 
        Regex.IsMatch(x, @"^[a-zA-Z0-9ａ-ｚＡ-Ｚ０-９]+$");
    
    // 📝 測試相同的驗證邏輯
    isValidOuterId("Ａ０").Dump();   // true
    isValidOuterId("a9ｚ").Dump();   // true  
    isValidOuterId(",").Dump();      // false
}
```