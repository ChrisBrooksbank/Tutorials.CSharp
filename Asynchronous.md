#Asynchronous Methods

Sometimes you will need to call a method which puts a heavy load on a local processor and takes a significent amount of time before it returns.


The following code :
* displays "About to start calculation"
* Hangs ( non responsive, blocked ), for three seconds, whilst it simulates running a, locally, computationally heavy process
* displays "Calculation has finished"
* displays "Returned from calculation with result : 43"
```c#
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("About to start calculation");
        int data = GetHeavyComputeCycleData();
        Console.WriteLine("Returned from calculation with result : " + data.ToString());
        Console.ReadLine();
    }

    static int GetHeavyComputeCycleData()
    {
        DateTime waitUntil = DateTime.Now.AddSeconds(3);
        while (DateTime.Now < waitUntil) { }
        Console.WriteLine("Calculation has finished");
        return 42;
    }

}
```

We will now improve the code, to remove that "Hangs ( non responsive, blocked )".
Its actually a console application to make the example as easy to understand as possible.
Imagine its in a Windows or Web application and we are keeping the UI responsive.

C# has a await operator which lets the caller attach a continuation.
C# also has a Task class, with a Run method, which runs a task on a different thread, helping keep the current one responsive.

We need to add the async keyword to methods that have calls to await. 
( This prevents potentially breaking old c# code, which may have used await as a variable name).

We cant mark a entry point, such as main, async, so we will need a wrapper for our compute heavy routine.
Lets call that wrapper GetHeavyComputeCycleDataWrapper()

The following, improved, code :
* displays "About to start calculation"
* displays "Returned from calculation with result currently unknown - still computing"
* Is not blocked, we could be running code in main thread, actually we just wait for user to press Enter
* Approximently three seconds later the calculation finishes, on a seperate flag, the continuation is activated, code exection gos back to where it was when calculation started
* displays "Returned from calculation with result : 43"
```c#
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("About to start calculation");
        GetHeavyComputeCycleDataWrappper();
        Console.WriteLine("Returned from calculation with result currently unknown - still computing");
        Console.ReadLine();
    }

    static async void GetHeavyComputeCycleDataWrappper()
    {
        int data =  await Task<int>.Run( ( )=> GetHeavyComputeCycleData());
        Console.WriteLine("Returned from calculation with result : " + data.ToString());
    }

    static int GetHeavyComputeCycleData()
    {
        DateTime waitUntil = DateTime.Now.AddSeconds(3);
        while (DateTime.Now < waitUntil) { }
        return 42;
    }

}
```

The above is a great solution for locally compute heavy calls.
We dont need the overhead of running code on a seperate threads when the slowness is caused by API calls across processes though.


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




