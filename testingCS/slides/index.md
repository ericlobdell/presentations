- title : Elements of Testable Code
- description : Attributes and Techniques of Testable Code
- author : Eric Lobdell
- theme : Night
- transition : default

***

# Road to Simpler C# Testing

***

# We'll Discuss
## Elements of Testable Code
## Elements of Simple Tests
## Tools

***

# Elements of Testable Code:
<p class="fragment fadeIn">Code that is Predictible</p>
<p class="fragment fadeIn">Code that is Easy to Run in Isolation</p>
<p class="fragment fadeIn">Code that Doesn't require Much Setup.</p>

***

## Taking Some Notes From Functional Programming
<p>How Some Functional Concepts Influence Your Code's Testabliity</p>

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
        var mockService = Mock<MyService>();
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

    [lang=cs]
    public string Add(int x, int y) => x + y;

    [Fact]
    public void Add_returns_expected_value()
    {
        var sum = Add(1,2);
        Assert.Equal(3, sum);
    }

---

    [lang=cs]
    public void Add(int x, int y)
    {
        var z = myService.GetValue(x);
        myInstance.Property = z;
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

# Immutability
## The Applaudable Fear of Change

---

When you code changes the state of an object, that state needs to be tracked and tested.

***

# Elements of Simple Tests

***

# Readibility
 
***

## Avoid Mocking

### Pass Values Not Value Providers

Value providers require setup
Values just need to be set = Less noise

---

example using provider

--- 

same example using passed values

***

## Be Explicit - name everything

---

example with ambiguoius values

--- 

example with named values


***

*** 

# Tools

<p>AutoFixture</p>
<p>FakeItEasy</p>
<p>AutoFixture + XUnit</p>
<p>AutoFixture + FakeItEasy</p>

***

## Keep noise out of the body of your test with AutoData

***

#AutoData

## Each Test is a Scene
### There are starring roles, supporting actors, and extras
### Extras = Noise, Good use of AutoData


