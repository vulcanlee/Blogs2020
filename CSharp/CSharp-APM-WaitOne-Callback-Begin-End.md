# 使用 APM 非同步設計模式加上 callback 的各個觸發時間

當在進行 APM 非同步程式設計的時候，首先會先呼叫 BeginXXX 方法，啟動非同步作業，接著，當使用 callback 方式的話，當非同步作業完成之後，就會呼叫這個 callback 委派方法；不過，此時，若想要設計出在主執行緒中，使用封鎖 Block 等待的方法，等候這個方同步作業與 callback 完成，若不使用其他執行緒同步功能，是否可以做到呢？

也就是說，在主執行緒下，呼叫 IAsyncResult.AsyncWaitHandle.WaitOne() 方法的時候，究竟會在執行 callback 的當時解除封鎖 block，還是於 callback 方法之後呢？ 為了要了解此一問題，設計了底下的程式碼來一探究竟。

首先，這裡使用 WebRequest.Create 建立起一個 HttpWebRequest 物件，而該 URL 會花費 5 秒鐘的時間，才會執行完畢，在主執行緒內，接著呼叫 myHttpWebRequest1.BeginGetResponse(callBack, myHttpWebRequest1) 方法，啟動非同步作業，這裡有宣告一個 callBack 委派方法，會於非同步作業完成之後，呼叫這個方法。最後，使用了 asyncResult.AsyncWaitHandle.WaitOne() 方法來等待非同步作業的完成。

不論在主執行緒內與委派 callback 方法內，都會使用 Console.WriteLine 方法輸出一些檢查點訊息，用來幫助確認這個問題。

```csharp
static void Main(string[] args)
{
    try
    {
        string url = $"https://hyperfullstack.azurewebsites.net/api/HandOnLab/Add/1/2/5";
 
        // 針對非同步請求，產生委派方法，用於處理非同步工作執行完成後的結果
        AsyncCallback callBack = new AsyncCallback(ResponseCallback);
 
        // 使用 WebRequest.Create 工廠方法建立一個 HttpWebrequest 物件
        HttpWebRequest myHttpWebRequest1 = (HttpWebRequest)WebRequest.Create(url);
 
        // 呼叫 BeginXXX 啟動非同步工作
        Console.WriteLine("Main 1");
        IAsyncResult asyncResult =
          (IAsyncResult)myHttpWebRequest1.BeginGetResponse(callBack, myHttpWebRequest1);
 
        Console.WriteLine("Main 2");
        asyncResult.AsyncWaitHandle.WaitOne();
        Console.WriteLine("Main 3");
 
        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine($"   處理其他事情");
            Thread.Sleep(1000);
        }
 
        // 主執行緒的工作已經完成
        Console.WriteLine("Press any key for continuing...");
        Console.ReadKey();
    }
    catch (WebException e)
    {
        Console.WriteLine("\nException raised!");
        Console.WriteLine("\nMessage:{0}", e.Message);
        Console.WriteLine("\nStatus:{0}", e.Status);
        Console.WriteLine("Press any key to continue..........");
    }
    catch (Exception e)
    {
        Console.WriteLine("\nException raised!");
        Console.WriteLine("Source :{0} ", e.Source);
        Console.WriteLine("Message :{0} ", e.Message);
        Console.WriteLine("Press any key to continue..........");
        Console.Read();
    }
}

private static void ResponseCallback(IAsyncResult ar)
{
    HttpWebRequest request = ar.AsyncState as HttpWebRequest;
    Console.WriteLine("Callback 1");
    Thread.Sleep(3000);
    Console.WriteLine("Callback 2");
    HttpWebResponse webResponse = request.EndGetResponse(ar) as HttpWebResponse;//取得資料
    Console.WriteLine("Callback 3");
 
    Stream ReceiveStream = webResponse.GetResponseStream();
    StreamReader reader = new StreamReader(ReceiveStream);
    string result = reader.ReadToEnd();
 
    Console.WriteLine("Callback 4");
}
```

這裡是檢查結果，從這裡可以看到，一旦非同步作業完成之後，就會使用訊號通知方式，解除封鎖 Block 等待，也就是說，住執行緒可以繼續往下執行，而此時，callback 委派方法會在另外一個執行緒下，繼續來執行。

```
Main 1
Main 2
Main 3
   處理其他事情
Callback 1
   處理其他事情
   處理其他事情
Callback 2
Callback 3
Callback 4
Press any key for continuing...
```

