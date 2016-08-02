#Asynchronous Methods

Sometimes you want to call a method which could be slow because of heavy compute cycle demands.

e.g. following code blocks for a few seconds, before the answer, 42 is returned.

```c#
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("About to get data that takes heavy compute resource");
        int data = GetHeavyComputeCycleData();
        Console.WriteLine("Returned from slow call with : " + data.ToString());
        Console.ReadLine();
    }

    static int GetHeavyComputeCycleData()
    {
        DateTime waitUntil = DateTime.Now.AddSeconds(3);
        while (DateTime.Now < waitUntil) { }
        Console.WriteLine("slow method about to return");
        return 42;
    }

}
```

C# has a await operator which lets the caller attach a continuation.

We would prefer to not wait for the slow method to call before resuming program execution.
We can run the task on a separate thread ( because its compute heavy, on same PC, so this is worth doing )

We need to add the async keyword to methods that have calls to await. ( This prevents breaking old c# code which may have used await as a variable name).

We cant make a entry point, such as main, async, so we will need a wrapper for our compute heavy routine.
Lets call that wrapper GetHeavyComputeCycleDataWrapper()

Here is code : 

```c#
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("About to get data that takes heavy compute resource");
        GetHeavyComputeCycleDataWrappper();
        Console.WriteLine("Returned from slow call, data unknown yet");
        Console.ReadLine();
    }

    static async void GetHeavyComputeCycleDataWrappper()
    {
        int data =  await Task<int>.Run( ( )=> GetHeavyComputeCycleData());
        Console.WriteLine("slow call returned with : " + data.ToString());
    }

    static int GetHeavyComputeCycleData()
    {
        DateTime waitUntil = DateTime.Now.AddSeconds(3);
        while (DateTime.Now < waitUntil) { }
        Console.WriteLine("slow method about to return");
        return 42;
    }

}
```

Now you see the immediate response : "Returned from slow call, data unknown yet"
After a few seconds : "heavy call returned with : 42"

Now in many cases the delay will be due to non-blocking API calls rather than local compute cycles. In such cases there is no need to await a task, you can simply await. Dotnet then generates a state machine for you and the continuation will work. To avoid running a task on another thread use code such as :

```c#
 class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("About to get data that is speed dependnant on network");
        GetNonBlockingAPIResultWrapper();
        Console.WriteLine("Done that call");
        Console.ReadLine();
    }

    static async Task GetNonBlockingAPIResultWrapper()
    {
        int data = await GetNonBlockingAPIResult();
        Console.WriteLine("slow network API call returned with : " + data.ToString());
    }

    static async Task<int> GetNonBlockingAPIResult()
    {
        await Task.Delay(5000);
        Console.WriteLine("slow network API method about to return");
        return 42;
    }

}
```




