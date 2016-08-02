#Delegates

A delegate is a reference which points to a method.

When you define a delegate type you specify the signature of the method you want it to be able to point to.

```c#
delegate int TakesIntReturnsInt(int x);
```

The above code declares a new type called TakesIntReturnsInt.
Which can point at methods which take an integer and return an integer.

so you could now say :
```c#
TakesIntReturnsInt incrementerDelegate = new TakesIntReturnsInt(i => i+1);
Console.WriteLine(incrementerDelegate.Invoke(42));
Console.ReadLine();
```

And this would display 43.
Actually you dont need tp specify Invoke explicitly, you can simplify above to :
```c#
TakesIntReturnsInt incrementerDelegate = new TakesIntReturnsInt(i => i+1);
Console.WriteLine(incrementerDelegate(42));
Console.ReadLine();
```

If you havent seen the lambda notation before.
```c#
i => i+1
```

Its simply a convient way of defining an anonymous function

i.e. we could have said :
```c#
class Program
{
    delegate int TakesIntReturnsInt(int x);

    static int Incrementer(int i)
    {
        return i + 1;
    }

    static void Main(string[] args)
    {
        TakesIntReturnsInt incrementerDelegate = new TakesIntReturnsInt(Incrementer);
        Console.WriteLine(incrementerDelegate(42));
        Console.ReadLine();
    }
}
```

But thats a lot of typing, here we had to create a named method.

##Multicast Delegates
Delegates are all multicast delegates. 
This means one delegate can point to a list of compatible methods.
These are added with +=
Or removed with -=


##Generic Delegate Types
Above we created a non generic delegate type :
```c#
delegate int TakesIntReturnsInt(int x);
```

However c# has support for generic delegates. 
See [generictypes.md](Generic Types)

e.g.
```c#
delegate void DrawShape<T>(T arg);
```

##Delegate types in the box

Some scenarios are very common and the dotnet framework comes with types in the box to reduce the number
of delegate types you need to define yourself.

###Func
These are a site of pre defined generic delegate classes taking from zero to 16 input parameters and returning a value.

###Action
These are a site of pre defined generic delegate classes taking from zero to 16 input parameters and returning void.

###Event
If you declare a type as a event rather than a delegate you get some important advantages when used in the popular publish, subscribe model. This is where a class publishes an event, e.g. that fires off whenever it changes. Subscribes can subscribe to this event to be notified of changes.

The advantage of using event rather than delegate is subscribers lose the ability to invoke the delegate themself. They are also unable to remove other subscribers from the event.

There is a standard pattern to follow when you want to define a event on a class and allow multiple subscribers.

```c#


```

