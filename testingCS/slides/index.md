- title : Elements of Testable Code
- description : Attributes and Techniques of Testable Code
- author : Eric Lobdell
- theme : Night
- transition : default

***

# Simpler C# Testing

--- 

define simple

---

example of complex test

--- 

example of simple test

***

# We'll Discuss
## Code That Makes for Simple Tests
## Tools That Can Simplify Your Tests

***

# Code That's Easy To Test

***

## Taking Some Notes From Functional Programming
<p>How Some Functional Concepts Can Aid Your Code's Testabliity</p>

***

# Avoid Void
<p>Void Methods are Black Boxes</p>
<p>You Have to Ask Someone Else What Happened</p>

---

    [lang=cs]
    public void DoThing(int x, int y)
    {
        myService.Thing(x,y);
    };

    [Fact]
    public void DoThing_calls_the_right_method_with_the_right_values()
    {
        var mockService = A.Fake<IMyService>();
        var x = 1;
        var y = 2;

        DoThing(x,y);

        A.CallTo(() => mockService.Thing(x,y))
            .MustHaveHappened();
    }

---

# Prefer Functions 
*Functions: Expressions That Always Return a Value*

---

    [lang=cs]
    public MyResultType DoThing(int x, int y)
    {
        return myService.Thing(x,y);
    };

    [Fact]
    public void DoThing_calls_the_right_method_with_the_right_values()
    {
        var x = 1;
        var y = 2;

        var sut = DoThing(x,y);

        Assert.True(sut.Success);
    }

---

### You're Free to Return Whatever You Need to Confirm Behavior! So do it!
</hr>
### You're Writing Code Around Your Tests, Not Tests Aroud Your Code

***

# Pure Functions

<p>Given the same input, always produces the same output</p>
<p>Have no side-effects</p>

---

## Add is Pure

    [lang=cs]
    public string Add(int x, int y) => x + y;

    [Fact]
    public void Add_returns_expected_value()
    {
        var x = 1;
        var y = 2;

        var sut = Add(x,y);
        var expected = 3;

        Assert.Equal(expected, sut);
    }

---

## Add is Impure

    [lang=cs]
    public void Add(int x, int y)
    {
        var z = myService.GetValue(x);
        myInstance.Property = z + y;
        myOtherService.DoAdd(myInstance);
    };

    [Fact]
    public void Add_properly_calls_method_on_my_service()
    {
        //
    }

    [Fact]
    public void Add_sets_state_on_instance_correctly()
    {
        //
    }

    [Fact]
    public void Add_properly_calls_method_on_other_service_with_instance()
    {
        //
    }

***

## Writing Tests That are Easy to Understand
 
***

## Avoid Mocking
### Mocking is a Layer of Indirection </br> Adds to Cognitive Load of Understanding the Test

---

### Pass Values Not Value Providers

<p>Value Providers Require Setup</p>
<p>Values Just Need to be Set = Less Noise</p>

---

    [lang=cs]
    public string Add(INumberService ns)
    {
        var x = ns.GetX();
        var y = ns.GetY();

        return x + y;
    }

    [Fact]
    public void Add_returns_expected_value()
    {
        var x = 1;
        var y = 2;
        var mockNumberService = A.Fake<INumberService>();

        A.CallTo(() => mockNumberService.GetX())
            .Returns(x);
        A.CallTo(() => mockNumberService.GetY())
            .Returns(y);

        var sut = Add(mockNumberService);
        var expected = 3;

        Assert.Equal(expected, sut);
    }

--- 

    [lang=cs]
    public string Add(int x, int y) => x + y;

    [Fact]
    public void Add_returns_expected_value()
    {
        var x = 1;
        var y = 2;

        var sut = Add(x,y);
        var expected = 3;

        Assert.Equal(expected, sut);
    }

***

# Be Explicit 
### Name Everything - No 'Magic Values'

---

## Too Magical

    [lang=cs]
    [Fact]
    public void DoWork_thows_when_work_type_invalid()
    {
        var sut = new Worker();

        var ex = Assert.Throws(() => sut.DoWork(1, 13));
    }
 
--- 

## Mortal

    [lang=cs]
    [Fact]
    public void DoWork_thows_when_work_type_invalid()
    {
        var userId = 1;
        var invalidWorkType = 13;

        var sut = new Worker();

        var ex = Assert.Throws(() => sut.DoWork(userId, invalidWorkType));
    }

***

# Test One Thing

---

## When Your Test Only Makes One Assertion
### You Know What Exactly Passed/Failed

---

## If This Test Fails,<br/> What Went Wrong?

    [lang=cs]
    public void DoesManyThings(int x, int y)
    {
        var z = myService.GetValue(x);
        myInstance.Property = z + y;
        myOtherService.DoAdd(myInstance);
    };

    [Fact]
    public void DoesManyThings_works_correctly()
    {
        // assert calls method on my service

        // assert state set on instance correctly

        // assert other service called with instance
    }

---

## If Any of These Tests Fail, We Know What Broke

    [lang=cs]
    public void DoesManyThings(int x, int y)
    {
        var z = myService.GetValue(x);
        myInstance.Property = z + y;
        myOtherService.DoAdd(myInstance);
    };

    [Fact]
    public void DoesManyThings_properly_calls_method_on_my_service()
    {
        // assert calls method on my service
    }
    [Fact]
    public void DoesManyThings_sets_state_on_instance_correctly()
    {
        // assert state set on instance correctly
    }
    [Fact]
    public void DoesManyThings_properly_calls_method_on_other_service_with_instance()
    {
        // assert other service called with instance
    }

*** 

# Tools!

--- 

### AutoFixture
### FakeItEasy
### AutoFixture + XUnit Plugin
### AutoFixture + FakeItEasy Plugin

***

## Using AutoFixture to Create Values and Instances
[See Cheat Sheet to Learn More](https://github.com/AutoFixture/AutoFixture/wiki/Cheat-Sheet)

---

## Using Create

    [lang=cs]
    var fixture = new Fixture();
    var name = fixture.Create<string>();
    var person = fixture.Create<Person>();

---

## Using Build

    [lang=cs]
    var fixture = new Fixture();
    var name = fixture.Create<string>();
    var person = fixture.Build<Person>()
        .With( p => p.Name, name);

---

## Customizing

    [lang=cs]
    var fixture = new Fixture();
    fixture.Register<string>(() => "asshat");
    string result = fixture.Create<string>(); // = 'asshat'

    var mc = fixture.Create<MyClass>();
    fixture.Customize<MyViewModel>(ob => ob
        .Do(x => x.AvailableItems.Add(mc))
        .With(x => x.SelectedItem, mc));

    var mvm = fixture.Create<MyViewModel>();

---

## Custom Fixtures

    [lang=cs]
    class MyFixture : Fixture
    {
        public MyFixture()
        {
            Customize<Person>( b => b
                .With(x => x.Name, "asshat"));
        }
    }

    var fixture = new MyFixture();

***

## Mocking with FakeItEasy
[See Quickstart to Learn More](http://fakeiteasy.readthedocs.io/en/stable/quickstart/)

---

## We Use Mocking to Control and/or Confirm Functionality

---

## Using Mocks To Control Functionality

    [Fact]
    public void Add_returns_expected_value()
    {
        var x = 1;
        var y = 2;
        var mockNumberService = A.Fake<INumberService>();

        A.CallTo(() => mockNumberService.GetX())
            .Returns(x);
        A.CallTo(() => mockNumberService.GetY())
            .Returns(y);

        // 
    }

---

## Using Mocks To Confirm Functionality

    [Fact]
    public void Add_returns_expected_value()
    {
        var x = 1;
        var y = 2;
        var mockNumberService = A.Fake<INumberService>();

        Adder.Add(mockNumberService);

        A.CallTo(() => mockNumberService.getX())
            .MustHaveHappened();
    }

***

## AutoFixture + XUnit Theories = AutoData

---

## Theories = Paramaterized Tests
<hr/>
## AutoData Supplies Random Values for Paramaters

---

## Think of Each Test as a Scene
<hr/>
### Starring Role (the SUT), </br>
### Supporting Roles,<br/> 
### and Extras
</hr>
### Extras = Good use of AutoData

---

    [lang=cs]
    [Fact]
    public void Add_returns_expected_value()
    {
        var x = 1; // extra
        var y = 2; // extra

        var sut = Add(x,y); // star
        var expected = x + y; // supporting

        Assert.Equal(expected, sut);
    }

---

    [lang=cs]
    [Theory, AutoData]
    public void Add_returns_expected_value(int x, int y)
    {
        var sut = Add(x,y);
        var expected = x + y;

        Assert.Equal(expected, sut);
    }

---  

TODO
custom autodata using custom fixtures

---

***

## Some Caveats when using AutoData

---

## The Entire Tree Structure
## Will be Hydrated
- This can be expensive if your tree structure is deep\*
- Any work kicked off in any constructors in the tree will be excuted\*

\* But you'd never do that, right!

***

## AutoFixture + FakeItEasy for Mocking

---

mocking with FakeItEasy

---

mocking with AutoFixture + FakeItEasy

---

Mocking with AutoData

***

# Questions? 

***

# Resources






