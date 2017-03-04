- title : Elements of Testable Code
- description : Attributes and Techniques of Testable Code
- author : Eric Lobdell
- theme : Night
- transition : default

***

# Testing C#

***

# We'll Discuss
## Elements of Testable Code
## & Tools of the Trade

***

# Elements of Testable Code

***

## Taking Some Notes From Functional Programming
- Prefer Functions Over Methods
- Use More Functions
- Using Pure Functions
- Immutability

***

## Prefer Functions

# Avoid Void
## Void Methods are Black Boxes
By returning a value, you have a way to know/assert what happened

***

# More Functions

### Prefer More Small Functions 
### That do One Thing

---

## Does Too Much

    [lang=cs]
    public string DoAllTheThings(int userId, bool isSober, int launchCode)
    {
        var user = repo.Get(id);
        
        if (IsValid(launchCode))
        {
            user.CanLaunch = true;
        }

        if(user.CanLaunch)
    }

We have to test with every permutation of those params, in each possible state.
Contributes to code's [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity).

---

## Does One Thing

    [lang=cs]
    public string DoThisThing(string id, string launchCode)
    {
        // single piece of work
    }

    public string DoThatThing(string id, string launchCode)
    {
        // single piece of work
    }

---

## Allows Us To Do Complex Tasks 
## With Tested Code

    [lang=cs]
    public string DoAllTheThings(string id, string launchCode)
    {
        DoThisThing(id);
        DoThatThing(launchCode);
    }

 Now we only have to test that the right code is executed.</br>
 We've already confirmed it's behavior.

***

# Pure Functions
### You might be a pure function if
<p class="fragment fadeIn">Given the same input, you always produce the same output</p>
<p class="fragment fadeIn">Have no side-effects</p>

---

IO example

--- 

side effects example

***

### Pass Values When possible

Arguments that are complex objects can be in any number of possible states, all of which will need to be tested against.
Limiting your args to values means less state to manage = simpler tests

***

# Immutability
## The Applaudable Fear of Change

***

# Readibility

***

## Avoid Mocking
## Be Explicit - name everything
## Keep noise out of the body of your test with AutoData

***

#AutoData

## Each Test is a Scene
### There are starring roles, supporting actors, and extras
### Extras = Noise, Good use of AutoData


