# C# Async 非同步寫法實務記錄 ⏰

> 紀錄開發中遇到的各種 async/await 寫法與最佳實踐

## 目錄
1. [基本非同步模式](#基本非同步模式)
   - 1.1 [Task 啟動與等待](#task-啟動與等待)
   - 1.2 [檔案非同步操作](#檔案非同步操作)
   - 1.3 [時間控制與監控](#時間控制與監控)

2. [常見寫法記錄](#常見寫法記錄)
   - 2.1 [立即啟動後等待](#立即啟動後等待)
   - 2.2 [多工並行處理](#多工並行處理)
   - 2.3 [條件等待模式](#條件等待模式)

3. [實務範例集](#實務範例集)
   - 3.3 [錯誤處理策略](#錯誤處理策略)

---

## 基本非同步模式

### Task 啟動與等待

> 💡 **核心概念**：先啟動 Task，讓它在背景執行，需要結果時再 await

**基本寫法記錄**
```csharp
async void Main()
{
    var start = DateTime.Now.ToString("HH:mm:ss");
    Task task = LogAsync();  // 立即啟動，不等待
    var end = DateTime.Now.ToString("HH:mm:ss");
    $"{start} -- {end}".Dump();
    
    await task;  // 在需要的時候等待完成
    $"完成沒: {task.IsCompleted}".Dump();
}
```

**執行時序分析**
```
開始時間: 14:30:15
結束時間: 14:30:15  ← 立即返回，Task 在背景執行
... 等待 2 秒 ...
完成沒: True        ← Task 執行完畢
```

### 檔案非同步操作

**日誌寫入實作**
```csharp
public async Task LogAsync()
{
    await Task.Delay(2000);  // 模擬處理時間
    
    var path = Path.Combine(
        Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
        "log0621.txt"
    );
    
    using (var writer = new StreamWriter(path, true))
    {
        await writer.WriteLineAsync(DateTime.Now.ToString("HH:mm:ss"));
    }
}
```

### 時間控制與監控

**計時與狀態檢查**
```csharp
async Task TimeMonitoringExample()
{
    var stopwatch = Stopwatch.StartNew();
    
    Task longTask = ProcessLongOperationAsync();
    
    // 可以做其他事情
    Console.WriteLine($"Task 啟動: {stopwatch.ElapsedMilliseconds}ms");
    
    await longTask;
    Console.WriteLine($"Task 完成: {stopwatch.ElapsedMilliseconds}ms");
    Console.WriteLine($"狀態: {longTask.Status}");
}
```

---

## 常見寫法記錄

### 立即啟動後等待

> 🚀 **使用時機**：需要讓 Task 立即開始執行，但不立即等待結果

**模式一：延遲等待**
```csharp
async Task DelayedAwaitPattern()
{
    // 立即啟動
    Task dataTask = FetchDataAsync();
    Task processTask = ProcessAsync();
    
    // 做其他事情
    PrepareUI();
    
    // 需要結果時才等待
    var data = await dataTask;
    await processTask;
}
```

**模式二：條件等待**
```csharp
async Task ConditionalAwaitPattern()
{
    Task backgroundTask = BackgroundProcessAsync();
    
    if (needResult)
    {
        await backgroundTask;
    }
    // 如果不需要結果，Task 會在背景繼續執行
}
```

### 多工並行處理

**並行執行多個 Task**
```csharp
async Task ParallelProcessing()
{
    // 同時啟動多個 Task
    Task task1 = ProcessFileAsync("file1.txt");
    Task task2 = ProcessFileAsync("file2.txt");
    Task task3 = ProcessFileAsync("file3.txt");
    
    // 等待全部完成
    await Task.WhenAll(task1, task2, task3);
    
    Console.WriteLine("所有檔案處理完成");
}
```

**等待第一個完成**
```csharp
async Task FirstCompletedPattern()
{
    Task<string> server1 = CallApiAsync("server1");
    Task<string> server2 = CallApiAsync("server2");
    Task<string> server3 = CallApiAsync("server3");
    
    // 等待第一個回應
    Task<string> completed = await Task.WhenAny(server1, server2, server3);
    string result = await completed;
    
    Console.WriteLine($"最快回應: {result}");
}
```

### 條件等待模式

**帶超時的等待**
```csharp
async Task TimeoutPattern()
{
    Task longTask = LongRunningTaskAsync();
    
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    
    try
    {
        await longTask.WaitAsync(cts.Token);
        Console.WriteLine("在時限內完成");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("超時取消");
    }
}
```

### 錯誤處理策略

**帶錯誤處理的非同步模式**
```csharp
async Task SafeAsyncPattern()
{
    Task riskyTask = RiskyOperationAsync();
    
    // 做其他工作
    DoOtherWork();
    
    try
    {
        await riskyTask;
        Console.WriteLine("操作成功完成");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"操作失敗: {ex.Message}");
        // 錯誤處理邏輯
    }
}

async Task<T> SafeExecuteAsync<T>(Func<Task<T>> operation, T defaultValue)
{
    try
    {
        Task<T> task = operation();
        return await task;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"執行失敗，使用預設值: {ex.Message}");
        return defaultValue;
    }
}
```

**重試機制**
```csharp
async Task<T> RetryAsync<T>(Func<Task<T>> operation, int maxRetries = 3)
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            Task<T> task = operation();
            return await task;
        }
        catch (Exception ex) when (i < maxRetries - 1)
        {
            Console.WriteLine($"重試 {i + 1}/{maxRetries}: {ex.Message}");
            await Task.Delay(1000 * (i + 1)); // 遞增延遲
        }
    }
    
    // 最後一次嘗試，不捕捉異常
    return await operation();
}
```

---

## 常用模式速查

### 📋 **模式對照表**

| 模式 | 寫法 | 使用時機 |
|------|------|----------|
| 立即等待 | `await TaskAsync()` | 需要結果才能繼續 |
| 延遲等待 | `Task t = TaskAsync(); ... await t;` | 先啟動，後等待 |
| 並行等待 | `await Task.WhenAll(t1, t2)` | 多個 Task 同時執行 |
| 搶先等待 | `await Task.WhenAny(t1, t2)` | 等待最快完成的 |
| 超時等待 | `await task.WaitAsync(timeout)` | 避免無限等待 |

### 🎯 **實務金句**

> "Task 一建立就開始執行，await 只是等待結果！"

> "先啟動、再等待，讓 CPU 時間不浪費！"

> "非同步不等於多執行緒，但能提升應用程式回應性！"

### ✅ **檢查清單**

- [ ] Task 是否在正確時機啟動？
- [ ] 是否有遺漏的 await？
- [ ] 錯誤處理是否完善？
- [ ] 是否需要設定超時？
- [ ] 資源是否正確釋放？

這份文件記錄了實務中常見的 async/await 寫法，幫助快速回顧和應用各種非同步程式設計模式！🚀
