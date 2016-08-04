#TDD ( Draft V0.1 )

TDD can stand for "Test Driven Development" or "Test Driven Design".

In this tutorial I am going to use TDD to start development of code to control my Phillips hue lights.
These are light bulbs which can be controlled ( turned on, turned off, dimmed, changed color, set on timers ) by making API calls to a Hue bridge.

This tutorial shows the important techniques. 
A tutorial showing all the details of this would be in multiple parts ( I may add this later )

##Create solution

I created a new console application in Visual Studio 2015 called Hue.CSharp.TDD
Next I created a new "Unit Test" project on the same solution called "Hue.CSharp.TDD.Tests"
And renamed the auto generated class UnitTest1.cs to BridgeTests.cs

We are going to do all our initial work in the file BridgeTests.cs
Which currently looks like this
```c#
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Hue.CSharp.TDD.Tests
{
    [TestClass]
    public class BridgeTests
    {
        [TestMethod]
        public void TestMethod1()
        {
        }
    }
}
```

OK where to start ?

Well in order to control any lights we need to be able to connect to the bridge.
The bridge is a physical device which connects to a internet router via a patch cable.
To setup a bulb we need to connect it to the bridge.
We can then control the bulb by sending commands to the Bridge via its Restful API.

Before a application can use the bridge it needs to create a new username.
This involves sending a command to the bridge and pressing the button on the bridge within 30 seconds ( for security, proving we have physical access ).

```c#
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Hue.CSharp.TDD.Tests
{
    public class HueBridge
    {
        public string GetNewUserName()
        {
            return null;
        }
    }

    [TestClass]
    public class BridgeTests
    {
        [TestMethod]
        public void CanGenerateNewUserName()
        {
            HueBridge bridge = new HueBridge();

            string newUserName = bridge.GetNewUserName();
            Assert.IsNotNull(newUserName);
        } 
    }
}
```

OK we created a abstraction for the Hue bridge by adding a HueBridge class.
( There was no justification to create a Interface type )

Then we added a test for CanGenerateNewUserName() and added this method.
The test fails as expected, because no username is generated.
Thats as expected.
TDD = test driven development.

Red, Green, Refactor : 
We create a failing test ( red ) then make it pass ( go green ), then we are free to refactor code
thats the normal flow in TDD


##Mocking ( with Moq )
We could continue building out the bridge tests and getting them to pass.
However I would rather talk about mocking as this is meant to be a relatively short tutorial.

Lets create a light bulb.
Which implements off and on.
The bulb is associated with a bridge.

The first thing is we dont want to connect the bulb to an actual physical bridge.
That would make it difficult to test without physical access to a real system.
Instead we want to create a mock bridge for our light bulb tests.

First is to install the NuGet package Moq
Then create a new unittest file called BulbTests.cs

So as stated, we are testing the bulbs logic here
We dont want to test the bridge, that has its own tests.
In fact we dont want the bulb testing to rely on having a physical working bridge at all.
Or a working IHueBridge implementation.
Which is why we mock the IHueBridge in the following code.


So we generate our failing test : 
```c#
namespace Hue.CSharp.TDD.Tests
{
    public class HueBulb
    {
        public HueBridge Bridge { get; set; }
        public string BulbName { get; set; }

        public bool On
        {
            get { throw new NotImplementedException(); }
            set { throw new NotImplementedException(); }
        }

        public int Brightness
        {
            get { throw new NotImplementedException(); }
            set { throw new NotImplementedException(); }
        }

        public void Refresh()
        {
            throw new NotImplementedException();
        }
    }
    public class HueColourBulb: HueBulb
    {
        public string Color
        {
            get { throw new NotImplementedException(); }
            set { throw new NotImplementedException(); }
        }
    }

    [TestClass]
    public class BulbTests
    {
        Mock<HueBridge> mockBridge;

        [TestInitialize]
        public void TestInit()
        {
            mockBridge = new Mock<HueBridge>();
        }

        [TestMethod]
        public void LandingBulbTurnsOn()
        {
            HueBulb landingBulb = new HueBulb { Bridge = mockBridge.Object, BulbName = "Landing" };

            landingBulb.On = true;
            Assert.IsTrue(landingBulb.On, "bulb didnt turn on");
        }

    }
}
```

We should now write the implementation code to turn this test green.
We need more thought to the design, e.g. the HueBridge will have a URL
We also need something to represent a user

A really important thing to note is the code we are writing to interact with our bulb :

```c#
HueBulb landingBulb = new HueBulb { Bridge = mockBridge.Object, BulbName = "Landing" };
landingBulb.On = true;
```

TDD can stand for "Test Driven Design"

You should examine the code above and decide if you are writing the best API.
i.e. is the code which accesses API clear, is it easy to discover, does it encourage best practices.

If not refactor the test code to be optimal
Then amend the API implementation so it compiles again, and the test is green.


