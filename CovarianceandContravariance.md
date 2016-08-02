#Covariance and Contravariance

If a ( non generic ) method has a input paramater of type Shape.
Its OK to pass in a paramater of type Square.
Thats because a Square is a Shape.

Similary if a method returns a Square.
Its OK to return that into a variable declared as a Shape.
Because a Square is a Shape.

What about if you have a generic type, such as 
```c# 
class ShapeStack<T> where T :IShape
```

maybe you would expect
ShapeStack<Shape> to be implicitly converted to a ShapeStack\<Square\>
Or maybe you would expect a ShapeStack\<Square\> to be implicitly converted to aS hapeStack\<Shape\>

You would be wrong. 
However this can be achieved with some changes to the code.

You wouldnt expect this code to compile, circles and squares are not compatible  
```c# 
ShapeStack<Circle> circleStack = new ShapeStack<Circle>();
ShapeStack<Square> squarestack = circleStack;
```

Error : 
Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Circle>' to 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Square>'

What about this code ?
```c#  
ShapeStack<IShape> shapeStack = new ShapeStack<IShape>();
ShapeStack< Square > squarestack = shapeStack;
```

Error :
Cannot implicitly convert type 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.IShape>' to 'ChrisBrooksbank.Shapes.ShapeStack<ChrisBrooksbank.Shapes.Square>'

This also fails to compile 
```c#   
ShapeStack<Square> squarestack = new ShapeStack<Square>();
ShapeStack<IShape> shapeStack = squarestack;
```

#Covariance

If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a covariant type parameter
If X\<A\> is implicitly convertable to X\<B\>

#Contravariant

If type A is implicitly convertable to type B.
And X is a generic type.
Then type X is said to have a contravariant type parameter
If X\<B\> is implicitly convertable to X<A>

#Creating CoVariant and ContraVariant Generic Types
So our ShapeStack has neither a Contravariant or a Covariant type parameter.
As you saw with the compiler errors above.

Why is this , and how would we fix this if we wanted to ?

Only "Input only" types result in Covariant generic types
Only "Output only" types result in Contravariant generic types

Confused ? Think about this in terms of shapes.
In only : void Push(Shape shape) can accept a Square ( or Circle, or Triangle or Shape)
Out only : Square Pop() can return into a Shape ( or Square )

We need to give the compiler a "in only" type and a "out only" type.

So we need to Split the generic ShapeStack class between two interfaces, and mark as implementing both.
One interface only takes in T, never returns it.
And the other only returns T, never accepts it.
Tell the compiler which is which.
Now you can have covariant and contravariant behaviour.

In ShapeStack \<T\> is both passed in and returned.
Passed in, in push.
Returned in pop.

So lets define define a IStackPusher interface :
```c#  
interface IStackPusher<in T> where T : IShape
{
   void Push(T shape);
}
```

Note the word in here : ```
c# <in T>
```
It tells the compiler that the type is passed in, but never returned.
i.e. that thhe type will be contravariantly valid.

If you had accidently typed ```<out T> rather than ```c# <in T>
You get a compiler error :
Invalid variance: The type parameter 'T' must be contravariantly valid on 'IStackPusher<T>.Push(T)'. 'T' is covariant.	


now we will modify existing class ShapeStack to mark it as implementing IStackPusher :
```c#  
class ShapeStack<T> : IStackPusher<T> where T :IShape
```

OK so if that worked
We should be able to implicitly cast a IStackPusher<Shape> to a IStackPusher<Square>

Lets try:
```c# 
ShapeStack<Shape> shapeStack = new ShapeStack<Shape>();
IStackPusher <Shape> shapePusher = shapeStack;
IStackPusher<Square> squarePusher = shapePusher;
```

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
```

and dont forget :
```c#  
class ShapeStack<T> : IStackPusher<T>, IStackPopper<T> where T :IShape
```

IStackPopper will have a contravariant type allowing code such as :
```c# 
ShapeStack<Square> shapeStack = new ShapeStack<Square>();
IStackPopper<Square> squarePopper = shapeStack;
IStackPopper<Shape> shapePopper = squarePopper;
```

Again this seems commonsense
If you are expecting to return a Square
It should also be fine to return a Shape
Because a Shape is what Square inherits from

#Summary
You probably know that when a method wants a Shape you could pass in a Square.
Or that when a method returns a Square you can return it into a Shape.

However if you want a generic type to have covariant and contravariant types then you must specify in and out on the types. This may mean splitting into two interfaces.







