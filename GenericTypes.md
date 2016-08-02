# What is a generic type ?

Generic Types give you all the advantages of code written to handle your specific types.
But without all the keyboard strokes that would be involved in writing lots of different classes.
And the maintenance of those extra types.
And of course you enable DRY ( Dont Repeat Yourself ), which is very important when writing code.

Say you are writing some shape handling software.
You define an interface chrisbrooksbank.Shapes.IShape
And a abstract base class chrisbrooksbank.Shapes.Shape which implements IShape

Then you create some concrete shape classes : 
* chrisbrooksbank.Shapes.Circle
* chrisbrooksbank.Shapes.Triangle
* chrisbrooksbank.Shapes.Square

Now you decide you need to implement a stack for storing your shapes.
You want a separate stack for each type of shape.

One stack must only store Circles, another only Triangles and the other only Squares.

=>

We could write three classes to do this, heres a naive CircleStack :

```c#
class CircleStack
{
    Circle[] circles = new Circle[100];
    int stackPointer = 0;

    public void Push ( Circle circle )
    {
        circles[stackPointer++] = circle;
    }

    public Circle Pop()
    {
        return circles[stackPointer--];
    }

}
```

So now we can use this class as :
```c#
CircleStack circleStack = new CircleStack();
circleStack.Push(new Circle());
```
          
This does mean the compiler makes it impossible to store anything but a circle on this circle stack.
And intellisense will also make that clear when to you.
That fits your aims.

Then add a TriangleStack and SquareStack.

But that seems like a lot of typing

And almost all of the code in CircleStack, TriangleStack and SquareStack is exactly the same.
Repeated.

=>

So, a better way is to define a generic class.
We can define our CircleStack, TriangleStack and SquareStack, but only write the code once.

We still get all the advantages of type safety, but we get to maintain DRY ( dont repeat yourself )
This is done by defining one class ShapeStack\<T\> :
```C#
class ShapeStack<T> where T :IShape
{
    T[] shapes = new T[100];
    int stackPointer = 0;

    public void Push(T shape)
    {
        shapes[stackPointer++] = shape;
    }

    public T Pop()
    {
        return shapes[stackPointer--];
    }
}
```

So the notation <T> is the most important part.
This is more like the template of a class, than a class.
The <T> is the marker for a type which we are not defining.

You can see we use the T, class marker, in different places in this class template
Push() takes a T
Pop() returns a T

So what about : where T :IShape
This is called a constraint
You dont need to specify a constraint
But in this case our class says that we can only specify a type which implements IShape
i.e. whatever class we decide to store in this stack, it must implement IShape the IShape interface 

This is called the Open form of the generic class, we havent specified a type.
We close the generic class when we define an instance of it and specify T.

So we simply change the line of code :
```c# 
CircleStack circleStack = new CircleStack();
```

to be :
```c# 
ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
```

And everything continues to work as before.

But we only wrote the code once, not three times.

##So to summarise : 

We showed how our program concerned itself with Triangle, Circle and Square classes.
That we wanted to create a seperate stack for each type of shape.
So we could have a CircleStack which only accepted circles.

The naive approach was to create a different stack class for each of the three shapes.
i.e. three class definitions.
* CircleStack
* SquareStack
* TriangleStack

All with near identical code, only the types would be different.

But a much better approach was to use generics to define a single class
```c# 
ShapeStack<T> where T :IShape
```

Where ShapeStack is a class template, the template is waiting for you to specify T when you create a instance of this class.

e.g. using
```c# 
ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
```

The dotnet framework comes with several inbuilt generic types but as you have seen you can also define your own.


