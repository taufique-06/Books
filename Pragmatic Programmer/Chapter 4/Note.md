# Pragmatic Paranoia

## DBC (Design By Contract)

It is a simple yet powerful technique that focuses on documenting (and agreeing to) the rights and responsibilities of software modules to ensure program correctness. What is a
correct program? One that does no more and no less than it claims
to do. Documenting and verifying that claim is the heart of Design by
Contract (DBC, for short).

1. <b>Preconditions: </b> What must be true in order for the routine to be
called; the routine’s requirements. A routine should never get called
when its preconditions would be violated. It is the caller’s responsibility to pass good data.

2. <b>Post-conditions:</b> What the routine is guaranteed to do; the state of
the world when the routine is done. The fact that the routine has
a post-condition implies that it will conclude: infinite loops aren’t
allowed.

3. <b>Class invariants:</b> A class ensures that this condition is always
true from the perspective of a caller. During internal processing of
a routine, the invariant may not hold, but by the time the routine
exits and control returns to the caller, the invariant must be true.
(Note that a class cannot give unrestricted write-access to any data
member that participates in the invariant.)

To summarize above, ask yourself:

```csharp
- What should the caller promise bfore calling a method?
- What does the method gurantee after it runs?
- What state the system must always maintain (invariant)?
```

For example:

```
decimal CalculateDiscount(decimal price, decimal percentage)

For the above method:

// Precondition: price ≥ 0, percentage is between 0 and 100
// Postcondition: returned discount ≤ price
// Invariant: system never allows negative price
```

The contract between a routine and any potential caller can thus be read as

`If all the routine’s preconditions are met by the caller, the routine shall
guarantee that all postconditions and invariants will be true when it
completes.`

If either party fails to live up to the terms of the contract, then a remedy (which was previously agreed to) is invoked—an exception is raised,
or the program terminates, for instance. Whatever happens, make no
mistake that failure to live up to the contract is a bug. It is not something that should ever happen, which is why preconditions should not
be used to perform things such as user-input validation.

<b>Note:</b>

In Orthogonality, we recommended writing “shy” code. Here,
the emphasis is on “lazy” code: be strict in what you will accept before
you begin, and promise as little as possible in return. Remember, if your
contract indicates that you’ll accept anything and promise the world in
return, then you’ve got a lot of code to write!


That’s the mental shift:

You stop thinking only about “what does this code do?”

and start asking “what does this code require and guarantee?”
```
DBC and Test-Driven Development

Is Design by Contract needed in a world where developers practice unit testing, test-driven development, property based testing, or defensive programming?

The Answer is "Yes".

DBC and testing are different approaches to the boarder topic of program correctness.
They both have value and both have uses in different situations. DBC offers several advantages over specific testing approaches:

1. DBC doesnt require any setup or mocking.
2. DBC defines the parameters for success or failure in all cases, whereas testing can only target one specific case at a time. 
3. TDD and other testing happens only at "test time" within the build cycle. But DBC and assertions are forever: during design, development, deployment, and maintenance.
4. TDD does not focus on checking internal invariants within the code under test, it's more black-box style to check the public interface.
5. DBC is more efficient than defensive programming where everyone has to validate data in case no one else does.
```

## Implementing DBC

## How do we express and enforce contracts in C#?

## Assertive Programming

It seems that there’s a mantra that every programmer must memorize early in his or her career. It is a fundamental tenet of computing, a core
belief that we learn to apply to requirements, designs, code, comments, just about everything we do. It goes

`THIS CAN NEVER HAPPEN`

`This code won’t be used 30 years from now, so two-digit dates are fine.`
`This application will never be used abroad, so why internationalize it?`
`Count ca not be negative. This logging can not fail.`

Let’s not practice this kind of self-deception, particularly when coding.

Don’t use assertions in place of real error handling. Assertions check for things that should never happen: you don’t want to be writing code such as:

### Bad Example
```csharp
printf("Enter 'Y' or 'N': ");
ch = getchar();
assert((ch == 'Y') || (ch == 'N'));
```

Here’s why it’s bad:

1. User input is not an assumption — it's external and unpredictable. 
2. If the user enters 'X', the program crashes... not a good user experience.

Instead

```csharp
Console.WriteLine("Enter Y or N:");
var input = Console.ReadKey().KeyChar;

if (input != 'Y' && input != 'N')
{
    Console.WriteLine("Invalid input. Please enter 'Y' or 'N'.");
    //Ask user input again or sth like that
}
```

### Good Example

`Debug.Assert(index >= 0 && index < items.Count, "Index out of bounds");`


## Questions:

Which of these impossible things can happen?
1. A month with fewer than 28 days
    </br >
    September 1752 had only 19 days,
   
2. Error code from the system memory: Cant access the current directory
   </br >
   The directory could have been removed by another process; I probably dont have permission to read; The drive might not be mounted;
3. In C++: a=2; b=3; but (a+b) does not equal to 5
   </br >
   The type of 'a' and 'b' is not specified. Operator Overloading:C++ allows you to change how operators like + (addition) work for custom types (like classes). This is called operator overloading. For example, if a and b were objects of a class, the + operator might have been redefined to do something different, like combining the objects in a non-numeric way.
   If your program is running on a multi-threaded system (or multiple processors), it’s possible that the values of a and b could change while you’re doing the addition. This could lead to unexpected results because another process or thread might be modifying a or b at the same time.
4. A triangle with an interior angle sum != 180
   </br >
    In non-Euclidean geometry, the sum of the angles of a triangle will not add up to 180 degree. 
5. A min that does not have 60 seconds
   </br >
    Leap mins may have 61 or 62 secs.
6. (a+1) <= a
   </br >
   1. Overflow or Underflow:
      </br >
   </br >
   If a is a fixed-size integer (like int), the operation (a + 1) could cause overflow (for a very large value of a) or underflow (for a very small value of a). Overflow and underflow happen when the result of an operation goes beyond the range that the data type can represent. 
   For example, in languages like C++ or C#, integers typically have a fixed range of values:
`For a signed 32-bit integer, the range is from -2,147,483,648 to 2,147,483,647.` If a is at the maximum value (2,147,483,647) and you do (a + 1), it will overflow and wrap around to the minimum value (-2,147,483,648).
   2. Floating-Point Precision
      </br >
      </br >
      If a is a floating-point number (like float or double), it could also happen due to precision errors in representing numbers. Floating-point numbers are approximations, so adding 1 to a might not give you exactly a + 1 due to rounding errors. If a is large enough, adding 1 might not change the number at all.


## How to Balance Resources