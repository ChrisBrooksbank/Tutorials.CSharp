#Covariance and Contravariance

Imagine you have a base class: Shape and two descendants: Square and Circle.
```c#
class Shape
{
}

class Square: Shape
{
}

class Circle: Shape
{
}
```

##Covariance

If you have a method that returns a Circle (or Square) you can assign it to a variable declared as a Shape.
This works because a Square (or Circle) is a Shape.

i.e. this code compiles and runs OK : 
```c#
static void Main(string[] args)
{
    Shape shape = PopCircle();           
}

static Circle PopCircle()
{
    return new Circle();
}
```

So now we need to think about generic types.
Heres one : 
```c#
class ShapeStack<T> where T :Shape
{
    T[] shapes = new T[100];
    int stackPointer = 0;

    public T Pop()
    {
        return shapes[stackPointer--];
    }

    public void Push(T shape)
    {
        shapes[stackPointer++] = shape;
    }
}
```

So here’s the covariance question :
Can a ShapeStack\<Circle\> be implicitly converted to a ShapeStack\<Shape\> ?

Whats covariance ?
If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a covariant type parameter
If X\<A\> is implicitly convertible to X\<B\>

Type Circle is implicitly convertible to type Shape.
As we already saw in this code:
```c#
static void Main(string[] args)
{
    Shape shape = PopCircle();           
}

static Square PopCircle()
{
    return new Circle();
}
```

So

Translating above definition to use our classes:
If type Circle is implicitly convertible to type Shape.
And ShapeStack is a generic type.
Then ShapeStack is said to have a covariant type parameter
If ShapeStack\<Circle\> is implicitly convertible to ShapeStack\<Shape\>

So does ShapeStack have a covariant type parameter ? No.

```c#   
ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
ShapeStack<Shape> shapeStack = circleStack;
```

Gives this error :
> Error CS0029  Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack\<ChrisBrooksbank.Shapes.Circle\>' to 'ChrisBrooksbank.Shapes.ShapeStack\<ChrisBrooksbank.Shapes.Shape\>'

So how do we fix this?

A generic type which accepts a T ( as opposed to returning a T ) can never be CoVariant for T.
We can create a generic interface which only returns a T ( and is covariant on T )
And another generic interface which only accepts a T ( and is contravariant on T )
Now we modify the definition of ShapeStack to implement both interfaces.

ShapeStack remains non covariant and non contravariant with respect to T.
However, we can work with its interfaces instead.

Define the T in and T out interfaces :
```c#
// IStackPopper has a covariant type parameter T
interface IStackPopper<out T> where T : Shape
{
    T Pop();
}
// IStackPusher has a contraVariant type parameter T
interface IStackPusher<in T> where T : Shape
{
   void Push(T shape);
}
```

Specify ShapeStack as implementing these interfaces:
```c#
class ShapeStack<T> : IStackPopper<T>, IStackPusher<T> where T :Shape
{
    T[] shapes = new T[100];
    int stackPointer = 0;

    public T Pop()
    {
        return shapes[stackPointer--];
    }

    public void Push(T shape)
    {
        shapes[stackPointer++] = shape;
    }
  
}
```

Now ShapeStack<T> still doesnt have a covariant or contravariant type parameter.
However IStackPopper does have a covariant type T.
And IStackPusher does have a contravariant type T.

So we can now convert our non-working code
```c#
class Shape
{
    ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
    ShapeStack<Shape> shapeStack = circleStack;
}
```

To working code which only works because covariance and contravariance type parameters of our new interfaces: 
```c#
static void Main(string[] args)
{
    // covariance
    IStackPopper<Circle> circlePopper = new ShapeStack<Circle>();
    IStackPopper<Shape> shapePopper = circlePopper;

    // contravariance
    IStackPusher<Shape> shapePusher = new ShapeStack<Shape>();
    IStackPusher<Circle> circlePusher = shapePusher;

    circlePusher.Push( new Circle());
    Shape aShape = shapePopper.Pop();
}
```

The working example of covariance are these lines.
```c#
// covariance
IStackPopper<Circle> circlePopper = new ShapeStack<Circle>();
IStackPopper<Shape> shapePopper = circlePopper;
```

#Contravariant
Contra variance is discussed above.
It allows for implicit conversion in the opposite direction.

If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a contravariant type parameter
If X\<B\> is implicitly convertable to X\<A\>

So a Circle can be implicitly converted to a Shape.
And IStackPusher is a generic type.
Type IStackPusher is said to have a contravariant type parameter
If IStackPusher\<Shape\> is implicitly convertable to IStackPusher\<Circle\>

And it can :
```c#
// contravariance
IStackPusher<Shape> shapePusher = new ShapeStack<Shape>();
IStackPusher<Circle> circlePusher = shapePusher;
```
