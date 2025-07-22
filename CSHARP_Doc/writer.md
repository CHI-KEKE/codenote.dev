# C# StreamWriter 檔案寫入完整實戰指南 📝

> 掌握檔案寫入的藝術，讓資料永久保存！

## 目錄
1. [StreamWriter 基礎篇](#streamwriter-基礎篇)
   - 1.1 [什麼是 StreamWriter？](#什麼是-streamwriter)
   - 1.2 [基本檔案寫入操作](#基本檔案寫入操作)
   - 1.3 [路徑處理與特殊資料夾](#路徑處理與特殊資料夾)

2. [StreamWriter 核心操作篇](#streamwriter-核心操作篇)
   - 2.1 [建立與初始化](#建立與初始化)
   - 2.2 [寫入方法大全](#寫入方法大全)
   - 2.3 [同步與非同步操作](#同步與非同步操作)

3. [檔案操作模式篇](#檔案操作模式篇)
   - 3.1 [覆寫模式 vs 附加模式](#覆寫模式-vs-附加模式)
   - 3.2 [編碼設定](#編碼設定)
   - 3.3 [緩衝區管理](#緩衝區管理)

4. [實務應用篇](#實務應用篇)
   - 4.1 [日誌記錄系統](#日誌記錄系統)
   - 4.2 [設定檔產生](#設定檔產生)
   - 4.3 [CSV 資料匯出](#csv-資料匯出)
   - 4.4 [報表產生](#報表產生)

5. [進階技巧篇](#進階技巧篇)
   - 5.1 [檔案鎖定與併發處理](#檔案鎖定與併發處理)
   - 5.2 [大檔案寫入優化](#大檔案寫入優化)
   - 5.3 [錯誤處理與復原](#錯誤處理與復原)

6. [最佳實踐與效能優化](#最佳實踐與效能優化)
   - 6.1 [常見陷阱](#常見陷阱)
   - 6.2 [效能優化技巧](#效能優化技巧)
   - 6.3 [安全性考量](#安全性考量)

---

## StreamWriter 基礎篇

### 什麼是 StreamWriter？

StreamWriter 是 .NET 中專門用於寫入文字檔案的類別，它提供了高效且靈活的檔案寫入功能。

🎯 **核心特性**
- **文字導向**: 專門處理文字資料
- **緩衝寫入**: 提升寫入效能
- **編碼支援**: 支援多種文字編碼
- **自動管理**: 實作 IDisposable，支援 using 語句

```csharp
// 基本使用範例
using (var writer = new StreamWriter("example.txt"))
{
    writer.WriteLine("Hello, World!");
    writer.WriteLine("這是第二行");
}
```

### 基本檔案寫入操作

讓我們從您提供的核心範例開始：

```csharp
public class BasicFileWriting
{
    public static async Task WriteTimeLog()
    {
        // 🎯 您的核心範例：建立路徑並寫入時間戳記
        string path = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), 
            "log.txt"
        );

        using (var writer = new StreamWriter(path, true)) // true = 附加模式
        {
            await writer.WriteLineAsync(DateTime.Now.ToString("HH:mm:ss"));
        }
    }
    
    public static async Task DetailedExample()
    {
        string path = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), 
            "detailed_log.txt"
        );

        using (var writer = new StreamWriter(path, true))
        {
            // 寫入詳細的日誌資訊
            await writer.WriteLineAsync($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] 應用程式啟動");
            await writer.WriteLineAsync($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] 使用者: {Environment.UserName}");
            await writer.WriteLineAsync($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] 電腦名稱: {Environment.MachineName}");
            await writer.WriteLineAsync(""); // 空行分隔
        }
    }
}
```

### 路徑處理與特殊資料夾

```csharp
public class PathHandling
{
    public static void DemonstrateSpecialFolders()
    {
        // 🏠 常用的特殊資料夾
        var specialFolders = new Dictionary<string, Environment.SpecialFolder>
        {
            ["我的文件"] = Environment.SpecialFolder.MyDocuments,
            ["桌面"] = Environment.SpecialFolder.Desktop,
            ["應用程式資料"] = Environment.SpecialFolder.ApplicationData,
            ["本機應用程式資料"] = Environment.SpecialFolder.LocalApplicationData,
            ["暫存資料夾"] = Environment.SpecialFolder.Templates, // 實際上應該用 Path.GetTempPath()
            ["程式檔案"] = Environment.SpecialFolder.ProgramFiles
        };

        foreach (var folder in specialFolders)
        {
            string folderPath = Environment.GetFolderPath(folder.Value);
            Console.WriteLine($"{folder.Key}: {folderPath}");
        }
        
        // 🗂️ 建立應用程式專用資料夾
        string appDataPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
            "MyApplication",
            "Logs"
        );
        
        // 確保目錄存在
        Directory.CreateDirectory(appDataPath);
        
        string logFile = Path.Combine(appDataPath, $"app_log_{DateTime.Now:yyyyMMdd}.txt");
        
        Console.WriteLine($"日誌檔案路徑: {logFile}");
    }
    
    public static string GetSafeFilePath(string fileName)
    {
        // 移除檔名中的非法字元
        char[] invalidChars = Path.GetInvalidFileNameChars();
        string safeFileName = string.Join("_", fileName.Split(invalidChars, StringSplitOptions.RemoveEmptyEntries));
        
        return Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
            safeFileName
        );
    }
}
```

---

## StreamWriter 核心操作篇

### 建立與初始化

```csharp
public class StreamWriterCreation
{
    public static async Task DemonstrateCreationMethods()
    {
        string filePath = "example.txt";
        
        // 1️⃣ 基本建立方式
        using (var writer1 = new StreamWriter(filePath))
        {
            await writer1.WriteLineAsync("覆寫模式");
        }
        
        // 2️⃣ 附加模式
        using (var writer2 = new StreamWriter(filePath, append: true))
        {
            await writer2.WriteLineAsync("附加模式");
        }
        
        // 3️⃣ 指定編碼
        using (var writer3 = new StreamWriter(filePath, append: true, encoding: Encoding.UTF8))
        {
            await writer3.WriteLineAsync("UTF-8 編碼");
        }
        
        // 4️⃣ 指定緩衝區大小
        using (var writer4 = new StreamWriter(filePath, append: true, encoding: Encoding.UTF8, bufferSize: 4096))
        {
            await writer4.WriteLineAsync("自訂緩衝區大小");
        }
        
        // 5️⃣ 從 FileStream 建立
        using (var fileStream = new FileStream(filePath, FileMode.Append))
        using (var writer5 = new StreamWriter(fileStream))
        {
            await writer5.WriteLineAsync("從 FileStream 建立");
        }
        
        // 6️⃣ 使用 File.CreateText 和 File.AppendText
        using (var writer6 = File.CreateText("new_file.txt"))
        {
            await writer6.WriteLineAsync("使用 File.CreateText");
        }
        
        using (var writer7 = File.AppendText("new_file.txt"))
        {
            await writer7.WriteLineAsync("使用 File.AppendText");
        }
    }
}
```

### 寫入方法大全

```csharp
public class WritingMethods
{
    public static async Task DemonstrateWritingMethods()
    {
        string path = "writing_methods.txt";
        
        using (var writer = new StreamWriter(path))
        {
            // 📝 基本寫入方法
            
            // Write - 不換行
            writer.Write("Hello ");
            writer.Write("World");
            
            // WriteLine - 自動換行
            writer.WriteLine("!"); // 完成 "Hello World!"
            
            // WriteAsync - 非同步不換行
            await writer.WriteAsync("Async ");
            await writer.WriteAsync("Write");
            
            // WriteLineAsync - 非同步換行
            await writer.WriteLineAsync("!"); // 完成 "Async Write!"
            
            // 📊 格式化寫入
            int number = 42;
            string name = "張三";
            
            // 使用字串插值
            await writer.WriteLineAsync($"姓名: {name}, 數字: {number}");
            
            // 使用 String.Format
            await writer.WriteLineAsync(string.Format("姓名: {0}, 數字: {1}", name, number));
            
            // 🔢 寫入不同資料型別
            await writer.WriteLineAsync(DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"));
            await writer.WriteLineAsync(123.456.ToString("F2"));
            await writer.WriteLineAsync(true.ToString());
            
            // 📋 寫入集合資料
            var items = new[] { "項目1", "項目2", "項目3" };
            foreach (var item in items)
            {
                await writer.WriteLineAsync($"- {item}");
            }
            
            // 或使用 LINQ
            await writer.WriteLineAsync(string.Join(Environment.NewLine, 
                items.Select(item => $"• {item}")));
        }
    }
    
    public static async Task WriteComplexData()
    {
        string path = "complex_data.txt";
        
        var products = new[]
        {
            new { Name = "筆記型電腦", Price = 25000, Stock = 10 },
            new { Name = "滑鼠", Price = 500, Stock = 50 },
            new { Name = "鍵盤", Price = 1200, Stock = 30 }
        };
        
        using (var writer = new StreamWriter(path))
        {
            // 寫入標題
            await writer.WriteLineAsync("產品清單報表");
            await writer.WriteLineAsync("=".PadRight(50, '='));
            await writer.WriteLineAsync();
            
            // 寫入表頭
            await writer.WriteLineAsync($"{"產品名稱",-15} {"價格",8} {"庫存",6}");
            await writer.WriteLineAsync("-".PadRight(50, '-'));
            
            // 寫入資料
            foreach (var product in products)
            {
                await writer.WriteLineAsync($"{product.Name,-15} {product.Price,8:C0} {product.Stock,6}");
            }
            
            // 寫入統計
            await writer.WriteLineAsync("-".PadRight(50, '-'));
            await writer.WriteLineAsync($"總計 {products.Length} 項產品");
            await writer.WriteLineAsync($"總價值: {products.Sum(p => p.Price * p.Stock):C0}");
        }
    }
}
```

### 同步與非同步操作

```csharp
public class SyncAsyncOperations
{
    // 🚀 非同步操作 - 推薦用於 I/O 密集型操作
    public static async Task AsyncWriteExample()
    {
        string path = "async_example.txt";
        
        using (var writer = new StreamWriter(path))
        {
            // 模擬大量資料寫入
            for (int i = 0; i < 1000; i++)
            {
                await writer.WriteLineAsync($"第 {i + 1} 行資料 - {DateTime.Now:HH:mm:ss.fff}");
                
                // 每 100 行強制清理緩衝區
                if (i % 100 == 0)
                {
                    await writer.FlushAsync();
                }
            }
        }
    }
    
    // 🐌 同步操作 - 適用於簡單、快速的寫入
    public static void SyncWriteExample()
    {
        string path = "sync_example.txt";
        
        using (var writer = new StreamWriter(path))
        {
            for (int i = 0; i < 100; i++)
            {
                writer.WriteLine($"第 {i + 1} 行資料 - {DateTime.Now:HH:mm:ss.fff}");
            }
            
            writer.Flush(); // 強制寫入
        }
    }
    
    // ⚡ 效能比較
    public static async Task PerformanceComparison()
    {
        const int iterations = 10000;
        
        // 同步寫入測試
        var syncWatch = Stopwatch.StartNew();
        using (var writer = new StreamWriter("sync_test.txt"))
        {
            for (int i = 0; i < iterations; i++)
            {
                writer.WriteLine($"同步資料 {i}");
            }
        }
        syncWatch.Stop();
        
        // 非同步寫入測試
        var asyncWatch = Stopwatch.StartNew();
        using (var writer = new StreamWriter("async_test.txt"))
        {
            for (int i = 0; i < iterations; i++)
            {
                await writer.WriteLineAsync($"非同步資料 {i}");
            }
        }
        asyncWatch.Stop();
        
        Console.WriteLine($"同步寫入時間: {syncWatch.ElapsedMilliseconds} ms");
        Console.WriteLine($"非同步寫入時間: {asyncWatch.ElapsedMilliseconds} ms");
    }
}
```

---

## 檔案操作模式篇

### 覆寫模式 vs 附加模式

```csharp
public class FileModesComparison
{
    public static async Task DemonstrateFileModes()
    {
        string testFile = "mode_test.txt";
        
        // 🔄 覆寫模式 (預設)
        Console.WriteLine("=== 覆寫模式測試 ===");
        
        // 第一次寫入
        using (var writer = new StreamWriter(testFile, append: false))
        {
            await writer.WriteLineAsync("第一次寫入 - 行 1");
            await writer.WriteLineAsync("第一次寫入 - 行 2");
        }
        
        Console.WriteLine("第一次寫入後的內容:");
        Console.WriteLine(await File.ReadAllTextAsync(testFile));
        
        // 第二次寫入 (覆寫)
        using (var writer = new StreamWriter(testFile, append: false))
        {
            await writer.WriteLineAsync("第二次寫入 - 覆寫了原內容");
        }
        
        Console.WriteLine("第二次寫入後的內容:");
        Console.WriteLine(await File.ReadAllTextAsync(testFile));
        
        // ➕ 附加模式
        Console.WriteLine("\n=== 附加模式測試 ===");
        
        // 重新建立檔案
        using (var writer = new StreamWriter(testFile, append: false))
        {
            await writer.WriteLineAsync("原始內容 - 行 1");
            await writer.WriteLineAsync("原始內容 - 行 2");
        }
        
        Console.WriteLine("原始內容:");
        Console.WriteLine(await File.ReadAllTextAsync(testFile));
        
        // 附加新內容
        using (var writer = new StreamWriter(testFile, append: true))
        {
            await writer.WriteLineAsync("附加內容 - 行 3");
            await writer.WriteLineAsync("附加內容 - 行 4");
        }
        
        Console.WriteLine("附加後的內容:");
        Console.WriteLine(await File.ReadAllTextAsync(testFile));
    }
    
    // 🎯 實務應用：根據日期決定是否附加
    public static async Task SmartLogWriting()
    {
        string logDir = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
            "ApplicationLogs"
        );
        
        Directory.CreateDirectory(logDir);
        
        string todayLogFile = Path.Combine(logDir, $"log_{DateTime.Now:yyyyMMdd}.txt");
        
        // 檢查是否為今天第一次寫入
        bool isFirstWriteToday = !File.Exists(todayLogFile);
        
        using (var writer = new StreamWriter(todayLogFile, append: true))
        {
            if (isFirstWriteToday)
            {
                await writer.WriteLineAsync($"=== 日誌開始 {DateTime.Now:yyyy-MM-dd} ===");
                await writer.WriteLineAsync($"應用程式版本: 1.0.0");
                await writer.WriteLineAsync($"使用者: {Environment.UserName}");
                await writer.WriteLineAsync("");
            }
            
            await writer.WriteLineAsync($"[{DateTime.Now:HH:mm:ss}] 新的日誌項目");
        }
    }
}
```

### 編碼設定

```csharp
public class EncodingExamples
{
    public static async Task DemonstrateEncodings()
    {
        var encodings = new Dictionary<string, Encoding>
        {
            ["UTF-8"] = Encoding.UTF8,
            ["UTF-8 with BOM"] = new UTF8Encoding(true),
            ["UTF-16"] = Encoding.Unicode,
            ["ASCII"] = Encoding.ASCII,
            ["Big5"] = Encoding.GetEncoding("big5"),
            ["GB2312"] = Encoding.GetEncoding("gb2312")
        };
        
        string testText = "測試文字 - Hello World! 中文字元測試 123";
        
        foreach (var encoding in encodings)
        {
            string fileName = $"encoding_test_{encoding.Key.Replace(" ", "_")}.txt";
            
            try
            {
                using (var writer = new StreamWriter(fileName, false, encoding.Value))
                {
                    await writer.WriteLineAsync($"編碼類型: {encoding.Key}");
                    await writer.WriteLineAsync($"內容: {testText}");
                    await writer.WriteLineAsync($"編碼名稱: {encoding.Value.EncodingName}");
                    await writer.WriteLineAsync($"碼頁: {encoding.Value.CodePage}");
                }
                
                // 檢查檔案大小
                var fileInfo = new FileInfo(fileName);
                Console.WriteLine($"{encoding.Key}: {fileInfo.Length} bytes");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"{encoding.Key}: 錯誤 - {ex.Message}");
            }
        }
    }
    
    // 🌍 處理多語言文字
    public static async Task WriteMultiLanguageContent()
    {
        string path = "multilanguage.txt";
        
        using (var writer = new StreamWriter(path, false, Encoding.UTF8))
        {
            await writer.WriteLineAsync("多語言文字測試");
            await writer.WriteLineAsync("===================");
            await writer.WriteLineAsync();
            
            // 中文
            await writer.WriteLineAsync("中文: 你好世界");
            
            // 英文
            await writer.WriteLineAsync("English: Hello World");
            
            // 日文
            await writer.WriteLineAsync("日本語: こんにちは世界");
            
            // 韓文
            await writer.WriteLineAsync("한국어: 안녕하세요 세계");
            
            // 阿拉伯文
            await writer.WriteLineAsync("العربية: مرحبا بالعالم");
            
            // 俄文
            await writer.WriteLineAsync("Русский: Привет мир");
            
            // Emoji
            await writer.WriteLineAsync("Emoji: 🌍🌎🌏 Hello World! 👋");
        }
    }
}
```

### 緩衝區管理

```csharp
public class BufferManagement
{
    public static async Task DemonstrateBuffering()
    {
        string path = "buffer_test.txt";
        
        // 🔄 自動緩衝 (預設)
        using (var writer = new StreamWriter(path))
        {
            for (int i = 0; i < 1000; i++)
            {
                await writer.WriteLineAsync($"自動緩衝 - 行 {i}");
                
                // 每 100 行檢查一次
                if (i % 100 == 0)
                {
                    Console.WriteLine($"已寫入 {i} 行，檔案大小: {new FileInfo(path).Length} bytes");
                }
            }
            // 使用 using 時會自動 Flush
        }
        
        // 🚀 手動控制緩衝
        using (var writer = new StreamWriter(path + "_manual", false, Encoding.UTF8, bufferSize: 1024))
        {
            for (int i = 0; i < 1000; i++)
            {
                await writer.WriteLineAsync($"手動緩衝 - 行 {i}");
                
                // 每 50 行強制寫入
                if (i % 50 == 0)
                {
                    await writer.FlushAsync();
                    Console.WriteLine($"強制寫入 - 行 {i}");
                }
            }
        }
        
        // 🎯 即時寫入 (無緩衝)
        using (var fileStream = new FileStream(path + "_immediate", FileMode.Create))
        using (var writer = new StreamWriter(fileStream))
        {
            writer.AutoFlush = true; // 每次寫入後自動 Flush
            
            for (int i = 0; i < 100; i++)
            {
                await writer.WriteLineAsync($"即時寫入 - 行 {i}");
                // 資料會立即寫入磁碟
            }
        }
    }
    
    public static async Task BufferSizeComparison()
    {
        const int lines = 10000;
        var bufferSizes = new[] { 512, 1024, 4096, 8192, 16384 };
        
        foreach (int bufferSize in bufferSizes)
        {
            string fileName = $"buffer_{bufferSize}.txt";
            
            var stopwatch = Stopwatch.StartNew();
            
            using (var writer = new StreamWriter(fileName, false, Encoding.UTF8, bufferSize))
            {
                for (int i = 0; i < lines; i++)
                {
                    await writer.WriteLineAsync($"緩衝區大小 {bufferSize} - 行 {i}");
                }
            }
            
            stopwatch.Stop();
            
            Console.WriteLine($"緩衝區 {bufferSize} bytes: {stopwatch.ElapsedMilliseconds} ms");
        }
    }
}
```

---

## 實務應用篇

### 日誌記錄系統

基於您的範例，讓我們建立一個完整的日誌系統：

```csharp
public class LoggingSystem
{
    private readonly string _logDirectory;
    private readonly object _lockObject = new object();
    
    public LoggingSystem(string applicationName = "MyApp")
    {
        _logDirectory = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
            applicationName,
            "Logs"
        );
        
        Directory.CreateDirectory(_logDirectory);
    }
    
    // 🎯 基於您的原始範例擴展
    public async Task LogSimpleTimeAsync()
    {
        string path = Path.Combine(_logDirectory, "simple_time_log.txt");
        
        using (var writer = new StreamWriter(path, true))
        {
            await writer.WriteLineAsync(DateTime.Now.ToString("HH:mm:ss"));
        }
    }
    
    // 📊 詳細的日誌記錄
    public async Task LogAsync(string level, string message, string category = "General")
    {
        string fileName = $"log_{DateTime.Now:yyyyMMdd}.txt";
        string path = Path.Combine(_logDirectory, fileName);
        
        string logEntry = $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss.fff}] [{level}] [{category}] {message}";
        
        // 使用鎖確保執行緒安全
        lock (_lockObject)
        {
            using (var writer = new StreamWriter(path, true, Encoding.UTF8))
            {
                writer.WriteLine(logEntry);
            }
        }
    }
    
    // 🚨 不同等級的日誌方法
    public async Task LogInfoAsync(string message, string category = "Info")
        => await LogAsync("INFO", message, category);
    
    public async Task LogWarningAsync(string message, string category = "Warning")
        => await LogAsync("WARN", message, category);
    
    public async Task LogErrorAsync(string message, string category = "Error")
        => await LogAsync("ERROR", message, category);
    
    public async Task LogErrorAsync(Exception ex, string category = "Exception")
        => await LogAsync("ERROR", $"{ex.Message}\n{ex.StackTrace}", category);
    
    // 📈 效能日誌
    public async Task LogPerformanceAsync(string operation, TimeSpan duration)
    {
        string fileName = $"performance_{DateTime.Now:yyyyMMdd}.txt";
        string path = Path.Combine(_logDirectory, fileName);
        
        string logEntry = $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] {operation}: {duration.TotalMilliseconds:F2} ms";
        
        using (var writer = new StreamWriter(path, true))
        {
            await writer.WriteLineAsync(logEntry);
        }
    }
}

// 使用範例
public class LoggingExample
{
    public static async Task DemonstrateLogging()
    {
        var logger = new LoggingSystem("MyApplication");
        
        // 簡單時間記錄 (您的原始範例)
        await logger.LogSimpleTimeAsync();
        
        // 各種日誌等級
        await logger.LogInfoAsync("應用程式啟動", "Startup");
        await logger.LogWarningAsync("設定檔案不存在，使用預設值", "Configuration");
        
        try
        {
            // 模擬可能發生錯誤的操作
            throw new InvalidOperationException("模擬錯誤");
        }
        catch (Exception ex)
        {
            await logger.LogErrorAsync(ex, "Operations");
        }
        
        // 效能記錄
        var stopwatch = Stopwatch.StartNew();
        await Task.Delay(100); // 模擬耗時操作
        stopwatch.Stop();
        
        await logger.LogPerformanceAsync("資料載入", stopwatch.Elapsed);
    }
}
```

### 設定檔產生

```csharp
public class ConfigurationFileGenerator
{
    public static async Task GenerateAppConfig()
    {
        string configPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
            "MyApp",
            "config.txt"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(configPath));
        
        using (var writer = new StreamWriter(configPath, false, Encoding.UTF8))
        {
            await writer.WriteLineAsync("# 應用程式設定檔");
            await writer.WriteLineAsync($"# 產生時間: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
            await writer.WriteLineAsync();
            
            await writer.WriteLineAsync("[Database]");
            await writer.WriteLineAsync("ConnectionString=Server=localhost;Database=MyApp;");
            await writer.WriteLineAsync("Timeout=30");
            await writer.WriteLineAsync();
            
            await writer.WriteLineAsync("[Logging]");
            await writer.WriteLineAsync("Level=Info");
            await writer.WriteLineAsync("MaxFileSize=10MB");
            await writer.WriteLineAsync("RetainDays=30");
            await writer.WriteLineAsync();
            
            await writer.WriteLineAsync("[Application]");
            await writer.WriteLineAsync($"Version=1.0.0");
            await writer.WriteLineAsync($"InstallDate={DateTime.Now:yyyy-MM-dd}");
            await writer.WriteLineAsync($"UserName={Environment.UserName}");
        }
        
        Console.WriteLine($"設定檔已產生: {configPath}");
    }
    
    // JSON 風格設定檔
    public static async Task GenerateJsonStyleConfig()
    {
        string configPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
            "MyApp",
            "settings.json"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(configPath));
        
        using (var writer = new StreamWriter(configPath, false, Encoding.UTF8))
        {
            await writer.WriteLineAsync("{");
            await writer.WriteLineAsync("  \"database\": {");
            await writer.WriteLineAsync("    \"connectionString\": \"Server=localhost;Database=MyApp;\",");
            await writer.WriteLineAsync("    \"timeout\": 30");
            await writer.WriteLineAsync("  },");
            await writer.WriteLineAsync("  \"logging\": {");
            await writer.WriteLineAsync("    \"level\": \"Info\",");
            await writer.WriteLineAsync("    \"maxFileSize\": \"10MB\",");
            await writer.WriteLineAsync("    \"retainDays\": 30");
            await writer.WriteLineAsync("  },");
            await writer.WriteLineAsync("  \"application\": {");
            await writer.WriteLineAsync("    \"version\": \"1.0.0\",");
            await writer.WriteLineAsync($"    \"installDate\": \"{DateTime.Now:yyyy-MM-dd}\",");
            await writer.WriteLineAsync($"    \"userName\": \"{Environment.UserName}\"");
            await writer.WriteLineAsync("  }");
            await writer.WriteLineAsync("}");
        }
    }
}
```

### CSV 資料匯出

```csharp
public class CsvExporter
{
    public class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Department { get; set; }
        public decimal Salary { get; set; }
        public DateTime HireDate { get; set; }
    }
    
    public static async Task ExportEmployeesToCsv()
    {
        var employees = new[]
        {
            new Employee { Id = 1, Name = "張三", Department = "資訊部", Salary = 50000, HireDate = new DateTime(2020, 1, 15) },
            new Employee { Id = 2, Name = "李四", Department = "業務部", Salary = 45000, HireDate = new DateTime(2019, 6, 20) },
            new Employee { Id = 3, Name = "王五", Department = "人事部", Salary = 48000, HireDate = new DateTime(2021, 3, 10) }
        };
        
        string csvPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.Desktop),
            $"employees_{DateTime.Now:yyyyMMdd_HHmmss}.csv"
        );
        
        using (var writer = new StreamWriter(csvPath, false, Encoding.UTF8))
        {
            // 寫入 BOM 以確保 Excel 正確識別 UTF-8
            writer.Write('\ufeff');
            
            // 寫入標題行
            await writer.WriteLineAsync("員工編號,姓名,部門,薪資,到職日期");
            
            // 寫入資料行
            foreach (var employee in employees)
            {
                string line = $"{employee.Id}," +
                             $"\"{employee.Name}\"," +
                             $"\"{employee.Department}\"," +
                             $"{employee.Salary}," +
                             $"{employee.HireDate:yyyy-MM-dd}";
                
                await writer.WriteLineAsync(line);
            }
        }
        
        Console.WriteLine($"CSV 檔案已匯出: {csvPath}");
    }
    
    // 泛型 CSV 匯出器
    public static async Task ExportToCsv<T>(IEnumerable<T> data, string fileName, 
        Dictionary<string, Func<T, object>> columnMappings)
    {
        string csvPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.Desktop),
            fileName
        );
        
        using (var writer = new StreamWriter(csvPath, false, Encoding.UTF8))
        {
            writer.Write('\ufeff'); // BOM
            
            // 寫入標題
            await writer.WriteLineAsync(string.Join(",", columnMappings.Keys));
            
            // 寫入資料
            foreach (var item in data)
            {
                var values = columnMappings.Values.Select(func => 
                {
                    var value = func(item);
                    if (value is string strValue && (strValue.Contains(',') || strValue.Contains('"')))
                    {
                        return $"\"{strValue.Replace("\"", "\"\"")}\"";
                    }
                    return value?.ToString() ?? "";
                });
                
                await writer.WriteLineAsync(string.Join(",", values));
            }
        }
    }
}
```

### 報表產生

```csharp
public class ReportGenerator
{
    public static async Task GenerateSalesReport()
    {
        var salesData = new[]
        {
            new { Date = DateTime.Now.AddDays(-30), Product = "筆記型電腦", Quantity = 5, Amount = 125000 },
            new { Date = DateTime.Now.AddDays(-25), Product = "滑鼠", Quantity = 20, Amount = 10000 },
            new { Date = DateTime.Now.AddDays(-20), Product = "鍵盤", Quantity = 15, Amount = 18000 },
            new { Date = DateTime.Now.AddDays(-15), Product = "螢幕", Quantity = 8, Amount = 64000 },
            new { Date = DateTime.Now.AddDays(-10), Product = "筆記型電腦", Quantity = 3, Amount = 75000 }
        };
        
        string reportPath = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.Desktop),
            $"銷售報表_{DateTime.Now:yyyyMMdd}.txt"
        );
        
        using (var writer = new StreamWriter(reportPath, false, Encoding.UTF8))
        {
            // 報表標題
            await writer.WriteLineAsync("╔══════════════════════════════════════════════════════════════╗");
            await writer.WriteLineAsync("║                          銷售報表                             ║");
            await writer.WriteLineAsync("╚══════════════════════════════════════════════════════════════╝");
            await writer.WriteLineAsync();
            
            await writer.WriteLineAsync($"報表產生時間: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
            await writer.WriteLineAsync($"報表期間: {salesData.Min(x => x.Date):yyyy-MM-dd} ~ {salesData.Max(x => x.Date):yyyy-MM-dd}");
            await writer.WriteLineAsync();
            
            // 詳細資料
            await writer.WriteLineAsync("詳細銷售資料:");
            await writer.WriteLineAsync("━".PadRight(80, '━'));
            await writer.WriteLineAsync($"{"日期",-12} {"產品名稱",-15} {"數量",6} {"金額",12}");
            await writer.WriteLineAsync("─".PadRight(80, '─'));
            
            foreach (var sale in salesData.OrderBy(x => x.Date))
            {
                await writer.WriteLineAsync($"{sale.Date:yyyy-MM-dd} {sale.Product,-15} {sale.Quantity,6} {sale.Amount,12:N0}");
            }
            
            // 統計資料
            await writer.WriteLineAsync("─".PadRight(80, '─'));
            await writer.WriteLineAsync();
            
            await writer.WriteLineAsync("統計摘要:");
            await writer.WriteLineAsync("━".PadRight(40, '━'));
            await writer.WriteLineAsync($"總銷售筆數: {salesData.Length}");
            await writer.WriteLineAsync($"總銷售金額: {salesData.Sum(x => x.Amount):N0}");
            await writer.WriteLineAsync($"平均銷售金額: {salesData.Average(x => x.Amount):N0}");
            await writer.WriteLineAsync($"最高單筆金額: {salesData.Max(x => x.Amount):N0}");
            await writer.WriteLineAsync($"最低單筆金額: {salesData.Min(x => x.Amount):N0}");
            
            // 產品統計
            await writer.WriteLineAsync();
            await writer.WriteLineAsync("產品銷售統計:");
            await writer.WriteLineAsync("━".PadRight(40, '━'));
            
            var productStats = salesData
                .GroupBy(x => x.Product)
                .Select(g => new
                {
                    Product = g.Key,
                    TotalQuantity = g.Sum(x => x.Quantity),
                    TotalAmount = g.Sum(x => x.Amount)
                })
                .OrderByDescending(x => x.TotalAmount);
            
            foreach (var stat in productStats)
            {
                await writer.WriteLineAsync($"{stat.Product}: 數量 {stat.TotalQuantity}, 金額 {stat.TotalAmount:N0}");
            }
        }
        
        Console.WriteLine($"銷售報表已產生: {reportPath}");
    }
}
```

由於文件內容豐富，我將按照您的指示考慮分割文件。讓我先完成第一部分，然後詢問是否需要建立第二部分文件。

這是第一部分，包含了基礎到實務應用。如果您覺得內容太長，我可以將進階技巧和最佳實踐部分建立為第二份文件。您希望我繼續完成剩餘部分，還是分割成兩份文件？