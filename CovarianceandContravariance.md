#Covariance and Contravariance

Imagine you have a base class : Shape and two descendants : Square and Circle.
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

If you have a method that takes IN a Shape you can pass in a Shape or Square or Circle.
This works because a Square is a Shape. And a Circle is a Shape.

i.e. this code compiles and runs OK : 
```c#
static void Main(string[] args)
{
    ProcessShape(new Shape());
    ProcessShape(new Circle());
    ProcessShape(new Square());
}

static void ProcessShape(Shape shape)
{
}
```

So now we need to think about generic types.
Heres one : 
```c#
class ShapeStack<T> where T :Shape
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

So heres the covariance question :
Can a ShapeStack\<Circle\> be implicitly converted to a ShapeStack\<Shape\> ?

Whats covariance ?
If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a covariant type parameter
If X\<A\> is implicitly convertable to X\<B\>

Type Circle is implicitly convertable to type Shape.
```c#
static void Main(string[] args)
{
    Circle circle = new Circle();
    Shape shape = circle;
}
```

So

Translating above definition to use our classes :
If type Circle is implicitly convertable to type Shape.
And ShapeStack is a generic type.
Then ShapeStack is said to have a covariant type parameter
If ShapeStack\<Circle\> is implicitly convertable to ShapeStack\<Shape\>

So does ShapeStack have a covariant type paramater ? No.
```c#   
ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
ShapeStack<Shape> shapeStack = circleStack;
'''

Gives this error :
> Error	CS0029	Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Circle>' to 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Shape>'

So how do we fix this ?

Split the generic ShapeStack class between two interfaces, and mark ShapeStack as implementing both.
One interface only takes in T, never returns it.
And the other only returns T, never accepts it.
Tell the compiler which is which.
Now you can have covariant and contravariant behaviour.

Define the T in and T out interfaces :
```c#
interface IStackPusher<in T> where T : Shape
{
   void Push(T shape);
}
 interface IStackPopper<out T> where T : Shape
{
    T Pop();
}
'''

Specify ShapeStack as implementing these interfaces :
'''c#
class ShapeStack<T> : IStackPusher<T>, IStackPopper<T> where T :Shape
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

Now ShapeStack<T> still doesnt have a covariant or contravariant type.
However IStackPusher does have a covariant type.
And IStackPopper does have a contravariant type.

So we can now convert our non working code
```c#
class Shape
{
    ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
    ShapeStack<Shape> shapeStack = circleStack;
}
```

To working code which only works because covariance and contravariance type parameters of our new interfaces : 
```c#
static void Main(string[] args)
{
    IStackPusher<Shape> shapePusher = new ShapeStack<Shape>();
    IStackPusher<Circle> circlePusher = shapePusher;

    IStackPopper<Circle> circlePopper = new ShapeStack<Circle>();
    IStackPopper<Shape> shapePopper = circlePopper;
}
```


#Contravariant

If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a contravariant type parameter
If X\<B\> is implicitly convertable to X\<A\>