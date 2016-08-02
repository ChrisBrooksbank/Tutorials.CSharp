#Asynchronous Methods

Sometimes you will need to call a method which puts a heavy load on a local processor and takes a significent amount of time before it returns.


The following code :
* displays "About to start calculation"
* Hangs ( non responsive, blocked ), for three seconds, whilst it simulates running a, locally, computationally heavy process
* displays "Calculation has finished"
* displays "Returned from calculation with result : 42"
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
* Approximently three seconds later the calculation finishes, on its seperate thread, the continuation is activated, code exection gos back to where it was when calculation started
* displays "Returned from calculation with result : 42"
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
However often the delay is because we are waiting for a API call on a seperate process, likely seperate server, to complete.
This delay is unpredictable e.g. because of network conditions and current API load.

We dont need to run the code on a seperate thread here.
But we will defintely benefit from using await to attach a continuation.

The following code :
* displays "About to make API call"
* displays "API call was made, we may still be waiting for result though"
* Is not blocked, we could be running code in main thread, actually we just wait for user to press Enter
* Approximently five seconds later, the API returns its result, our contineuation is activated
* displays "API call is now returning"
```c#
 class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("About to make API call");
        GetNonBlockingAPIResultWrapper();
        Console.WriteLine("API call was made, we may still be waiting for result though");
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
        Console.WriteLine("API call is now returning");
        return 42;
    }

}
```

The above samples are designed to be simple to illustrate whats happening.
They are not production ready, e.g. no exception handling code !

What is a continuation ? whats actually happening above ?
Well its all on one thread.
Dotnet is creating a state machine in the background to implement the above flow.

Maybe Wikipedia will help with understanding : 
"First-class continuations are a language's ability to completely control the execution order of instructions. They can be used to jump to a function that produced the call to the current function, or to a function that has previously exited. One can think of a first-class continuation as saving the execution state of the program. It is important to note that true first-class continuations do not save program data – unlike a process image – only the execution context. This is illustrated by the "continuation sandwich" description:

Say you're in the kitchen in front of the refrigerator, thinking about a sandwich. You take a continuation right there and stick it in your pocket. Then you get some turkey and bread out of the refrigerator and make yourself a sandwich, which is now sitting on the counter. You invoke the continuation in your pocket, and you find yourself standing in front of the refrigerator again, thinking about a sandwich. But fortunately, there's a sandwich on the counter, and all the materials used to make it are gone. So you eat it. :-)[4]"

