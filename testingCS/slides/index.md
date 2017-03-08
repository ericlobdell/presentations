- title : Elements of Testable Code
- description : Attributes and Techniques of Testable Code
- author : Eric Lobdell
- theme : Night
- transition : default

***

# Road to Simpler C# Testing

***

# We'll Discuss
## Code That's Easy to Test
## Elements of Simple Tests
## Tools

***

# Code That's Easy To Test

***

## Taking Some Notes From Functional Programming
<p>How Some Functional Concepts Can Aid Your Code's Testabliity</p>

***

# Prefer Functions 
*Functions: Expressions That Always Return a Value*

---

    [lang=cs]
    public string Add(int x, int y) => x + y;

    [Fact]
    public void Add_returns_expected_value()
    {
        var sum = Add(1,2);
        Assert.Equal(3, sum);
    }

---

# Avoid Void
<p>Void Methods are Black Boxes</p>
<p>You Have to Ask Someone Else What Happened</p>

---

    [lang=cs]
    public void Add(int x, int y)
    {
        myService.DoAdd(x,y);
    };

    [Fact]
    public void Add_calls_the_right_method_with_the_right_values()
    {
        var mockService = A.Fake<IMyService>();
        var x = 1;
        var y = 2;

        Add(x,y);

        A.CallTo(() => mockService.DoAdd(x,y))
            .MustHaveHappened();
    }

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

# Elements of Simple Tests

## Readibility
## Tests One Thing
 
***

***

# Readibility

***

## Avoid Mocking
### Mocking is a Layer of Indirection </br> Adds to Cognitive Load of Understanding the Test

---

example of test with lots of mocking

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

## Be Explicit - Name Everything
### No 'Magic Values'

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

## Sans Magic

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

    [lang=cs]
    public void Add(int x, int y)
    {
        var z = myService.GetValue(x);
        myInstance.Property = z + y;
        myOtherService.DoAdd(myInstance);
    };

    [Fact]
    public void Add_works_correctly()
    {
        // assert calls method on my service

        // assert state set on instance correctly

        // assert other service called with instance
    }

---

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
        // assert calls method on my service
    }

    [Fact]
    public void Add_sets_state_on_instance_correctly()
    {
        // assert state set on instance correctly
    }

    [Fact]
    public void Add_properly_calls_method_on_other_service_with_instance()
    {
        // assert other service called with instance
    }

*** 

# Tools!

--- 

### AutoFixture
### AutoFixture + XUnit
### AutoFixture + FakeItEasy

***

use autofixture to create values and instances

---

examples

---

customizing fixtures

***

AutoFixture + XUnit Theories = AutoData
(heart) AutoData

---

## Each Test is a Scene
### There are starring roles, supporting actors, and extras
### Extras = Noise, Good use of AutoData

---

example of using autodata

---  

custom autodata using custom fixtures

---

***

Caveats when using AutoData

---

## The Entire Tree Structure Will be Hydrated
- This can be expensive if your tree structure is deep\*
- There can be dependencies that may require setup before AF can properly hydrate
- Any work kicked off in any constructors in the tree will be excuted\*

\* But you'd never do that, right...

***

AutoFixture + FakeItEasy for Mocking

---

mocking with FakeItEasy

---

mocking with AutoFixture + FakeItEasy

---

Mocking with AutoData




