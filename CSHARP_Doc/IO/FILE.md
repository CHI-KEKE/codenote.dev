# 📁 C# 檔案操作完整指南

## 📖 目錄
- [一、檔案路徑操作](#一檔案路徑操作)
  - [1. 副檔名處理](#1-副檔名處理)
    - [1.1 讀取副檔名](#11-讀取副檔名)
    - [1.2 使用範例](#12-使用範例)
    - [1.3 實際應用](#13-實際應用)

---

## 一、檔案路徑操作

### 1. 副檔名處理

#### 1.1 讀取副檔名

使用 `Path.GetExtension()` 方法可以輕鬆取得檔案的副檔名資訊。

```csharp
Path.GetExtension("problematic-xstate-machine.ts").Dump();
```

##### 🔍 執行結果

```
.ts
```

#### 1.2 使用範例

```csharp
void Main()
{
    // 基本使用
    string fileName = "problematic-xstate-machine.ts";
    string extension = Path.GetExtension(fileName);
    Console.WriteLine($"副檔名: {extension}"); // 輸出: .ts
}
```

#### 1.3 實際應用

📋 **常見使用場景**
- **檔案類型判斷** - 根據副檔名決定處理方式
- **檔案分類** - 將不同類型的檔案進行分組
- **驗證檔案格式** - 確認上傳檔案的格式是否正確

🎯 **重要特點**
- 回傳值包含點號 (.) 
- 如果沒有副檔名則回傳空字串
- 不區分大小寫