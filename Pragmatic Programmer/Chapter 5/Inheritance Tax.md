# Inheritance Tax

## What is Inheritance? 
It is a feature of OOP that allows a class to inherit behaviour and properties from another class.

```csharp
// Base class (or parent class)
public class Animal
{
    public void Eat()
    {
        Console.WriteLine("Eating...");
    }
}

// Derived class (or child class)
public class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine("Barking...");
    }
}
```

## Problems Using Inheritance to Share Code

### Inheritance Creates Coupling

```csharp
public class Vehicle
{
    protected int Speed;

    public void MoveAt(int speed)
    {
        Speed = speed;
    }

    public void Stop()
    {
        Speed = 0;
    }
}
```
When a class inherits from another, it gets:
- All of its methods (like move_at)
- All of its fields/variables (like @speed)
- And any future changes too — good or bad

So if you write:

```csharp
public class Car : Vehicle
{
    public string Info()
    {
        return $"I'm a car going at {Speed} km/h";
    }
}
```
Then car is now tightly coupled to the internal design of vehicle. 
Now if one day the developer who is in charge of `Vehicle` changes the API, so MOVE_AT becomes set_velocity and the instance variable @speed becomes @velocity.
An API change is expected to break clients of Vehicle class. But the top-level is
not: as far as it is concerned it is using a Car. What the Car class does in terms
of implementation is not the concern of the top-level code, but it still breaks.

Now our Car class (which relies on @speed) breaks silently. Why? 
- @speed is gone. 
- move_at method is gone. 
- Even the top-level code calling move_at on Car breaks.

## Problems Using Inheritance To Build Types
Some folks view inheritance as a way of defining new types. For instance:

```csharp
public class Vehicle { }

public class Car : Vehicle { }

public class Bicycle : Vehicle { }

public class Boat : Vehicle { }
```

At first, this seems fine. But then you get a requirement like:
- A Car is also an Asset (because it can be leased)
- A Car is also Insurable 
- A Car might also be LoanCollateral

```csharp
public class Asset { }
public class InsurableItem { }
public class LoanCollateral { }

public class Car : Vehicle, Asset, InsurableItem, LoanCollateral
```
C# or other OO languages does not support multiple inheritance of classes. Now You're now stuck. You can’t model this hierarchy correctly, even though conceptually it makes sense.

## Alternatives

Three techniques that mean you should never need to use inheritance again:
- Interfaces and protocols 
- Delegation 
- Mixins and traits

### Interfaces and Protocols
Most OO languages allow you to specify that a class implements one or more
sets of behaviors. You could say, for example, that a Car class implements the
Drivable behavior and the Locatable behavior. The syntax used for doing this varies:
in Java, it might look like this:

```csharp
public class Car implements Drivable, Locatable {
// ...
}
```

Drivable and Locatable are what Java calls interfaces; other languages call them
protocols, and some call them traits. 

What makes interfaces and protocols so powerful is that we can use them as
types, and any class that implements the appropriate interface will be compatible with that type. If Car and Phone both implement Locatable, we could store
both in a list of locatable items:

```csharp
List<Locatable> items = new ArrayList<>();
items.add(new Car(...));
items.add(new Phone(...));
items.add(new Car(...));
```
Interfaces and protocols give us polymorphism without inheritance.

### Delegation

It means Instead of doing it myself, I’ll ask someone else to do it.
Think of a manager (a class) that asks an assistant (another class) to handle tasks like booking meetings or sending emails.
In code, one class has a reference to another class and calls methods on it to get things done. That’s delegation.

```csharp
// This is the 'assistant' that knows how to save
public class AccountRepository
{
    public void Save(Account account)
    {
        Console.WriteLine($"Saving account: {account.Name}");
    }
}

// This is the business object that delegates saving to the repository
public class Account
{
    public string Name { get; set; }

    private AccountRepository _repository;

    public Account(string name)
    {
        Name = name;
        _repository = new AccountRepository();
    }

    public void Save()
    {
        _repository.Save(this); 
    }
}

```

### Enter Mixins / Traits / Extensions (a.k.a Composition Helpers)

In some languages (like Ruby, Scala, Kotlin), you can use mixins or traits to reuse behavior across classes without inheritance.

C# doesn’t have "mixins" natively, but it has something very similar: Extension Methods, Interfaces, and Partial Classes.

For instance: you want to display different features for different users:

- Customers need strict password rules.
- Admins do not. 
- Some operations skip password validation altogether.

If you used inheritance, you’d cram all the validations into the base class and toggle them with flags:

`if (isAdmin)
SkipPasswordValidation();
`

Instead

````csharp
public interface IValidation
{
    void Validate(Account account);
}

public class BasicAccountValidation : IValidation
{
    public void Validate(Account account)
    {
        // common validations
    }
}

public class CustomerValidation : IValidation
{
    public void Validate(Account account)
    {
        // extra customer validations
    }
}

public class AdminValidation : IValidation
{
    public void Validate(Account account)
    {
        // admin validations
    }
}
````

Then compose them:

```csharp
public class AccountValidator
{
    private readonly IEnumerable<IValidation> _rules;

    public AccountValidator(IEnumerable<IValidation> rules)
    {
        _rules = rules;
    }

    public void Validate(Account account)
    {
        foreach (var rule in _rules)
            rule.Validate(account);
    }
}

```
