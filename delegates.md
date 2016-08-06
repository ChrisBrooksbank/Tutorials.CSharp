#Delegates

A delegate is a reference which points to a method.

When you define a delegate type you specify the signature of the method you want it to be able to point to.

```c#
delegate int TakesIntReturnsInt(int x);
```

The above code declares a new type called TakesIntReturnsInt.
Which can point at methods which take an integer and return an integer.

so you could now say:
```c#
TakesIntReturnsInt incrementerDelegate = new TakesIntReturnsInt(i => i+1);
Console.WriteLine(incrementerDelegate.Invoke(42));
Console.ReadLine();
```

And this would display 43.
Actually you don’t need to specify Invoke explicitly, you can simplify above to :
```c#
TakesIntReturnsInt incrementerDelegate = new TakesIntReturnsInt(i => i+1);
Console.WriteLine(incrementerDelegate(42));
Console.ReadLine();
```

If you haven’t seen the lambda notation before.
```c#
i => i+1
```

Its simply a convenient way of defining an anonymous function

i.e. we could have said:
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

But that’s a lot of typing, here we had to create a named method.

##Multicast Delegates
Delegates are all multicast delegates. 
This means one delegate can point to a list of compatible methods.
These are added with +=
Or removed with -=


##Generic Delegate Types
Above we created a non-generic delegate type :
```c#
delegate int TakesIntReturnsInt(int x);
```

However c# has support for generic delegates. 
See [Generic Types](GenericTypes.md)

e.g.
```c#
delegate void DrawShape<T>(T arg);
```

##Delegate types in the box

Some scenarios are very common and the dotnet framework comes with types in the box to reduce the number
of delegate types you need to define yourself.

###Func
These are a set of pre-defined generic delegate classes taking from zero to 16 input parameters and returning a value.

###Action
These are a set of pre-defined generic delegate classes taking from zero to 16 input parameters and returning void.

###EventHandler
This is a generic delegate type supporting #Event.

###Event
If you declare a type as a event rather than a delegate you get some important advantages when used in the popular publish, subscribe model. This is where a class publishes an event, e.g. that fires off whenever it changes. Subscribes can subscribe to this event to be notified of changes.

The advantage of using event rather than delegate is subscribers lose the ability to invoke the delegate themselves. They are also unable to remove other subscribers from the event.

There is a standard pattern to follow when you want to define a event on a class and allow multiple subscribers.

First you should create a descendant of the EventArgs class, to pass the information you want subscribers to see.
Now you can define an event on your class of type EventHandler<CustomEventArgsClassName>
Create a protected virtual void method called On<EventName> which invokes the event
Add the code which fires the event e.g. in a setter

Now lets add an event to our Shape class, which will now look like this :
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ChrisBrooksbank.Shapes
{

    public class ShapeChangedEventArgs : EventArgs
    {
        public readonly string OldShapeDescription;
        public readonly string NewShapeDescription;
        public ShapeChangedEventArgs(string oldShapeDescription, string newShapDescription)
        {
            OldShapeDescription= oldShapeDescription;
            NewShapeDescription = newShapDescription;
        }
    }


    class Shape: IShape
    {
        public String ShapeDescription
        {
            get { return ShapeDescription; }
            set {
                if (ShapeDescription.Equals(value)) return;
                OnShapeChanged(new ShapeChangedEventArgs(ShapeDescription, value));
                ShapeDescription = value;
            }
        }

        public event EventHandler<ShapeChangedEventArgs> ShapeChanged;

        protected virtual void OnShapeChanged(ShapeChangedEventArgs e)
        {
            if (ShapeChanged != null) ShapeChanged(this, e);
        }


    }
}
```

