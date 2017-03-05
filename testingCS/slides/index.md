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
### Code that is Simple to Test

***

## Taking Some Notes From Functional Programming
- Embrace Functions
- Use More Functions
- Using Pure Functions
- Immutability

***

# Prefer Functions 
### Functions Always Return a Value

---

example of function returning a value making it easy to assert what happened

---

# Avoid Void
### Void Methods are Black Boxes

---

example of void method illustrating how we need to as somebody else what happened

***

# More Functions

### Prefer More Small Functions 
### That do One Thing

---

## Does Too Much

    [lang=cs]
    public string DoAllTheThings(int userId, bool isSober, int launchCode)
    {
        // example of complex method
    }

We have to test with every permutation of those params, in each possible state.
Contributes to code's [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity).

---

## Does One Thing

example of functionality broken up into functions

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

example of original process now using functions

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
### Your Function Might be Pure if
<p class="fragment fadeIn">Given the same input, you always produce the same output</p>
<p class="fragment fadeIn">Have no side-effects</p>

---

IO example

--- 

side effects example

---

pure example

***

# Immutability
## The Applaudable Fear of Change

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

## Keep noise out of the body of your test with AutoData

***

#AutoData

## Each Test is a Scene
### There are starring roles, supporting actors, and extras
### Extras = Noise, Good use of AutoData


