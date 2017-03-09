- title : Elements of Testable Code
- description : Attributes and Techniques of Testable Code
- author : Eric Lobdell
- theme : Night
- transition : default

***

# Simpler C# Testing

--- 

# The Value of Testing
## Tests = Production Code

---

## Complex vs Simple

---

### A Complex Test Requires a Lot of Setup in the Body of the Test, Which Creates a lot of 'Noise'

---

    [lang=cs]
    [Fact]
    public void ServiceWriteService_Create_CallsDomain()
    {
      Fixture fixture = GetFixture();
      var sut = GetSut();
      var action = fixture.Create<CreateServiceAction>();
      var identity = fixture.Create<TestIdentity>();

      A.CallTo(() => ident.Identity).Returns(identity);
      A.CallTo(() => serviceService.Get(action.Name, identity.CompanyId)).Returns(null);

      var domainAction = new DomainActions.CreateAction()
      {
        CompanyId = identity.CompanyId,
        Name = action.Name
      };

      var likeness = new Likeness<DomainActions.CreateAction, 
        DomainActions.CreateAction>(domainAction);

      sut.Create(action);

      A.CallTo(() => serviceService.CreateService(A<DomainActions.CreateAction>
        .That.Matches(a => likeness.Equals(a))))
        .MustHaveHappened();
    }

--- 

### A Simpe Test Requires Less Setup, and Keeps as Much Noise out of the Body of the Test as Possible

---

### A Simple Test
    [lang=cs]
    [Theory, AutoIdentityServerData]
    public void TryRegisterSession_returns_true_when_no_token_cached( [Frozen]ICacheService<string,string> cacheService, MemoryCache blackList, string newToken, string userId)
    {
      var sut = new SessionManagementService(cacheService, blackList);

      A.CallTo(() => cacheService.Get(userId))
        .Returns(null);

      var result = sut.TryRegisterSession(userId, newToken);

      Assert.True(result);
    }

---

## The Value of Simple Tests
</hr>

- Easy to Read
- Easy to See What's Being Tested
- Easy to Debug
- Easy to Maintain/Modify
- More Reliable

***

## We Get Simpler Tests By
- Adopting Certain Coding Habits
- Using the Right Tools

***

# Coding for Simpler Tests

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

### You're Free to Return Whatever You Need to Confirm Behavior!

<hr>

### Write Code Around Your Tests,</br> Not Tests Aroud Your Code

***

# Pure Functions

- Given the same input, always produce the same output
- Stateless
- Have no side-effects

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

---

## Pure Functions => </br> Fewer Moving Parts => </br> Simpler Tests

***

# Avoid Mocking

---

### Mocking is a Layer of Indirection </br> Which Adds Complexity to Your Test

---

### Pass Values, Not 'Value Providers'

---

## Value Providers Require Setup = Noise

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

### Values Just Need to be Set = Less Noise

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

---

# Bonus
### Forcing the Call Site to Provide the Values Helps Push/Limit This Kind of Complexity to the Boundary of Your Application

***

## Writing Tests That are Easy to Understand
 
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

---

## Take Every Opportunity to Add Meaning to the Elements of Your Tests
<hr>
## Strive for Obvious

***

# Test One Thing

---

## When Your Test Only Makes One Assertion
### You Know Exactly What Passed/Failed

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

---

## A Unit is a Scenario
<hr>
## Isolate a Single Scenario in Your Tests

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

    // creating a value type
    var name = fixture.Create<string>();

    // creating a reference type
    var person = fixture.Create<Person>();

---

## Using Build to Specify Values

    [lang=cs]
    var fixture = new Fixture();

    var name = "yo mamma";

    var person = fixture.Build<Person>()
        .With( p => p.Name, name);

---

## Customizing the Fixture

---

### Custom Value Types

    [lang=cs]
    var fixture = new Fixture();

    // Now every string created will = 'asshat'!
    fixture.Register<string>(() => "asshat");
    
    string result = fixture.Create<string>(); // = 'asshat'

---

### Custom Reference Types

    [lang=cs]
    var fixture = new Fixture();
    var item = 123

    fixture.Customize<MyViewModel>(ob => ob
        .Do(x => x.AvailableItems.Add(item))
        .With(x => x.SelectedItem, item));

    /// mvm with values set from builder
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

## Custom AutoData Using Custom Fixtures

---

    [lang=cs]
    public class AutoFieldForwardData : AutoDataAttribute
    {
        public AutoFieldForwardData()
            : base(new FieldForwardFixture()) { }
    }

    [Theory, AutoFieldForwardData]
    public void Add_returns_expected_value(int x, int y)
    {
        // Generated data uses any customizations made 
        // on FieldForwardFixture
    }

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

## Creating a Mock With FakeItEasy

    [lang=cs]
    var mockNumberService = A.Fake<INumberService>();

    A.CallTo(() => mockNumberService.GetX())
        .Returns(x);

---

## Creating a Mock With AutoFixture + FakeItEasy

    [lang=cs]
    var fixture = new Fixture()
        .Customize(new AutoFakeItEasyCustomization());

    var mockNumberService = fixture.Freeze<INumberService>();

    A.CallTo(() => mockNumberService.GetX())
        .Returns(x);

[Learn About "Freeze" Here](http://blog.ploeh.dk/2010/03/17/AutoFixtureFreeze/)

---

## What's the Difference?

<hr>

Nothing

---

## So Why is This Valuable?

<hr>

We Only Use One Tool's Vocabluary/Idioms, Which Adds Consistency, Which is Very Valuable

---

## Unlock Moar Power!

<hr>

## Custom AutoData + FakeItEasy

---

    [lang=cs]
    public class AutoFieldForwardData : AutoDataAttribute
    {
        public AutoFieldForwardData()
            : base(new FieldForwardFixture()) 
        {
            Customize(new AutoFakeItEasyCustomization());
            ...
        }
    }

---

    [lang=cs]
    [Theory, AutoFieldForwardData]
    public void TryRegisterSession_caches_new_token_when_no_token_cached_for_user_id(
        [Frozen]ICacheService<string,string> cacheService, 
        MemoryCache blackList, 
        string newToken, 
        string userId)
    {
      var sut = new SessionManagementService(cacheService, blackList);

      A.CallTo(() => cacheService.Get(userId))
        .Returns(null);

      var result = sut.TryRegisterSession(userId, newToken);

      A.CallTo(() => cacheService.AddOrReplace(userId, newToken))
        .MustHaveHappened();
    }

***

## With Great Power Comes Great Responsibility

---

## These Tools Make it Easy to Abstract Away the Wrong Thing
<hr>
## Be Mindful to Keep All Relavent Information in the Body of Your Tests

***

# Explore
<hr>
### These Tools can do Much More Than What I've Covered

***

# Questions? 

***

# Resources






