# While You Are Coding

## How to Program Deliberately

1. Always be aware of what you are doing.
2. Can you explain the code in details to a more junior developer? If not, perhaps, you are relying on coincidences.
3. Dont code in the dark. Build an application you dont fully grasp, or use a technology you dont understand, you will likely to be bitten by coincidences. If you are not sure why it works, you wont know why it fails.
4. Processed from a plan
5. Rely on reliable things. Dont depend on assumptions. If you cant tell if something is reliable, assume the worst.
6. Document your assumptions.
7. Dont just test your code,  but test your assumptions as well. Dont guess actually try it. Write an assertions to test your assumptions. 
8. Prioritize your effort. Spend time on the important aspects; more than likely, these are the hard parts.
9. Dont be a slave to history. Dont let existing code dictate the future code.All code can be replaced if it is no longer appropriate. 

## Refactoring
### When Should You Refactor?

1. Duplication. You’ve discovered a violation of the DRY principle 
2. Non-orthogonal design. You’ve discovered some code or design
that could be made more orthogonal. 
3. Outdated knowledge. Things change, requirements drift, and your
knowledge of the problem increases. Code needs to keep up. 
4. Performance. You need to move functionality from one area of the
system to another to improve performance.

### Real-World Complications

So you go to your boss or client and say, “This code works, but I need
another week to refactor it.

We can’t print their reply.

Time pressure is often used as an excuse for not refactoring. But this
excuse just doesn’t hold up: fail to refactor now, and there’ll be a far
greater time investment to fix the problem down the road—when there
are more dependencies to reckon with. Will there be more time available
then? Not in our experience.

You might want to explain this principle to the boss by using a medical
analogy: think of the code that needs refactoring as a “growth.” Removing it requires invasive surgery. You can go in now, and take it out while
it is still small. Or, you could wait while it grows and spreads—but removing it then will be both more expensive and more dangerous. Wait
even longer, and you may lose the patient entirely.

### How Do You Refactor?

1. Don’t try to refactor and add functionality at the same time.
2. Make sure you have good tests before you begin refactoring. Run
   the tests as often as possible. That way you will know quickly if
   your changes have broken anything.
3. Take short, deliberate steps: move a field from one class to another,
   fuse two similar methods into a superclass. Refactoring often involves making many localized changes that result in a larger-scale
   change. If you keep your steps small, and test after each step, you
   will avoid prolonged debugging.

### Test to Code

A function or method that is tightly coupled to other code is hard to test, because you have to set up all that environment before you can even run the method.
So, making your stuff testable also reduces its coupling.

### Property-Based Testing

If you write the original code and you write the tests, is it possible that an incorrect assumption could be expressed in both? The code passes the tests, because it does what it is supposed to based on your understanding.
One way around to this is to have different people to write tests. Or we can ask the computer to do some testing for us.

#### What is property based testing? 
In normal unit tests, we pick a few specific inputs and check the output. But in the property based testing:
we describe the properties that should always be true, then the system would generate lots of random test data to try to break those properties.

For instance:
```csharp
//Traditional Unit Test
[Test]
public void Reverse_Twice_ReturnsOriginal()
{
    var input = "hello";
    var reversedTwice = new string(input.Reverse().ToArray().Reverse().ToArray());

    Assert.Equal(input, reversedTwice);
}

//You’re testing just one example: "hello".
```

```csharp
Property ReverseTwice_ReturnsOriginal => Prop.ForAll<string>(input =>
{
    var reversedTwice = new string(input.Reverse().ToArray().Reverse().ToArray());
    return input == reversedTwice;
});

//This will run hundreds of randomized strings (including edge cases like null, empty strings, emoji, etc.) to see if the property holds.
```

### When Should You Use Property-Based Testing?

1. There’s a general rule that must always hold true, regardless of input. 
`e.g., x + y = y + x, or sort(x) is always in ascending order.`
2. Function is Pure(No side effects), Stateless and Input -> Output.
3. You want to test many edge cases with minimal effort. Nulls, empty strings, special characters, large inputs, etc.
4. You want to avoid “example bias”. Example-based unit tests often don’t cover weird edge cases.
5. You’re working with:
   - Math 
   - Parsing 
   - Sorting 
   - Converters (e.g., JSON <-> Object)
   - Encoders/Decoders 
   - Data transformation logic