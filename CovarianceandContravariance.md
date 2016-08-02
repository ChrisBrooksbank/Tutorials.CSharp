#Covariance and Contravariance

If a ( non generic ) method has a input paramater of type Shape.
Its OK to pass in a paramater of type Square.
Thats because a Square is a Shape.

Similary if a method returns a Square.
Its OK to return that into a variable declared as a Shape.
Because a Square is a Shape.

What about if you have a generic type, such as 
```c# class ShapeStack<T> where T :IShape

maybe you would expect
ShapeStack<Shape> to be implicitly converted to a ShapeStack<Square>
That would be termed contravariance.
Or maybe you would expect a ShapeStack<Square> to be implicitly converted to aS hapeStack<Shape>
That would be termed contravariance.

You would be wrong. However this can be done.

Would you expect this code to compile :
```c# ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
ShapeStack<Square> squarestack = circleStack;

No, you get a error : 
Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Circle>' to 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Square>'

What about this code ? : 
```c#  ShapeStack<IShape> shapeStack = new ShapeStack<IShape>();
ShapeStack< Square > squarestack = shapeStack;

No that doesnt compile either :
Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.IShape>' to 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Square>'

But you cant even compile this code :
```c#   ShapeStack<Square> squarestack = new ShapeStack<Square>();
ShapeStack<IShape> shapeStack = squarestack;

That might appear puzzling/annoying

=>

#Covariance

If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a covariant type parameter
If X<A> is implicitly convertable to X<B>

#Contravariant
If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a contravariant type parameter
If X<B> is implicitly convertable to X<A>

=>
So our ShapeStack has neither a Contravariant or a Covariant type parameter.
As you saw with the compiler errors above.

Why is this , and how would we fix this if we wanted to ?

The problem is that only types which are passed as parameters ( and never returned can be covariant)
And only types which are returned ( and never passed in ) can be contravariant.

A method accepting a Shape, you also want to accept a Square. ( Covariance )
But its impossible for a method accepting a Square to accept a Shape, that makes no sense.

Similary its OK to return a Square into a Shape ( Contravarince )
But you could never return a Shape into a Square, that makes no sense.

So you need to Split your generic ShapeStack class between two interfaces. 
One interface only takes in T, never returns it.
And the other only returns T, never accepts it.
Tell the compiler which is which.
Now you can have covariant and contravariant behaviour.

In ShapeStack <T> is both passed in and returned.
Passed in, in push.
Returned in pop.

So lets define define a IStackPusher interface :
```c#  
interface IStackPusher<in T> where T : IShape
{
   void Push(T shape);
}

Note the word in here : ```c# <in T>
It tells the compiler that the type is passed in, but never returned.
i.e. that thhe type will be contravariantly valid.

If you had accidently typed ```<out T> rather than ```c# <in T>
You get a compiler error :
Invalid variance: The type parameter 'T' must be contravariantly valid on 'IStackPusher<T>.Push(T)'. 'T' is covariant.	


now we will modify existing class ShapeStack to mark it as implementing IStackPusher :
```c#  class ShapeStack<T> : IStackPusher<T> where T :IShape

OK so if that worked
We should be able to implicitly cast a IStackPusher<Shape> to a IStackPusher<Square>

Lets try:
```c# 
ShapeStack<Shape> shapeStack = new ShapeStack<Shape>();
IStackPusher <Shape> shapePusher = shapeStack;
IStackPusher<Square> squarePusher = shapePusher;

Yes, that compiles OK as expected
Confiming that IStackPusher<T> has a covariant type parameter.

This is how you might expect the generic type to behave.
i.e. its expecting a Shape and you pass in a Square.
But a Square is a Shape ( it descends from it )

If you want to go ahead and create a contravariant type you will similary create :
```c# interface IStackPopper<out T> where T : IShape
{
   T Pop();
}

and dont forget :
```c#  class ShapeStack<T> : IStackPusher<T>, IStackPopper<T> where T :IShape

IStackPopper will have a contravariant type allowing code such as :
```c# 
ShapeStack<Square> shapeStack = new ShapeStack<Square>();
IStackPopper<Square> squarePopper = shapeStack;
IStackPopper<Shape> shapePopper = squarePopper;

Again this seems commonsense
If you are expecting to return a Square
It should also be fine to return a Shape
Because a Shape is what Square inherits from

#Summary
If you want to be able to implicitly convert between closed generic types, closed around related ( by inheritence ) types you need to either specify :
* The open type must either only accept the type, never return it
* Or the open type must only return the type, never accept it.
* That may be impossible, so instead split these two cases into two different generic interfaces, which your generic type implement.
* The interface must specify <in T> or <out T>
* Cast the closed generic type to the relevant interface
* These interfaces are covariant or contravariant








