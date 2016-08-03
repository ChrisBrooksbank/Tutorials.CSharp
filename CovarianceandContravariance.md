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

If you have a method that returns a Circle ( or Square ) you can assign it to a variable declared as a Shape.
This works because a Square ( or Circle ) is a Shape.

i.e. this code compiles and runs OK : 
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

So heres the covariance question :
Can a ShapeStack\<Circle\> be implicitly converted to a ShapeStack\<Shape\> ?

Whats covariance ?
If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a covariant type parameter
If X\<A\> is implicitly convertable to X\<B\>

Type Circle is implicitly convertable to type Shape.
As we already saw in this code :
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

Translating above definition to use our classes :
If type Circle is implicitly convertable to type Shape.
And ShapeStack is a generic type.
Then ShapeStack is said to have a covariant type parameter
If ShapeStack\<Circle\> is implicitly convertable to ShapeStack\<Shape\>

So does ShapeStack have a covariant type paramater ? No.

```c#   
ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
ShapeStack<Shape> shapeStack = circleStack;
```

Gives this error :
> Error	CS0029	Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack\<ChrisBrooksbank.Shapes.Circle\>' to 'ChrisBrooksbank.Shapes.ShapeStack\<ChrisBrooksbank.Shapes.Shape\>'

So how do we fix this ?

Split the generic ShapeStack class between two interfaces, and mark ShapeStack as implementing both.
One interface only returns T, never accepts it. ( and has covariant type parameter )
The other interface only takes in T, never returns it. ( and has contravariant type parameter )
Tell the compiler which is which.
Now you can have covariant and contravariant behaviour.

Define the T in and T out interfaces :
```c#
interface IStackPopper<out T> where T : Shape
{
    T Pop();
}
interface IStackPusher<in T> where T : Shape
{
   void Push(T shape);
}
'''

Specify ShapeStack as implementing these interfaces :
'''c#
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
However IStackPopper does have a covariant type.
And IStackPusher does have a contravariant type.

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
    // covariance
    IStackPopper<Circle> circlePopper = new ShapeStack<Circle>();
    IStackPopper<Shape> shapePopper = circlePopper;

    // contravariance
    IStackPusher<Shape> shapePusher = new ShapeStack<Shape>();
    IStackPusher<Circle> circlePusher = shapePusher;

    circlePusher.Push( new Circle());
    Circle aCircle = circlePopper.Pop();
}
```


#Contravariant
If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a contravariant type parameter
If X\<B\> is implicitly convertable to X\<A\>