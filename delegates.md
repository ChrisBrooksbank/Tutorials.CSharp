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


