#Dependency-Injection using IoC containers ( e.g. Autofac )

These are rough notes from which a awesome tutorial will emerge

Inversion of Control is a vague concept
Changes control from normal way, inverting it
e.g.
* Control over the interface between two systems or components
* Control over the flow of an operation
* Control over dependency creation and binding

Principle IS "Dependency Inversion"
Pattern(ish) IS : Inversion of Control
* Interface Inversion
* Flow Inversion
* Dependency Creation/Binding Inversion = this is what people think of when they think of IOC

##Interface Inversion
Just because you created an interface doesnt mean you are implementing the inversion of control correctly.
Some people just put an I in front of class and call it an interface.
BoxingMatch IKangaroo, implementation doesnt buy us anything
Maybe we later define a IMikeTyson
No !
Lets invert interfaces and have BoxingMatch ( the user ) defining its own provider
a IBoxer and a BoxingMatch
( HueBridge IHueBridgeClient ? )
now JoeBoxer and MikeTyson depend on IBoxer
If Kangaroo wants to be a provider for a boxing match, they have to become a IBoxer
* BoxingMatch has control over the interface that users have to follow
* An interface should have more than one implentation, no point, no benefit unless multiple implementations of an interface *

Another example of interface inversion is the Provider Model
Consumer is in control of interface providers have to follow
Thats inversion from traditional approach when Consumer had to support multiple Providers
Some reasons for using a factory pattern !
* We want to have creation of a set of objects in one central place
* 

##Flow Inversion
Traditional : Step by step, e.g. Console Application
Inverted Flow = GUI, we can fill in whichever field, whichever order, then click the "Submit" button whenever we want

##Creation Inversion
The type of IOC most people are familiar with
Even with interface inversion we can still not have creation inversion.
If we have code as :
IMyInterface myObject = new MyImplementation();

We dont want switch(ButtonType) repeated, instead put it in a ButtonFactory class
Now dependant on ButtonFactory, higher level

Types of creation inversion ( more than just Dependency Injection ) :
* Factory Pattern
	Button button = ButtonFactory.CreateButton()
* Service Locator
  Button button = ServiceLocator.Create(IButton.class)
* Dependency Injection
  Button button = GetTheRightDangButton();
  OurScreen ourScreen = new OutScreen(button);
* More...


Constructor injection is most common way to inject

Bob Martin's paper, article in C++ Report May 1996
Founder of SOLID principles
* Single responsibility principle
* Open / closed principle
* Liskov substiution principle
* Interface segregation principle
* Dependency inversion principle

High level-modules should not depend on low-level modules. Both should depend on abstractions.
Abstractions should not depend upon details. Details should depend ob abstractions.

Layering ( Traditional )
Policy Layer calls Mechanism Layer calls Utility Layer
Problem each layer is dependent on the one below it
So changes bubble up

So apply DIP to Layering instead. IOC
Now Policy Layer depends on a mechanism "*interface*"
Mechanism layer implements that and depends on utility *interface*
Utility layer implements that
Now : you can change implementation details without affecting layers above
protecting system against changes

Button / Lamp
Button depends on lamp.
Pass lamp into button.
When turnon or turnoff on button it calls turnon or turnoff on lamp.

Solution : define IButtonClient


A key benefit of DI is to avoid class coupling.
- One class depending on another
- Limits functionality to single implementation
- If classes perform DB work its difficult to test without hitting the DB

itempotent
Embrace abstractions to write testable software.

Define dependencies as interfaces.
Can use different implementations at different times. e.g. at test time and production time.

Creating classes in your classes with new Class( )is bad

https://autofac.org/

DI containers end result is always a bucket of name/value pairs associating interfaces with concrete classes

Service locator pattern ( or abstract factory )

Big issue is DI containers hold onto resolved instances
Dont really know when container is disposed

Can ask DI container for single instance

[See this video](https://www.youtube.com/watch?v=I6-QI1qN4pU)
[Pluralsight with John Sonmez for asp.net mvc 4](https://app.pluralsight.com/player?course=ioc-aspdotnet-mvc4&author=john-sonmez&name=ioc-aspdotnet-mvc4-m1&clip=0&mode=live)


Martin Fowler. 
"Dont call us, we'll call you"


##Inversion of Control ( IOC ) with structuremap
I can immeditely see a problem, looking at
```c#
IHueBridge bridge = new HueBridge();
```

Our code should follow SOLID ( https://en.wikipedia.org/wiki/SOLID_(object-oriented_design) )
And the D is "Dependency inversion principle"
"one should “Depend upon Abstractions. Do not depend upon concretions"

OK so our variable may be defined as the interface type, but the instantation of concrete class HueBridge is hardcoded into the test.
Lets fix that first.

Install the Nuget package : "StructureMap" and configure the IOC container in a test intitialiation method.
Remove the hardcoding of the concrete HueBridge from the test.
This gives us :

```c#
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Hue.CSharp.TDD.Tests
{

    public interface IHueBridge
    {
        string GetNewUserName();
    }

    public class HueBridge: IHueBridge
    {
        public string GetNewUserName()
        {
            return null;
        }
    }

    [TestClass]
    public class BridgeTests
    {
        static StructureMap.Container iocContainer;

        [TestInitialize]
        public void TestInit()
        {
            iocContainer = new StructureMap.Container(_ =>
            {
                _.For<IHueBridge>().Use<HueBridge>();
            });
        }

        [TestMethod]
        public void CanGenerateNewUserName()
        {
            IHueBridge bridge = iocContainer.GetInstance<IHueBridge>();

            string newUserName = bridge.GetNewUserName();
            Assert.IsNotNull(newUserName);
        }
    }
}
```