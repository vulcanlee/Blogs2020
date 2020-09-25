# 在執行緒內拋出例外異常，可否捕捉到

許多人在使用 C# 非同步程式設計的 TAP 模式時候，往往會聽到這樣的強調內容：當設計 非同步方法，也就是在方法前面加上 `async` 修飾詞，而該方法的回傳值需要為 `Task` 或者 `Task<T>`，千萬不要把這個非同步方法的回傳值宣告為 `void`，因為，這樣的宣告方式，僅適用於有事件訂閱的非同步委派方法內使用。

可是，往往大家在設計非同步方法的時候，都僅會在非同步方法前面加入 `async` 修飾詞，而將該非同步方法的回傳值宣告為 `void`；就是因為回傳值為 void，因此，當要呼叫這個非同步方法的時候，就僅能夠直接呼叫該非同步方法，而不能使用 `await` 關鍵字 (這是因為該非同步方法的回傳值必須是一個 Task 型別的物件，因為，await 之後，要接著一個 Task 物件，這樣才能等待)。

接著，更可怕的事情就發生了，因為這樣的呼叫方式，造成了射後不理的情況，也因為如此，也就無法使用 try...catch 敘述來捕捉這樣的非同步方法的例外異常。為了要讓大家更清楚問題在哪裡，因此，底下的程式碼將會使用執行緒來說明。

首先，在主執行緒內，故意拋出例外異常 Exception，並且使用 try ... catch 敘述來捕捉這個例外異常，這樣的寫法是標準作法，理所當然會捕捉到例外異常，而不會造成處理程序崩潰；不過，接下來使用 try...catch 將建立一個執行緒，等候三秒鐘之後，拋出例外異常，最後啟動該執行緒這樣的過程都包起來，因此，在程式執行後的三秒鐘後，並無法成功攔截捕捉到例外異常，而是造成 Process 崩潰，也就是說，在該執行緒的外面，是無法使用 try...catch 敘述來捕捉到該執行緒內所產生的任何例外異常錯誤。

```csharp
static void Main(string[] args)
{
    Console.WriteLine($"Main Thread Id :" +
       $"{Thread.CurrentThread.ManagedThreadId}");
    try
    {
        throw new Exception("Capture Main Exception");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
 
    try
    {
        Thread thread = new Thread(() =>
        {
            Console.WriteLine($"New Thread Id :" +
                $"{Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(3000);
            throw new Exception("Oh Oh");
        });
        thread.Start();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
 
 
}
```
