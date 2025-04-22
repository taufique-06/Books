# How do we express and enforce contracts in C#?

C# doesn't have built-in DbC like Eiffel, but we can implement it using standard tools and patterns:

## Option A: Using Debug.Assert() for Lightweight Contracts

Preconditions: Debug.Assert(price >= 0)

Post-conditions: Debug.Assert(discount <= price)

These only run in Debug mode — great during development, not enforced in production builds.

```csharp
using System.Diagnostics;

public decimal CalculateDiscount(decimal price, decimal percentage)
{
    Debug.Assert(price >= 0, "Price must be non-negative");
    Debug.Assert(percentage >= 0 && percentage <= 100, "Percentage must be between 0 and 100");

    var discount = price * (percentage / 100);

    Debug.Assert(discount <= price, "Discount cannot be more than the price");
    return discount;
}

```

## Option B: Throwing Exceptions for Runtime Contracts

```csharp
public decimal CalculateDiscount(decimal price, decimal percentage)
{
    if (price < 0)
        throw new ArgumentOutOfRangeException(nameof(price), "Price must be non-negative");

    if (percentage < 0 || percentage > 100)
        throw new ArgumentOutOfRangeException(nameof(percentage), "Percentage must be between 0 and 100");

    var discount = price * (percentage / 100);

    if (discount > price)
        throw new InvalidOperationException("Discount cannot exceed the price");

    return discount;
}
```

## Option C: Use Helper Methods

```csharp
public static class Contract
{
    public static void Requires(bool condition, string message)
    {
        if (!condition)
            throw new ArgumentException(message);
    }

    public static void Ensures(bool condition, string message)
    {
        if (!condition)
            throw new InvalidOperationException(message);
    }
}

public decimal CalculateDiscount(decimal price, decimal percentage)
{
    Contract.Requires(price >= 0, "Price must be non-negative");
    Contract.Requires(percentage >= 0 && percentage <= 100, "Invalid percentage");

    var discount = price * (percentage / 100);

    Contract.Ensures(discount <= price, "Discount too large");
    return discount;
}
```