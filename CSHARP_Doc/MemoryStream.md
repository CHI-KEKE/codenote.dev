# C# MemoryStream 完整實戰指南 🚀

> 掌握記憶體中的虛擬檔案，讓資料處理飛起來！

## 目錄
1. [MemoryStream 基礎篇](#memorystream-基礎篇)
   - 1.1 [什麼是 MemoryStream？](#什麼是-memorystream)
   - 1.2 [MemoryStream vs 實體檔案](#memorystream-vs-實體檔案)
   - 1.3 [byte[] 與編碼轉換](#byte-與編碼轉換)

2. [MemoryStream 核心操作篇](#memorystream-核心操作篇)
   - 2.1 [建立與初始化](#建立與初始化)
   - 2.2 [讀寫操作](#讀寫操作)
   - 2.3 [位置控制與重設](#位置控制與重設)

3. [AWS S3 整合篇](#aws-s3-整合篇)
   - 3.1 [為什麼 S3 需要 MemoryStream？](#為什麼-s3-需要-memorystream)
   - 3.2 [字串上傳到 S3](#字串上傳到-s3)
   - 3.3 [圖片檔案上傳](#圖片檔案上傳)
   - 3.4 [JSON 資料上傳](#json-資料上傳)

4. [實務應用篇](#實務應用篇)
   - 4.1 [動態內容產生](#動態內容產生)
   - 4.2 [檔案格式轉換](#檔案格式轉換)
   - 4.3 [壓縮與解壓縮](#壓縮與解壓縮)
   - 4.4 [加密與解密](#加密與解密)

5. [效能優化篇](#效能優化篇)
   - 5.1 [記憶體管理](#記憶體管理)
   - 5.2 [大檔案處理策略](#大檔案處理策略)
   - 5.3 [池化技術](#池化技術)

6. [最佳實踐與常見陷阱](#最佳實踐與常見陷阱)
   - 6.1 [資源管理](#資源管理)
   - 6.2 [效能陷阱](#效能陷阱)
   - 6.3 [安全性考量](#安全性考量)

---

## MemoryStream 基礎篇

### 什麼是 MemoryStream？

> 📦 **生動比喻**：MemoryStream 就像一個**用 RAM 做的暫存小硬碟**！

MemoryStream 是一個存在記憶體中的虛擬檔案流，它讓我們可以在記憶體中模擬檔案操作，而不需要實際建立實體檔案。

🎯 **核心特性**
- **純記憶體操作**: 所有資料都存在 RAM 中
- **高速存取**: 比實體檔案快數十倍
- **動態大小**: 可以自動擴展容量
- **暫時性**: 程式結束後自動消失

```csharp
// 基本使用範例
using (var memoryStream = new MemoryStream())
{
    // 這裡的 memoryStream 就是一個虛擬檔案
    // 可以像操作真實檔案一樣讀寫資料
    byte[] data = Encoding.UTF8.GetBytes("Hello, Memory!");
    memoryStream.Write(data, 0, data.Length);
    
    Console.WriteLine($"虛擬檔案大小: {memoryStream.Length} bytes");
}
```

### MemoryStream vs 實體檔案

| 特性 | MemoryStream | 實體檔案 | 適用場景 |
|-----|-------------|----------|----------|
| **速度** | ⚡ 極快 | 🐌 相對慢 | 臨時處理、頻繁讀寫 |
| **容量** | 🔸 受記憶體限制 | 🔸 受硬碟限制 | 小到中型資料 |
| **持久性** | ❌ 程式結束即消失 | ✅ 永久保存 | 暫時性資料 |
| **併發存取** | ⚠️ 需要同步 | ⚠️ 需要鎖定 | 單執行緒優先 |
| **記憶體使用** | 🔴 高 | 🟢 低 | 依據資料大小決定 |

```csharp
public class StreamComparison
{
    public static async Task ComparePerformance()
    {
        string testData = string.Join("\n", Enumerable.Range(1, 1000).Select(i => $"測試資料行 {i}"));
        byte[] dataBytes = Encoding.UTF8.GetBytes(testData);
        
        // 🚀 MemoryStream 效能測試
        var stopwatch = Stopwatch.StartNew();
        
        for (int i = 0; i < 1000; i++)
        {
            using (var memStream = new MemoryStream())
            {
                await memStream.WriteAsync(dataBytes, 0, dataBytes.Length);
                memStream.Position = 0;
                var readBuffer = new byte[dataBytes.Length];
                await memStream.ReadAsync(readBuffer, 0, readBuffer.Length);
            }
        }
        
        stopwatch.Stop();
        Console.WriteLine($"MemoryStream: {stopwatch.ElapsedMilliseconds} ms");
        
        // 🐌 實體檔案效能測試
        stopwatch.Restart();
        
        for (int i = 0; i < 1000; i++)
        {
            string tempFile = Path.GetTempFileName();
            try
            {
                await File.WriteAllBytesAsync(tempFile, dataBytes);
                var readData = await File.ReadAllBytesAsync(tempFile);
            }
            finally
            {
                File.Delete(tempFile);
            }
        }
        
        stopwatch.Stop();
        Console.WriteLine($"實體檔案: {stopwatch.ElapsedMilliseconds} ms");
    }
}
```

### byte[] 與編碼轉換

> 🔄 **生動比喻**：把中文信寫好之後，要轉成信封裡的二進位檔案才能寄出！

理解 `byte[]` 和字串編碼是使用 MemoryStream 的關鍵。

```csharp
public class EncodingExamples
{
    public static void DemonstrateEncodingConversion()
    {
        string originalText = "Hello 世界! 🌍";
        
        // 🔤 字串 → byte[] (編碼)
        byte[] utf8Bytes = Encoding.UTF8.GetBytes(originalText);
        byte[] utf16Bytes = Encoding.Unicode.GetBytes(originalText);
        byte[] asciiBytes = Encoding.ASCII.GetBytes(originalText); // 會遺失非 ASCII 字元
        
        Console.WriteLine($"原始字串: {originalText}");
        Console.WriteLine($"UTF-8 位元組數: {utf8Bytes.Length}");
        Console.WriteLine($"UTF-16 位元組數: {utf16Bytes.Length}");
        Console.WriteLine($"ASCII 位元組數: {asciiBytes.Length}");
        
        // 🔤 byte[] → 字串 (解碼)
        string decodedFromUtf8 = Encoding.UTF8.GetString(utf8Bytes);
        string decodedFromUtf16 = Encoding.Unicode.GetString(utf16Bytes);
        string decodedFromAscii = Encoding.ASCII.GetString(asciiBytes);
        
        Console.WriteLine($"UTF-8 解碼: {decodedFromUtf8}");
        Console.WriteLine($"UTF-16 解碼: {decodedFromUtf16}");
        Console.WriteLine($"ASCII 解碼: {decodedFromAscii}"); // 會顯示問號或遺失字元
        
        // 🔍 檢視位元組內容
        Console.WriteLine($"UTF-8 位元組: {string.Join(" ", utf8Bytes.Select(b => b.ToString("X2")))}");
    }
    
    public static void ShowEncodingDifferences()
    {
        var testStrings = new[]
        {
            "Hello",           // 純英文
            "你好",            // 中文
            "🚀🌟✨",          // Emoji
            "Café naïve",      // 歐洲字元
            "Привет мир"       // 俄文
        };
        
        foreach (string text in testStrings)
        {
            var utf8 = Encoding.UTF8.GetBytes(text);
            var utf16 = Encoding.Unicode.GetBytes(text);
            
            Console.WriteLine($"文字: {text}");
            Console.WriteLine($"  UTF-8:  {utf8.Length} bytes");
            Console.WriteLine($"  UTF-16: {utf16.Length} bytes");
            Console.WriteLine();
        }
    }
}
```

---

## MemoryStream 核心操作篇

### 建立與初始化

```csharp
public class MemoryStreamCreation
{
    public static void DemonstrateCreationMethods()
    {
        // 1️⃣ 空的 MemoryStream
        using (var emptyStream = new MemoryStream())
        {
            Console.WriteLine($"空串流容量: {emptyStream.Capacity}");
            Console.WriteLine($"空串流長度: {emptyStream.Length}");
        }
        
        // 2️⃣ 指定初始容量
        using (var preAllocatedStream = new MemoryStream(1024)) // 1KB 初始容量
        {
            Console.WriteLine($"預配置容量: {preAllocatedStream.Capacity}");
        }
        
        // 3️⃣ 從現有 byte[] 建立
        byte[] existingData = Encoding.UTF8.GetBytes("現有資料");
        using (var fromBytesStream = new MemoryStream(existingData))
        {
            Console.WriteLine($"從位元組建立的長度: {fromBytesStream.Length}");
        }
        
        // 4️⃣ 唯讀模式
        using (var readOnlyStream = new MemoryStream(existingData, false)) // false = 唯讀
        {
            Console.WriteLine($"唯讀串流可寫: {readOnlyStream.CanWrite}");
        }
        
        // 5️⃣ 可擴展模式
        using (var expandableStream = new MemoryStream(existingData, 0, existingData.Length, true, true))
        {
            // 第四個參數 = 可寫入，第五個參數 = 可公開位元組
            Console.WriteLine($"可擴展: {expandableStream.CanWrite}");
        }
    }
    
    public static async Task CreateWithInitialContent()
    {
        string initialContent = "這是初始內容\n第二行\n第三行";
        byte[] contentBytes = Encoding.UTF8.GetBytes(initialContent);
        
        using (var stream = new MemoryStream(contentBytes))
        {
            // 讀取內容驗證
            stream.Position = 0; // 重設到開頭
            
            using (var reader = new StreamReader(stream, Encoding.UTF8))
            {
                string readContent = await reader.ReadToEndAsync();
                Console.WriteLine($"讀取到的內容:\n{readContent}");
            }
        }
    }
}
```

### 讀寫操作

```csharp
public class MemoryStreamOperations
{
    public static async Task DemonstrateBasicOperations()
    {
        using (var stream = new MemoryStream())
        {
            // ✍️ 寫入操作
            
            // 方法 1: 直接寫入 byte[]
            byte[] data1 = Encoding.UTF8.GetBytes("第一段資料\n");
            await stream.WriteAsync(data1, 0, data1.Length);
            
            // 方法 2: 使用 StreamWriter
            using (var writer = new StreamWriter(stream, Encoding.UTF8, leaveOpen: true))
            {
                await writer.WriteLineAsync("第二段資料");
                await writer.WriteLineAsync("第三段資料");
                await writer.FlushAsync();
            }
            
            // 方法 3: 寫入單一位元組
            stream.WriteByte(65); // ASCII 'A'
            stream.WriteByte(66); // ASCII 'B'
            
            Console.WriteLine($"寫入完成，總長度: {stream.Length} bytes");
            
            // 📖 讀取操作
            
            // 重設位置到開頭
            stream.Position = 0;
            
            // 方法 1: 讀取全部資料
            byte[] allData = stream.ToArray();
            string allText = Encoding.UTF8.GetString(allData);
            Console.WriteLine($"全部內容:\n{allText}");
            
            // 方法 2: 使用 StreamReader 逐行讀取
            stream.Position = 0;
            using (var reader = new StreamReader(stream, Encoding.UTF8, leaveOpen: true))
            {
                string line;
                int lineNumber = 1;
                while ((line = await reader.ReadLineAsync()) != null)
                {
                    Console.WriteLine($"第 {lineNumber} 行: {line}");
                    lineNumber++;
                }
            }
            
            // 方法 3: 部分讀取
            stream.Position = 0;
            byte[] buffer = new byte[10];
            int bytesRead = await stream.ReadAsync(buffer, 0, buffer.Length);
            string partialText = Encoding.UTF8.GetString(buffer, 0, bytesRead);
            Console.WriteLine($"部分讀取 ({bytesRead} bytes): {partialText}");
        }
    }
    
    public static void DemonstrateRandomAccess()
    {
        string content = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
        byte[] contentBytes = Encoding.UTF8.GetBytes(content);
        
        using (var stream = new MemoryStream(contentBytes))
        {
            // 🎯 隨機存取範例
            
            // 讀取中間部分 (位置 10-15)
            stream.Position = 10;
            byte[] middlePart = new byte[6];
            stream.Read(middlePart, 0, 6);
            Console.WriteLine($"中間部分: {Encoding.UTF8.GetString(middlePart)}");
            
            // 從結尾往前讀取
            stream.Position = stream.Length - 5;
            byte[] endPart = new byte[5];
            stream.Read(endPart, 0, 5);
            Console.WriteLine($"結尾部分: {Encoding.UTF8.GetString(endPart)}");
            
            // 覆寫中間內容
            stream.Position = 5;
            byte[] replacement = Encoding.UTF8.GetBytes("*****");
            stream.Write(replacement, 0, replacement.Length);
            
            // 檢視修改結果
            stream.Position = 0;
            byte[] modifiedContent = new byte[stream.Length];
            stream.Read(modifiedContent, 0, (int)stream.Length);
            Console.WriteLine($"修改後: {Encoding.UTF8.GetString(modifiedContent)}");
        }
    }
}
```

### 位置控制與重設

```csharp
public class PositionControl
{
    public static void DemonstratePositionManagement()
    {
        string data = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
        byte[] dataBytes = Encoding.UTF8.GetBytes(data);
        
        using (var stream = new MemoryStream(dataBytes))
        {
            Console.WriteLine($"初始位置: {stream.Position}");
            Console.WriteLine($"串流長度: {stream.Length}");
            
            // 🔄 不同的定位方法
            
            // Seek 到絕對位置
            stream.Seek(10, SeekOrigin.Begin);
            Console.WriteLine($"Seek 到位置 10: {stream.Position}");
            
            // 相對目前位置移動
            stream.Seek(5, SeekOrigin.Current);
            Console.WriteLine($"相對移動 +5: {stream.Position}");
            
            // 從結尾往前移動
            stream.Seek(-3, SeekOrigin.End);
            Console.WriteLine($"從結尾 -3: {stream.Position}");
            
            // 🔄 重設到開頭
            stream.Position = 0;
            Console.WriteLine($"重設後位置: {stream.Position}");
            
            // 📍 實用的位置操作範例
            DemonstratePositionTricks(stream);
        }
    }
    
    private static void DemonstratePositionTricks(MemoryStream stream)
    {
        // 技巧 1: 記住當前位置
        long savedPosition = stream.Position;
        
        // 做一些操作...
        stream.Seek(0, SeekOrigin.End); // 移到結尾
        Console.WriteLine($"移到結尾: {stream.Position}");
        
        // 恢復之前的位置
        stream.Position = savedPosition;
        Console.WriteLine($"恢復位置: {stream.Position}");
        
        // 技巧 2: 檢查是否到達結尾
        stream.Seek(0, SeekOrigin.End);
        bool isAtEnd = stream.Position == stream.Length;
        Console.WriteLine($"是否在結尾: {isAtEnd}");
        
        // 技巧 3: 計算剩餘可讀取的位元組數
        stream.Position = 10;
        long remainingBytes = stream.Length - stream.Position;
        Console.WriteLine($"從位置 10 還可讀取: {remainingBytes} bytes");
    }
    
    public static async Task PositionSafety()
    {
        using (var stream = new MemoryStream())
        {
            // 寫入一些測試資料
            byte[] testData = Encoding.UTF8.GetBytes("測試資料");
            await stream.WriteAsync(testData, 0, testData.Length);
            
            // ⚠️ 常見錯誤：忘記重設位置
            Console.WriteLine("忘記重設位置的問題:");
            
            // 嘗試讀取（位置在結尾）
            byte[] buffer1 = new byte[testData.Length];
            int bytesRead1 = await stream.ReadAsync(buffer1, 0, buffer1.Length);
            Console.WriteLine($"未重設位置，讀取到: {bytesRead1} bytes");
            
            // ✅ 正確做法：重設位置
            stream.Position = 0;
            byte[] buffer2 = new byte[testData.Length];
            int bytesRead2 = await stream.ReadAsync(buffer2, 0, buffer2.Length);
            string readText = Encoding.UTF8.GetString(buffer2, 0, bytesRead2);
            Console.WriteLine($"重設位置後，讀取到: {bytesRead2} bytes, 內容: {readText}");
        }
    }
}
```

---

## AWS S3 整合篇

### 為什麼 S3 需要 MemoryStream？

> 📦 **核心概念**：S3 接受的是「檔案的原始內容」，不管是圖片、文字還是影片，其實本質上都是一堆 bytes！

AWS S3 的 SDK 只能上傳「位元資料流（byte stream）」而不是純文字。這就是為什麼我們需要 MemoryStream：

🔍 **S3 上傳需求分析**：
- ✅ S3 接受：`Stream` 物件（包含 byte 資料）
- ❌ S3 不接受：純文字 `string`
- 🔄 解決方案：`string` → `byte[]` → `MemoryStream`

```csharp
// ❌ 這樣不行：S3 不接受純文字
// await s3Client.PutObjectAsync("bucket", "key", "Hello World");

// ✅ 這樣才對：需要轉換成位元組流
string content = "Hello World";
byte[] contentBytes = Encoding.UTF8.GetBytes(content);

using (var memoryStream = new MemoryStream(contentBytes))
{
    var request = new PutObjectRequest
    {
        BucketName = "my-bucket",
        Key = "my-file.txt",
        InputStream = memoryStream,
        ContentType = "text/plain"
    };
    
    await s3Client.PutObjectAsync(request);
}
```

### 字串上傳到 S3

```csharp
public class S3StringUpload
{
    private readonly AmazonS3Client _s3Client;
    
    public S3StringUpload()
    {
        _s3Client = new AmazonS3Client(); // 假設已設定認證
    }
    
    // 🎯 基礎字串上傳（基於您的核心概念）
    public async Task UploadStringToS3(string bucketName, string key, string content)
    {
        // 步驟 1: 字串轉 byte[]
        byte[] contentBytes = Encoding.UTF8.GetBytes(content);
        
        // 步驟 2: byte[] 放進 MemoryStream（虛擬檔案）
        using (var memoryStream = new MemoryStream(contentBytes))
        {
            // 步驟 3: MemoryStream 上傳到 S3
            var request = new PutObjectRequest
            {
                BucketName = bucketName,
                Key = key,
                InputStream = memoryStream,
                ContentType = "text/plain; charset=utf-8",
                ServerSideEncryptionMethod = ServerSideEncryptionMethod.AES256
            };
            
            try
            {
                var response = await _s3Client.PutObjectAsync(request);
                Console.WriteLine($"✅ 字串上傳成功: {key}");
                Console.WriteLine($"ETag: {response.ETag}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"❌ 上傳失敗: {ex.Message}");
                throw;
            }
        }
    }
    
    // 🌟 進階字串上傳（含 metadata）
    public async Task UploadStringWithMetadata(string bucketName, string key, string content, Dictionary<string, string> metadata = null)
    {
        byte[] contentBytes = Encoding.UTF8.GetBytes(content);
        
        using (var memoryStream = new MemoryStream(contentBytes))
        {
            var request = new PutObjectRequest
            {
                BucketName = bucketName,
                Key = key,
                InputStream = memoryStream,
                ContentType = "text/plain; charset=utf-8"
            };
            
            // 新增自訂 metadata
            if (metadata != null)
            {
                foreach (var kvp in metadata)
                {
                    request.Metadata.Add(kvp.Key, kvp.Value);
                }
            }
            
            // 新增系統 metadata
            request.Metadata.Add("upload-time", DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"));
            request.Metadata.Add("content-length", contentBytes.Length.ToString());
            request.Metadata.Add("encoding", "UTF-8");
            
            var response = await _s3Client.PutObjectAsync(request);
            Console.WriteLine($"✅ 帶 metadata 的字串上傳成功: {key}");
        }
    }
    
    // 📄 多行文字上傳
    public async Task UploadMultilineText(string bucketName, string key, string[] lines)
    {
        string content = string.Join(Environment.NewLine, lines);
        
        // 加入檔案資訊標頭
        var fileInfo = new[]
        {
            $"# 檔案: {key}",
            $"# 產生時間: {DateTime.Now:yyyy-MM-dd HH:mm:ss}",
            $"# 總行數: {lines.Length}",
            "# " + new string('=', 50),
            ""
        };
        
        string fullContent = string.Join(Environment.NewLine, fileInfo.Concat(lines));
        await UploadStringToS3(bucketName, key, fullContent);
    }
    
    // 📊 CSV 資料上傳
    public async Task UploadCsvData<T>(string bucketName, string key, IEnumerable<T> data, Dictionary<string, Func<T, object>> columnMappings)
    {
        var csvContent = new StringBuilder();
        
        // CSV 標頭
        csvContent.AppendLine(string.Join(",", columnMappings.Keys));
        
        // CSV 資料行
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
            
            csvContent.AppendLine(string.Join(",", values));
        }
        
        // 上傳 CSV
        byte[] csvBytes = Encoding.UTF8.GetBytes(csvContent.ToString());
        
        using (var memoryStream = new MemoryStream(csvBytes))
        {
            var request = new PutObjectRequest
            {
                BucketName = bucketName,
                Key = key,
                InputStream = memoryStream,
                ContentType = "text/csv; charset=utf-8"
            };
            
            await _s3Client.PutObjectAsync(request);
            Console.WriteLine($"✅ CSV 上傳成功: {key} ({csvBytes.Length} bytes)");
        }
    }
}
```

### 圖片檔案上傳

```csharp
public class S3ImageUpload
{
    private readonly AmazonS3Client _s3Client;
    
    public S3ImageUpload()
    {
        _s3Client = new AmazonS3Client();
    }
    
    // 🖼️ 從本機檔案上傳圖片
    public async Task UploadImageFromFile(string bucketName, string key, string filePath)
    {
        // 讀取檔案到 byte[]
        byte[] imageBytes = await File.ReadAllBytesAsync(filePath);
        
        // 檢測檔案類型
        string contentType = GetContentTypeFromExtension(Path.GetExtension(filePath));
        
        using (var memoryStream = new MemoryStream(imageBytes))
        {
            var request = new PutObjectRequest
            {
                BucketName = bucketName,
                Key = key,
                InputStream = memoryStream,
                ContentType = contentType
            };
            
            var response = await _s3Client.PutObjectAsync(request);
            Console.WriteLine($"✅ 圖片上傳成功: {key} ({imageBytes.Length} bytes)");
        }
    }
    
    // 🎨 動態產生圖片並上傳
    public async Task UploadGeneratedImage(string bucketName, string key, int width, int height, Color backgroundColor, string text)
    {
        using (var bitmap = new Bitmap(width, height))
        using (var graphics = Graphics.FromImage(bitmap))
        {
            // 繪製背景
            graphics.Clear(backgroundColor);
            
            // 繪製文字
            using (var font = new Font("Arial", 24, FontStyle.Bold))
            using (var brush = new SolidBrush(Color.White))
            {
                var textSize = graphics.MeasureString(text, font);
                var x = (width - textSize.Width) / 2;
                var y = (height - textSize.Height) / 2;
                graphics.DrawString(text, font, brush, x, y);
            }
            
            // 將圖片轉為 byte[]
            using (var tempStream = new MemoryStream())
            {
                bitmap.Save(tempStream, ImageFormat.Png);
                byte[] imageBytes = tempStream.ToArray();
                
                // 上傳到 S3
                using (var uploadStream = new MemoryStream(imageBytes))
                {
                    var request = new PutObjectRequest
                    {
                        BucketName = bucketName,
                        Key = key,
                        InputStream = uploadStream,
                        ContentType = "image/png"
                    };
                    
                    await _s3Client.PutObjectAsync(request);
                    Console.WriteLine($"✅ 動態圖片上傳成功: {key}");
                }
            }
        }
    }
    
    // 🔄 圖片格式轉換並上傳
    public async Task ConvertAndUploadImage(string bucketName, string key, byte[] originalImageBytes, ImageFormat targetFormat)
    {
        using (var originalStream = new MemoryStream(originalImageBytes))
        using (var originalImage = Image.FromStream(originalStream))
        using (var convertedStream = new MemoryStream())
        {
            // 轉換格式
            originalImage.Save(convertedStream, targetFormat);
            byte[] convertedBytes = convertedStream.ToArray();
            
            // 上傳轉換後的圖片
            using (var uploadStream = new MemoryStream(convertedBytes))
            {
                string contentType = GetContentTypeFromFormat(targetFormat);
                
                var request = new PutObjectRequest
                {
                    BucketName = bucketName,
                    Key = key,
                    InputStream = uploadStream,
                    ContentType = contentType
                };
                
                await _s3Client.PutObjectAsync(request);
                Console.WriteLine($"✅ 轉換後圖片上傳成功: {key} ({convertedBytes.Length} bytes)");
            }
        }
    }
    
    private string GetContentTypeFromExtension(string extension)
    {
        return extension.ToLower() switch
        {
            ".jpg" or ".jpeg" => "image/jpeg",
            ".png" => "image/png",
            ".gif" => "image/gif",
            ".bmp" => "image/bmp",
            ".webp" => "image/webp",
            _ => "application/octet-stream"
        };
    }
    
    private string GetContentTypeFromFormat(ImageFormat format)
    {
        if (format.Equals(ImageFormat.Jpeg)) return "image/jpeg";
        if (format.Equals(ImageFormat.Png)) return "image/png";
        if (format.Equals(ImageFormat.Gif)) return "image/gif";
        if (format.Equals(ImageFormat.Bmp)) return "image/bmp";
        return "application/octet-stream";
    }
}
```

### JSON 資料上傳

```csharp
public class S3JsonUpload
{
    private readonly AmazonS3Client _s3Client;
    
    public S3JsonUpload()
    {
        _s3Client = new AmazonS3Client();
    }
    
    // 📋 物件序列化上傳
    public async Task UploadObjectAsJson<T>(string bucketName, string key, T data, bool prettyPrint = true)
    {
        var options = new JsonSerializerOptions
        {
            WriteIndented = prettyPrint,
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping // 支援中文等字元
        };
        
        string jsonString = JsonSerializer.Serialize(data, options);
        byte[] jsonBytes = Encoding.UTF8.GetBytes(jsonString);
        
        using (var memoryStream = new MemoryStream(jsonBytes))
        {
            var request = new PutObjectRequest
            {
                BucketName = bucketName,
                Key = key,
                InputStream = memoryStream,
                ContentType = "application/json; charset=utf-8"
            };
            
            // 新增 JSON 相關 metadata
            request.Metadata.Add("data-type", typeof(T).Name);
            request.Metadata.Add("serialization-time", DateTime.UtcNow.ToString("O"));
            request.Metadata.Add("pretty-print", prettyPrint.ToString());
            
            var response = await _s3Client.PutObjectAsync(request);
            Console.WriteLine($"✅ JSON 上傳成功: {key} ({jsonBytes.Length} bytes)");
        }
    }
    
    // 📊 複雜資料結構上傳
    public async Task UploadComplexData(string bucketName, string keyPrefix)
    {
        // 模擬複雜的業務資料
        var salesData = new
        {
            ReportId = Guid.NewGuid(),
            GeneratedAt = DateTime.UtcNow,
            Period = new { Start = DateTime.Today.AddDays(-30), End = DateTime.Today },
            Summary = new
            {
                TotalSales = 1250000,
                TransactionCount = 3420,
                AverageOrderValue = 365.2m,
                TopProducts = new[]
                {
                    new { Name = "筆記型電腦", Sales = 450000 },
                    new { Name = "智慧型手機", Sales = 380000 },
                    new { Name = "平板電腦", Sales = 220000 }
                }
            },
            Details = Enumerable.Range(1, 100).Select(i => new
            {
                TransactionId = $"TXN-{i:D6}",
                Date = DateTime.Today.AddDays(-Random.Shared.Next(30)),
                Amount = Random.Shared.Next(100, 2000),
                Product = $"產品 {i % 10 + 1}"
            }).ToArray()
        };
        
        // 上傳完整資料
        await UploadObjectAsJson(bucketName, $"{keyPrefix}/full-report.json", salesData);
        
        // 上傳摘要資料
        await UploadObjectAsJson(bucketName, $"{keyPrefix}/summary.json", salesData.Summary);
        
        // 上傳詳細資料（分頁）
        const int pageSize = 25;
        var pages = salesData.Details
            .Select((item, index) => new { item, index })
            .GroupBy(x => x.index / pageSize)
            .Select(g => g.Select(x => x.item).ToArray());
        
        int pageNumber = 1;
        foreach (var page in pages)
        {
            var pageData = new
            {
                Page = pageNumber,
                TotalPages = (int)Math.Ceiling((double)salesData.Details.Length / pageSize),
                Items = page
            };
            
            await UploadObjectAsJson(bucketName, $"{keyPrefix}/details-page-{pageNumber}.json", pageData);
            pageNumber++;
        }
        
        Console.WriteLine($"✅ 複雜資料結構上傳完成，共 {pageNumber} 個檔案");
    }
    
    // 📈 即時資料流上傳
    public async Task UploadStreamingData(string bucketName, string keyPrefix, IAsyncEnumerable<object> dataStream)
    {
        var batch = new List<object>();
        const int batchSize = 100;
        int batchNumber = 1;
        
        await foreach (var item in dataStream)
        {
            batch.Add(item);
            
            if (batch.Count >= batchSize)
            {
                var batchData = new
                {
                    BatchNumber = batchNumber,
                    Timestamp = DateTime.UtcNow,
                    Items = batch.ToArray()
                };
                
                string key = $"{keyPrefix}/batch-{batchNumber:D6}.json";
                await UploadObjectAsJson(bucketName, key, batchData, prettyPrint: false);
                
                Console.WriteLine($"✅ 批次 {batchNumber} 上傳完成 ({batch.Count} 項目)");
                
                batch.Clear();
                batchNumber++;
            }
        }
        
        // 上傳剩餘的資料
        if (batch.Count > 0)
        {
            var finalBatch = new
            {
                BatchNumber = batchNumber,
                Timestamp = DateTime.UtcNow,
                Items = batch.ToArray(),
                IsFinalBatch = true
            };
            
            string finalKey = $"{keyPrefix}/batch-{batchNumber:D6}.json";
            await UploadObjectAsJson(bucketName, finalKey, finalBatch, prettyPrint: false);
            
            Console.WriteLine($"✅ 最終批次上傳完成 ({batch.Count} 項目)");
        }
    }
}
```

---

## 實務應用篇

### 動態內容產生

```csharp
public class DynamicContentGeneration
{
    // 📄 動態產生 HTML 報表
    public static async Task<MemoryStream> GenerateHtmlReport(object reportData)
    {
        var html = new StringBuilder();
        
        html.AppendLine("<!DOCTYPE html>");
        html.AppendLine("<html lang='zh-TW'>");
        html.AppendLine("<head>");
        html.AppendLine("    <meta charset='UTF-8'>");
        html.AppendLine("    <title>動態報表</title>");
        html.AppendLine("    <style>");
        html.AppendLine("        body { font-family: Arial, sans-serif; margin: 20px; }");
        html.AppendLine("        table { border-collapse: collapse; width: 100%; }");
        html.AppendLine("        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }");
        html.AppendLine("        th { background-color: #f2f2f2; }");
        html.AppendLine("    </style>");
        html.AppendLine("</head>");
        html.AppendLine("<body>");
        html.AppendLine($"    <h1>系統報表 - {DateTime.Now:yyyy-MM-dd}</h1>");
        
        // 動態插入報表內容
        html.AppendLine("    <div id='report-content'>");
        html.AppendLine(GenerateReportContent(reportData));
        html.AppendLine("    </div>");
        
        html.AppendLine("</body>");
        html.AppendLine("</html>");
        
        byte[] htmlBytes = Encoding.UTF8.GetBytes(html.ToString());
        return new MemoryStream(htmlBytes);
    }
    
    private static string GenerateReportContent(object data)
    {
        // 使用反射動態產生表格
        var properties = data.GetType().GetProperties();
        var content = new StringBuilder();
        
        content.AppendLine("    <table>");
        content.AppendLine("        <thead>");
        content.AppendLine("            <tr>");
        
        foreach (var prop in properties)
        {
            content.AppendLine($"                <th>{prop.Name}</th>");
        }
        
        content.AppendLine("            </tr>");
        content.AppendLine("        </thead>");
        content.AppendLine("        <tbody>");
        content.AppendLine("            <tr>");
        
        foreach (var prop in properties)
        {
            var value = prop.GetValue(data);
            content.AppendLine($"                <td>{value}</td>");
        }
        
        content.AppendLine("            </tr>");
        content.AppendLine("        </tbody>");
        content.AppendLine("    </table>");
        
        return content.ToString();
    }
    
    // 📊 動態產生 XML 設定檔
    public static async Task<MemoryStream> GenerateXmlConfig(Dictionary<string, object> settings)
    {
        var xml = new StringBuilder();
        xml.AppendLine("<?xml version='1.0' encoding='UTF-8'?>");
        xml.AppendLine("<configuration>");
        xml.AppendLine($"    <generated>{DateTime.UtcNow:yyyy-MM-ddTHH:mm:ssZ}</generated>");
        xml.AppendLine("    <settings>");
        
        foreach (var setting in settings)
        {
            var value = setting.Value?.ToString() ?? "";
            xml.AppendLine($"        <{setting.Key}>{SecurityElement.Escape(value)}</{setting.Key}>");
        }
        
        xml.AppendLine("    </settings>");
        xml.AppendLine("</configuration>");
        
        byte[] xmlBytes = Encoding.UTF8.GetBytes(xml.ToString());
        return new MemoryStream(xmlBytes);
    }
    
    // 🎵 動態產生 CSS 樣式表
    public static MemoryStream GenerateDynamicCss(string primaryColor, string secondaryColor, int fontSize)
    {
        var css = $@"
/* 動態產生的 CSS - {DateTime.Now:yyyy-MM-dd HH:mm:ss} */

:root {{
    --primary-color: {primaryColor};
    --secondary-color: {secondaryColor};
    --base-font-size: {fontSize}px;
}}

body {{
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    font-size: var(--base-font-size);
    color: #333;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}}

.header {{
    background-color: var(--primary-color);
    color: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
}}

.content {{
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}}

.btn-primary {{
    background-color: var(--primary-color);
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
}}

.btn-secondary {{
    background-color: var(--secondary-color);
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
}}
";
        
        byte[] cssBytes = Encoding.UTF8.GetBytes(css);
        return new MemoryStream(cssBytes);
    }
}
```

### 檔案格式轉換

```csharp
public class FileFormatConversion
{
    // 📄 JSON 轉 CSV
    public static async Task<MemoryStream> JsonToCsv(string jsonData)
    {
        using var jsonDoc = JsonDocument.Parse(jsonData);
        var root = jsonDoc.RootElement;
        
        if (root.ValueKind != JsonValueKind.Array)
        {
            throw new ArgumentException("JSON 必須是陣列格式");
        }
        
        var csv = new StringBuilder();
        var headers = new HashSet<string>();
        var rows = new List<Dictionary<string, object>>();
        
        // 收集所有可能的欄位名稱
        foreach (var item in root.EnumerateArray())
        {
            var row = new Dictionary<string, object>();
            
            foreach (var property in item.EnumerateObject())
            {
                headers.Add(property.Name);
                row[property.Name] = property.Value.ToString();
            }
            
            rows.Add(row);
        }
        
        // 寫入 CSV 標頭
        csv.AppendLine(string.Join(",", headers.Select(h => $"\"{h}\"")));
        
        // 寫入資料行
        foreach (var row in rows)
        {
            var values = headers.Select(header => 
            {
                var value = row.ContainsKey(header) ? row[header]?.ToString() ?? "" : "";
                return $"\"{value.Replace("\"", "\"\"")}\"";
            });
            
            csv.AppendLine(string.Join(",", values));
        }
        
        byte[] csvBytes = Encoding.UTF8.GetBytes(csv.ToString());
        return new MemoryStream(csvBytes);
    }
    
    // 📊 CSV 轉 JSON
    public static async Task<MemoryStream> CsvToJson(Stream csvStream)
    {
        using var reader = new StreamReader(csvStream, Encoding.UTF8);
        
        var lines = new List<string>();
        string line;
        while ((line = await reader.ReadLineAsync()) != null)
        {
            lines.Add(line);
        }
        
        if (lines.Count == 0)
        {
            throw new ArgumentException("CSV 檔案為空");
        }
        
        // 解析標頭
        var headers = ParseCsvLine(lines[0]);
        var jsonArray = new List<Dictionary<string, string>>();
        
        // 解析資料行
        for (int i = 1; i < lines.Count; i++)
        {
            var values = ParseCsvLine(lines[i]);
            var jsonObject = new Dictionary<string, string>();
            
            for (int j = 0; j < Math.Min(headers.Length, values.Length); j++)
            {
                jsonObject[headers[j]] = values[j];
            }
            
            jsonArray.Add(jsonObject);
        }
        
        var options = new JsonSerializerOptions
        {
            WriteIndented = true,
            Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping
        };
        
        string jsonString = JsonSerializer.Serialize(jsonArray, options);
        byte[] jsonBytes = Encoding.UTF8.GetBytes(jsonString);
        return new MemoryStream(jsonBytes);
    }
    
    private static string[] ParseCsvLine(string line)
    {
        var result = new List<string>();
        var current = new StringBuilder();
        bool inQuotes = false;
        
        for (int i = 0; i < line.Length; i++)
        {
            char c = line[i];
            
            if (c == '"')
            {
                if (inQuotes && i + 1 < line.Length && line[i + 1] == '"')
                {
                    current.Append('"');
                    i++; // 跳過下一個引號
                }
                else
                {
                    inQuotes = !inQuotes;
                }
            }
            else if (c == ',' && !inQuotes)
            {
                result.Add(current.ToString());
                current.Clear();
            }
            else
            {
                current.Append(c);
            }
        }
        
        result.Add(current.ToString());
        return result.ToArray();
    }
    
    // 🔄 Base64 編碼/解碼
    public static MemoryStream EncodeToBase64(byte[] binaryData)
    {
        string base64String = Convert.ToBase64String(binaryData);
        byte[] base64Bytes = Encoding.UTF8.GetBytes(base64String);
        return new MemoryStream(base64Bytes);
    }
    
    public static MemoryStream DecodeFromBase64(string base64String)
    {
        byte[] binaryData = Convert.FromBase64String(base64String);
        return new MemoryStream(binaryData);
    }
}
```

### 壓縮與解壓縮

```csharp
public class CompressionOperations
{
    // 🗜️ GZip 壓縮
    public static async Task<MemoryStream> CompressWithGZip(byte[] data)
    {
        using var originalStream = new MemoryStream(data);
        var compressedStream = new MemoryStream();
        
        using (var gzipStream = new GZipStream(compressedStream, CompressionMode.Compress, true))
        {
            await originalStream.CopyToAsync(gzipStream);
        }
        
        compressedStream.Position = 0;
        return compressedStream;
    }
    
    public static async Task<MemoryStream> DecompressGZip(byte[] compressedData)
    {
        using var compressedStream = new MemoryStream(compressedData);
        var decompressedStream = new MemoryStream();
        
        using (var gzipStream = new GZipStream(compressedStream, CompressionMode.Decompress))
        {
            await gzipStream.CopyToAsync(decompressedStream);
        }
        
        decompressedStream.Position = 0;
        return decompressedStream;
    }
    
    // 📊 壓縮效果分析
    public static async Task<CompressionResult> AnalyzeCompression(string text)
    {
        byte[] originalBytes = Encoding.UTF8.GetBytes(text);
        
        // GZip 壓縮
        using var gzipCompressed = await CompressWithGZip(originalBytes);
        byte[] gzipBytes = gzipCompressed.ToArray();
        
        // Deflate 壓縮
        using var deflateCompressed = await CompressWithDeflate(originalBytes);
        byte[] deflateBytes = deflateCompressed.ToArray();
        
        return new CompressionResult
        {
            OriginalSize = originalBytes.Length,
            GZipSize = gzipBytes.Length,
            DeflateSize = deflateBytes.Length,
            GZipRatio = (double)gzipBytes.Length / originalBytes.Length,
            DeflateRatio = (double)deflateBytes.Length / originalBytes.Length
        };
    }
    
    private static async Task<MemoryStream> CompressWithDeflate(byte[] data)
    {
        using var originalStream = new MemoryStream(data);
        var compressedStream = new MemoryStream();
        
        using (var deflateStream = new DeflateStream(compressedStream, CompressionMode.Compress, true))
        {
            await originalStream.CopyToAsync(deflateStream);
        }
        
        compressedStream.Position = 0;
        return compressedStream;
    }
    
    // 📦 批量檔案壓縮成 ZIP
    public static async Task<MemoryStream> CreateZipArchive(Dictionary<string, byte[]> files)
    {
        var zipStream = new MemoryStream();
        
        using (var archive = new ZipArchive(zipStream, ZipArchiveMode.Create, true))
        {
            foreach (var file in files)
            {
                var entry = archive.CreateEntry(file.Key);
                
                using var entryStream = entry.Open();
                await entryStream.WriteAsync(file.Value, 0, file.Value.Length);
            }
        }
        
        zipStream.Position = 0;
        return zipStream;
    }
}

public class CompressionResult
{
    public int OriginalSize { get; set; }
    public int GZipSize { get; set; }
    public int DeflateSize { get; set; }
    public double GZipRatio { get; set; }
    public double DeflateRatio { get; set; }
    
    public override string ToString()
    {
        return $"原始: {OriginalSize} bytes, " +
               $"GZip: {GZipSize} bytes ({GZipRatio:P1}), " +
               $"Deflate: {DeflateSize} bytes ({DeflateRatio:P1})";
    }
}
```

### 加密與解密

```csharp
public class EncryptionOperations
{
    // 🔐 AES 加密
    public static async Task<MemoryStream> EncryptAES(byte[] data, string password)
    {
        using var aes = Aes.Create();
        
        // 從密碼產生金鑰
        var key = new Rfc2898DeriveBytes(password, new byte[] { 0x49, 0x76, 0x61, 0x6e, 0x20, 0x4d, 0x65, 0x64, 0x76, 0x65, 0x64, 0x65, 0x76 }, 10000);
        aes.Key = key.GetBytes(32);
        aes.IV = key.GetBytes(16);
        
        var encryptedStream = new MemoryStream();
        
        // 先寫入 IV
        await encryptedStream.WriteAsync(aes.IV, 0, aes.IV.Length);
        
        // 加密資料
        using (var cryptoStream = new CryptoStream(encryptedStream, aes.CreateEncryptor(), CryptoStreamMode.Write))
        {
            await cryptoStream.WriteAsync(data, 0, data.Length);
        }
        
        encryptedStream.Position = 0;
        return encryptedStream;
    }
    
    public static async Task<MemoryStream> DecryptAES(byte[] encryptedData, string password)
    {
        using var aes = Aes.Create();
        using var encryptedStream = new MemoryStream(encryptedData);
        
        // 讀取 IV
        var iv = new byte[16];
        await encryptedStream.ReadAsync(iv, 0, 16);
        
        // 從密碼產生金鑰
        var key = new Rfc2898DeriveBytes(password, new byte[] { 0x49, 0x76, 0x61, 0x6e, 0x20, 0x4d, 0x65, 0x64, 0x76, 0x65, 0x64, 0x65, 0x76 }, 10000);
        aes.Key = key.GetBytes(32);
        aes.IV = iv;
        
        var decryptedStream = new MemoryStream();
        
        using (var cryptoStream = new CryptoStream(encryptedStream, aes.CreateDecryptor(), CryptoStreamMode.Read))
        {
            await cryptoStream.CopyToAsync(decryptedStream);
        }
        
        decryptedStream.Position = 0;
        return decryptedStream;
    }
    
    // 🔑 RSA 加密（適合小量資料）
    public static MemoryStream EncryptRSA(byte[] data, string publicKeyXml)
    {
        using var rsa = RSA.Create();
        rsa.FromXmlString(publicKeyXml);
        
        byte[] encryptedData = rsa.Encrypt(data, RSAEncryptionPadding.OaepSHA256);
        return new MemoryStream(encryptedData);
    }
    
    public static MemoryStream DecryptRSA(byte[] encryptedData, string privateKeyXml)
    {
        using var rsa = RSA.Create();
        rsa.FromXmlString(privateKeyXml);
        
        byte[] decryptedData = rsa.Decrypt(encryptedData, RSAEncryptionPadding.OaepSHA256);
        return new MemoryStream(decryptedData);
    }
    
    // 🔐 混合加密（RSA + AES）
    public static async Task<MemoryStream> HybridEncrypt(byte[] data, string rsaPublicKeyXml)
    {
        // 產生隨機 AES 金鑰
        var aesKey = new byte[32];
        var aesIV = new byte[16];
        
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(aesKey);
            rng.GetBytes(aesIV);
        }
        
        // 用 AES 加密資料
        using var aes = Aes.Create();
        aes.Key = aesKey;
        aes.IV = aesIV;
        
        var encryptedDataStream = new MemoryStream();
        using (var cryptoStream = new CryptoStream(encryptedDataStream, aes.CreateEncryptor(), CryptoStreamMode.Write))
        {
            await cryptoStream.WriteAsync(data, 0, data.Length);
        }
        
        // 用 RSA 加密 AES 金鑰
        using var rsa = RSA.Create();
        rsa.FromXmlString(rsaPublicKeyXml);
        
        var keyAndIV = new byte[aesKey.Length + aesIV.Length];
        Buffer.BlockCopy(aesKey, 0, keyAndIV, 0, aesKey.Length);
        Buffer.BlockCopy(aesIV, 0, keyAndIV, aesKey.Length, aesIV.Length);
        
        byte[] encryptedKey = rsa.Encrypt(keyAndIV, RSAEncryptionPadding.OaepSHA256);
        
        // 組合最終結果
        var result = new MemoryStream();
        
        // 寫入加密金鑰長度
        var keyLengthBytes = BitConverter.GetBytes(encryptedKey.Length);
        await result.WriteAsync(keyLengthBytes, 0, keyLengthBytes.Length);
        
        // 寫入加密金鑰
        await result.WriteAsync(encryptedKey, 0, encryptedKey.Length);
        
        // 寫入加密資料
        var encryptedDataBytes = encryptedDataStream.ToArray();
        await result.WriteAsync(encryptedDataBytes, 0, encryptedDataBytes.Length);
        
        result.Position = 0;
        return result;
    }
}
```

---

## 效能優化篇

### 記憶體管理

```csharp
public class MemoryManagement
{
    // 🧠 記憶體使用監控
    public static void MonitorMemoryUsage(Action<MemoryStream> operation)
    {
        var initialMemory = GC.GetTotalMemory(false);
        Console.WriteLine($"初始記憶體: {initialMemory:N0} bytes");
        
        using (var memoryStream = new MemoryStream())
        {
            var beforeOperation = GC.GetTotalMemory(false);
            Console.WriteLine($"建立 MemoryStream 後: {beforeOperation:N0} bytes (+{beforeOperation - initialMemory:N0})");
            
            operation(memoryStream);
            
            var afterOperation = GC.GetTotalMemory(false);
            Console.WriteLine($"操作完成後: {afterOperation:N0} bytes (+{afterOperation - beforeOperation:N0})");
            Console.WriteLine($"MemoryStream 大小: {memoryStream.Length:N0} bytes");
            Console.WriteLine($"MemoryStream 容量: {memoryStream.Capacity:N0} bytes");
        }
        
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        
        var finalMemory = GC.GetTotalMemory(false);
        Console.WriteLine($"GC 後記憶體: {finalMemory:N0} bytes");
        Console.WriteLine($"記憶體變化: {finalMemory - initialMemory:N0} bytes");
    }
    
    // 📈 容量管理最佳化
    public static void OptimalCapacityManagement()
    {
        Console.WriteLine("=== 容量管理比較 ===");
        
        // ❌ 不佳做法：讓 MemoryStream 自動擴展
        var sw = Stopwatch.StartNew();
        using (var badStream = new MemoryStream())
        {
            for (int i = 0; i < 10000; i++)
            {
                byte[] data = Encoding.UTF8.GetBytes($"資料行 {i} - 這是一些測試資料\n");
                badStream.Write(data, 0, data.Length);
            }
        }
        sw.Stop();
        Console.WriteLine($"自動擴展: {sw.ElapsedMilliseconds} ms");
        
        // ✅ 良好做法：預先配置適當容量
        sw.Restart();
        using (var goodStream = new MemoryStream(1024 * 1024)) // 1MB 初始容量
        {
            for (int i = 0; i < 10000; i++)
            {
                byte[] data = Encoding.UTF8.GetBytes($"資料行 {i} - 這是一些測試資料\n");
                goodStream.Write(data, 0, data.Length);
            }
        }
        sw.Stop();
        Console.WriteLine($"預配置容量: {sw.ElapsedMilliseconds} ms");
    }
    
    // 🔄 重複使用 MemoryStream
    public static void ReuseMemoryStream()
    {
        using (var reusableStream = new MemoryStream())
        {
            for (int iteration = 1; iteration <= 5; iteration++)
            {
                // 清空但保留容量
                reusableStream.SetLength(0);
                reusableStream.Position = 0;
                
                // 寫入新資料
                string data = $"第 {iteration} 次迭代的資料：{string.Join(",", Enumerable.Range(1, 100))}";
                byte[] dataBytes = Encoding.UTF8.GetBytes(data);
                reusableStream.Write(dataBytes, 0, dataBytes.Length);
                
                Console.WriteLine($"迭代 {iteration}: 長度 {reusableStream.Length}, 容量 {reusableStream.Capacity}");
            }
        }
    }
}
```

### 大檔案處理策略

```csharp
public class LargeFileHandling
{
    // 📊 分塊處理大檔案
    public static async Task ProcessLargeFileInChunks(byte[] largeData, int chunkSize = 1024 * 1024) // 1MB chunks
    {
        using var sourceStream = new MemoryStream(largeData);
        var buffer = new byte[chunkSize];
        int chunkNumber = 1;
        
        while (sourceStream.Position < sourceStream.Length)
        {
            int bytesRead = await sourceStream.ReadAsync(buffer, 0, chunkSize);
            
            if (bytesRead > 0)
            {
                // 處理這個區塊
                await ProcessChunk(buffer, bytesRead, chunkNumber);
                chunkNumber++;
            }
        }
        
        Console.WriteLine($"✅ 大檔案處理完成，共 {chunkNumber - 1} 個區塊");
    }
    
    private static async Task ProcessChunk(byte[] buffer, int length, int chunkNumber)
    {
        // 模擬區塊處理 (例如：上傳到雲端、壓縮、加密等)
        using var chunkStream = new MemoryStream(buffer, 0, length);
        
        // 這裡可以進行各種處理
        Console.WriteLine($"處理區塊 {chunkNumber}: {length:N0} bytes");
        
        // 模擬處理時間
        await Task.Delay(10);
    }
    
    // 🔄 串流管線處理
    public static async Task StreamPipelineProcessing(byte[] sourceData)
    {
        using var sourceStream = new MemoryStream(sourceData);
        using var processedStream = new MemoryStream();
        
        // 建立處理管線
        await ProcessingPipeline(sourceStream, processedStream);
        
        Console.WriteLine($"管線處理完成: {sourceStream.Length:N0} → {processedStream.Length:N0} bytes");
    }
    
    private static async Task ProcessingPipeline(Stream input, Stream output)
    {
        // 第一階段：壓縮
        using var compressedStream = new MemoryStream();
        using (var gzipStream = new GZipStream(compressedStream, CompressionMode.Compress, true))
        {
            await input.CopyToAsync(gzipStream);
        }
        
        // 第二階段：Base64 編碼
        compressedStream.Position = 0;
        var compressedBytes = compressedStream.ToArray();
        string base64String = Convert.ToBase64String(compressedBytes);
        
        // 第三階段：加上標頭和結尾
        var finalContent = $"-----BEGIN COMPRESSED DATA-----\n{base64String}\n-----END COMPRESSED DATA-----";
        byte[] finalBytes = Encoding.UTF8.GetBytes(finalContent);
        
        await output.WriteAsync(finalBytes, 0, finalBytes.Length);
    }
    
    // ⚡ 並行處理多個資料流
    public static async Task ParallelStreamProcessing(IEnumerable<byte[]> dataSources)
    {
        var tasks = dataSources.Select(async (data, index) =>
        {
            using var stream = new MemoryStream(data);
            
            // 模擬一些處理工作
            var processedData = await ProcessStreamAsync(stream, $"Stream-{index}");
            
            return new { Index = index, ProcessedSize = processedData.Length };
        });
        
        var results = await Task.WhenAll(tasks);
        
        foreach (var result in results)
        {
            Console.WriteLine($"串流 {result.Index}: 處理後大小 {result.ProcessedSize:N0} bytes");
        }
    }
    
    private static async Task<byte[]> ProcessStreamAsync(MemoryStream stream, string name)
    {
        // 模擬非同步處理
        await Task.Delay(Random.Shared.Next(50, 200));
        
        // 簡單的處理：反轉資料
        var data = stream.ToArray();
        Array.Reverse(data);
        
        Console.WriteLine($"✅ {name} 處理完成");
        return data;
    }
}
```

### 池化技術

```csharp
public class MemoryStreamPooling
{
    private static readonly ObjectPool<MemoryStream> _memoryStreamPool;
    
    static MemoryStreamPooling()
    {
        var provider = new DefaultObjectPoolProvider();
        _memoryStreamPool = provider.Create(new MemoryStreamPooledObjectPolicy());
    }
    
    // 🏊‍♂️ 使用物件池
    public static async Task<string> ProcessWithPool(string input)
    {
        var stream = _memoryStreamPool.Get();
        try
        {
            // 重設串流
            stream.SetLength(0);
            stream.Position = 0;
            
            // 使用串流進行處理
            byte[] inputBytes = Encoding.UTF8.GetBytes(input);
            await stream.WriteAsync(inputBytes, 0, inputBytes.Length);
            
            // 模擬一些處理
            stream.Position = 0;
            using var reader = new StreamReader(stream, Encoding.UTF8);
            string content = await reader.ReadToEndAsync();
            
            return content.ToUpper(); // 簡單的處理範例
        }
        finally
        {
            _memoryStreamPool.Return(stream);
        }
    }
    
    // 📊 效能比較：池化 vs 非池化
    public static async Task ComparePoolingPerformance()
    {
        const int iterations = 10000;
        var testData = "這是一些測試資料，用來比較效能差異。" + string.Join("", Enumerable.Range(1, 100));
        
        // 測試 1: 不使用池化
        var sw = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            using (var stream = new MemoryStream())
            {
                byte[] data = Encoding.UTF8.GetBytes(testData);
                await stream.WriteAsync(data, 0, data.Length);
                var result = stream.ToArray();
            }
        }
        sw.Stop();
        Console.WriteLine($"不使用池化: {sw.ElapsedMilliseconds} ms");
        
        // 測試 2: 使用池化
        sw.Restart();
        for (int i = 0; i < iterations; i++)
        {
            await ProcessWithPool(testData);
        }
        sw.Stop();
        Console.WriteLine($"使用池化: {sw.ElapsedMilliseconds} ms");
        
        // 測試記憶體使用
        var beforeGC = GC.GetTotalMemory(false);
        
        // 觸發一些物件建立
        for (int i = 0; i < 1000; i++)
        {
            await ProcessWithPool(testData);
        }
        
        var afterOperations = GC.GetTotalMemory(false);
        
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        
        var afterGC = GC.GetTotalMemory(false);
        
        Console.WriteLine($"操作前記憶體: {beforeGC:N0} bytes");
        Console.WriteLine($"操作後記憶體: {afterOperations:N0} bytes");
        Console.WriteLine($"GC 後記憶體: {afterGC:N0} bytes");
    }
}

// 自訂池化策略
public class MemoryStreamPooledObjectPolicy : IPooledObjectPolicy<MemoryStream>
{
    public MemoryStream Create()
    {
        return new MemoryStream();
    }
    
    public bool Return(MemoryStream obj)
    {
        // 檢查物件是否適合重複使用
        if (obj.Capacity > 1024 * 1024) // 如果容量超過 1MB，不要重複使用
        {
            return false;
        }
        
        // 重設狀態
        obj.SetLength(0);
        obj.Position = 0;
        
        return true;
    }
}
```

---

## 最佳實踐與常見陷阱

### 資源管理

```csharp
public class ResourceManagement
{
    // ✅ 正確的資源管理
    public static async Task CorrectResourceManagement()
    {
        // 使用 using 語句確保資源釋放
        using (var memoryStream = new MemoryStream())
        {
            await WriteDataToStream(memoryStream, "測試資料");
            
            // 做一些處理...
            var result = ProcessStream(memoryStream);
            Console.WriteLine($"處理結果: {result}");
        } // MemoryStream 會自動 Dispose
    }
    
    // ❌ 錯誤的資源管理範例
    public static async Task IncorrectResourceManagement()
    {
        MemoryStream memoryStream = null;
        try
        {
            memoryStream = new MemoryStream();
            await WriteDataToStream(memoryStream, "測試資料");
            
            // 如果這裡發生例外...
            throw new InvalidOperationException("模擬錯誤");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"發生錯誤: {ex.Message}");
            // ❌ memoryStream 可能不會被正確釋放
        }
        finally
        {
            // ✅ 至少在 finally 中釋放
            memoryStream?.Dispose();
        }
    }
    
    // 🔄 正確處理多個 MemoryStream
    public static async Task HandleMultipleStreams()
    {
        using var stream1 = new MemoryStream();
        using var stream2 = new MemoryStream();
        using var stream3 = new MemoryStream();
        
        // 並行處理多個串流
        var tasks = new[]
        {
            WriteDataToStream(stream1, "資料1"),
            WriteDataToStream(stream2, "資料2"),
            WriteDataToStream(stream3, "資料3")
        };
        
        await Task.WhenAll(tasks);
        
        Console.WriteLine($"串流大小: {stream1.Length}, {stream2.Length}, {stream3.Length}");
    }
    
    private static async Task WriteDataToStream(MemoryStream stream, string data)
    {
        byte[] bytes = Encoding.UTF8.GetBytes(data);
        await stream.WriteAsync(bytes, 0, bytes.Length);
    }
    
    private static string ProcessStream(MemoryStream stream)
    {
        stream.Position = 0;
        using var reader = new StreamReader(stream, Encoding.UTF8);
        return reader.ReadToEnd();
    }
}
```

### 效能陷阱

```csharp
public class PerformancePitfalls
{
    // ❌ 陷阱 1: 頻繁的 ToArray() 呼叫
    public static void AvoidFrequentToArray()
    {
        using var stream = new MemoryStream();
        
        for (int i = 0; i < 1000; i++)
        {
            string data = $"資料行 {i}";
            byte[] bytes = Encoding.UTF8.GetBytes(data);
            stream.Write(bytes, 0, bytes.Length);
            
            // ❌ 每次迴圈都呼叫 ToArray() 會建立新陣列
            byte[] currentData = stream.ToArray();
            Console.WriteLine($"目前大小: {currentData.Length}");
        }
        
        // ✅ 只在最後呼叫一次
        byte[] finalData = stream.ToArray();
        Console.WriteLine($"最終大小: {finalData.Length}");
    }
    
    // ❌ 陷阱 2: 忘記重設 Position
    public static async Task AvoidPositionIssues()
    {
        using var stream = new MemoryStream();
        
        // 寫入資料
        string testData = "測試資料";
        byte[] bytes = Encoding.UTF8.GetBytes(testData);
        await stream.WriteAsync(bytes, 0, bytes.Length);
        
        // ❌ 忘記重設位置
        using (var reader = new StreamReader(stream, Encoding.UTF8))
        {
            string result1 = await reader.ReadToEndAsync();
            Console.WriteLine($"第一次讀取: '{result1}'"); // 可能是空的
        }
        
        // ✅ 正確重設位置
        stream.Position = 0;
        using (var reader = new StreamReader(stream, Encoding.UTF8))
        {
            string result2 = await reader.ReadToEndAsync();
            Console.WriteLine($"第二次讀取: '{result2}'"); // 正確讀取到資料
        }
    }
    
    // ❌ 陷阱 3: 不必要的編碼轉換
    public static void AvoidUnnecessaryEncoding()
    {
        string originalText = "測試文字";
        
        // ❌ 多次編碼轉換
        byte[] bytes1 = Encoding.UTF8.GetBytes(originalText);
        string text1 = Encoding.UTF8.GetString(bytes1);
        byte[] bytes2 = Encoding.UTF8.GetBytes(text1);
        string text2 = Encoding.UTF8.GetString(bytes2);
        
        // ✅ 直接使用原始資料
        byte[] bytes = Encoding.UTF8.GetBytes(originalText);
        using var stream = new MemoryStream(bytes);
        
        // 直接從串流讀取，避免多次轉換
        stream.Position = 0;
        using var reader = new StreamReader(stream, Encoding.UTF8);
        string result = reader.ReadToEnd();
    }
    
    // 📊 效能基準測試
    public static async Task PerformanceBenchmark()
    {
        const int iterations = 10000;
        var testData = "這是效能測試資料 " + string.Join("", Enumerable.Range(1, 100));
        
        // 測試 1: 頻繁建立新 MemoryStream
        var sw = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            using var stream = new MemoryStream();
            byte[] bytes = Encoding.UTF8.GetBytes(testData);
            await stream.WriteAsync(bytes, 0, bytes.Length);
            var result = stream.ToArray();
        }
        sw.Stop();
        Console.WriteLine($"頻繁建立: {sw.ElapsedMilliseconds} ms");
        
        // 測試 2: 重複使用 MemoryStream
        sw.Restart();
        using (var reusableStream = new MemoryStream())
        {
            for (int i = 0; i < iterations; i++)
            {
                reusableStream.SetLength(0);
                reusableStream.Position = 0;
                
                byte[] bytes = Encoding.UTF8.GetBytes(testData);
                await reusableStream.WriteAsync(bytes, 0, bytes.Length);
                var result = reusableStream.ToArray();
            }
        }
        sw.Stop();
        Console.WriteLine($"重複使用: {sw.ElapsedMilliseconds} ms");
    }
}
```

### 安全性考量

```csharp
public class SecurityConsiderations
{
    // 🔒 敏感資料處理
    public static async Task HandleSensitiveData(string sensitiveData)
    {
        // 使用 SecureString 或其他安全措施處理敏感資料
        byte[] sensitiveBytes = Encoding.UTF8.GetBytes(sensitiveData);
        
        try
        {
            using (var secureStream = new MemoryStream(sensitiveBytes))
            {
                // 處理敏感資料
                await ProcessSecureData(secureStream);
            }
        }
        finally
        {
            // 清除敏感資料
            Array.Clear(sensitiveBytes, 0, sensitiveBytes.Length);
        }
    }
    
    private static async Task ProcessSecureData(MemoryStream stream)
    {
        // 確保不會記錄敏感資料
        Console.WriteLine($"處理 {stream.Length} bytes 的敏感資料");
        
        // 進行必要的處理...
        await Task.Delay(10);
    }
    
    // 🛡️ 輸入驗證
    public static bool ValidateInputData(byte[] data, int maxSize = 10 * 1024 * 1024) // 10MB 預設限制
    {
        if (data == null)
        {
            Console.WriteLine("❌ 資料不能為 null");
            return false;
        }
        
        if (data.Length == 0)
        {
            Console.WriteLine("❌ 資料不能為空");
            return false;
        }
        
        if (data.Length > maxSize)
        {
            Console.WriteLine($"❌ 資料大小 ({data.Length:N0} bytes) 超過限制 ({maxSize:N0} bytes)");
            return false;
        }
        
        Console.WriteLine($"✅ 資料驗證通過 ({data.Length:N0} bytes)");
        return true;
    }
    
    // 🔍 安全的檔案類型檢測
    public static string DetectFileType(byte[] data)
    {
        if (data == null || data.Length < 4)
            return "unknown";
        
        // 檢查檔案簽名
        if (data.Length >= 4)
        {
            // PNG: 89 50 4E 47
            if (data[0] == 0x89 && data[1] == 0x50 && data[2] == 0x4E && data[3] == 0x47)
                return "image/png";
            
            // JPEG: FF D8 FF
            if (data[0] == 0xFF && data[1] == 0xD8 && data[2] == 0xFF)
                return "image/jpeg";
            
            // PDF: 25 50 44 46
            if (data[0] == 0x25 && data[1] == 0x50 && data[2] == 0x44 && data[3] == 0x46)
                return "application/pdf";
        }
        
        // 檢查是否為文字檔案
        if (IsTextFile(data))
            return "text/plain";
        
        return "application/octet-stream";
    }
    
    private static bool IsTextFile(byte[] data)
    {
        // 簡單的文字檔案檢測
        for (int i = 0; i < Math.Min(1024, data.Length); i++)
        {
            byte b = data[i];
            if (b == 0) return false; // 發現 null 字元，可能是二進位檔案
            if (b < 32 && b != 9 && b != 10 && b != 13) return false; // 不是可列印字元
        }
        
        return true;
    }
}
```

---

## 總結 🎉

### MemoryStream 的超能力 💪

1. **虛擬檔案系統** 💾 - 在記憶體中模擬檔案操作
2. **極速存取** ⚡ - 比實體檔案快數十倍
3. **動態大小** 📈 - 自動擴展容量
4. **完整 Stream 介面** 🔄 - 支援所有標準串流操作
5. **雲端整合完美搭檔** ☁️ - S3、Azure Blob 等服務的最佳選擇

### AWS S3 整合核心概念 🎯

> 💡 **關鍵理解**：S3 接受的是「檔案的原始內容」，不管是圖片、文字還是影片，其實本質上都是一堆 bytes！

🔄 **完美轉換鏈**：
```
string → Encoding.UTF8.GetBytes() → byte[] → MemoryStream → S3 Upload
```

### 實戰檢查清單 ✅

使用 MemoryStream 時，確保您做到了：

- ✅ 使用 `using` 語句管理資源
- ✅ 適當設定初始容量避免頻繁擴展
- ✅ 記住重設 `Position` 以便重複讀取
- ✅ 選擇正確的編碼方式（建議 UTF-8）
- ✅ 大檔案使用分塊處理策略
- ✅ 敏感資料處理後清除記憶體
- ✅ 驗證輸入資料大小和格式

### 效能優化金句 💎

> "預先配置容量，避免記憶體重新配置的效能損失！"

> "重複使用 MemoryStream，讓物件池化成為您的秘密武器！"

> "位置控制很重要，讀取前記得 Position = 0！"

> "大檔案分塊處理，小檔案直接載入，選對策略效能翻倍！"

### 記憶體管理最佳實踐 🧠

| 資料大小 | 建議策略 | 理由 |
|---------|----------|------|
| < 1MB | 直接使用 MemoryStream | 記憶體負擔小 |
| 1MB - 10MB | 考慮分塊處理 | 平衡效能與記憶體 |
| > 10MB | 必須分塊處理 | 避免記憶體不足 |
| > 100MB | 考慮使用 FileStream | 記憶體限制 |

現在您已經完全掌握了 MemoryStream 的精髓，無論是簡單的字串處理還是複雜的雲端整合，都能輕鬆應對！

### 您的核心概念再回顧 🎯

正如您所說：
> 📦 **MemoryStream 就像一個用 RAM 做的暫存小硬碟**
> 🔄 **把中文信寫好之後，要轉成信封裡的二進位檔案才能寄出**

這兩個比喻完美地詮釋了 MemoryStream 的本質和用途。現在您可以自信地處理任何需要記憶體串流的場景！🚀✨