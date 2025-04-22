# DRY - DONT REPEAT YOURSELF

It is about duplication of knowledge, of intent. It is about expressing the same thing in two different places, possibly in two different ways.

Here is the Acid test:

When some single facet of the code has to change,
do you find yourself making that change in multiple places and in multiple different formats? Do you have to change code and documentation or a db schema and a structure that holds.
If so, the code is not DRY. 

## Duplication in Code


```csharp 
def print_balance(account)
 printf "Debits: %10.2f\n", account.debits
 printf "Credits: %10.2f\n", account.credits
if account.fees < 0
 printf "Fees: %10.2f-\n", -account.fees
else
 printf "Fees: %10.2f\n", account.credits
end
 printf " ———-\n"
if account.balance < 0
 printf "Balance: %10.2f-\n", -account.balance
 else
 printf "Balance: %10.2f\n", account.balance
end
end

```

<b>Let's find the duplication</b>

First, there’s clearly a copy-and-paste duplication of handling the negative
numbers.

```csharp
def format_amount(value)
 result = sprintf("%10.2f", value.abs)
if value < 0
 result + "-"
else
 result + " "
end
end

//Updated print_balance
def print_balance(account)
 printf "Debits: %10.2f\n", account.debits
 printf "Credits: %10.2f\n", account.credits
 printf "Fees: %s\n", format_amount(account.fees)
 printf " ———-\n"
 printf "Balance: %s\n", format_amount(account.balance)
end



```

Another duplication is the repetition of the field width in all the printf calls. We
could fix this by introducing a constant and passing it to each call, but why
not just use the existing function?

```csharp
def format_amount(value)
 result = sprintf("%10.2f", value.abs)
if value < 0
 result + "-"
else
 result + " "
end
end
    
//Updated print_balance
def print_balance(account)
 printf "Debits: %s\n", format_amount(account.debits)
 printf "Credits: %s\n", format_amount(account.credits)
 printf "Fees: %s\n", format_amount(account.fees)
 printf " ———-\n"
 printf "Balance: %s\n", format_amount(account.balance)
end
```

Anything more? Well, what if the client asks for an extra space between the
labels and the numbers? We’d have to change 5 lines. Let’s remove that
duplication.

```csharp
def format_amount(value)
 result = sprintf("%10.2f", value.abs)
if value < 0
 result + "-"
else
 result + " "
end
end
def print_line(label, value)
 printf "%-9s%s\n", label, value
end
def report_line(label, amount)
 print_line(label + ":", format_amount(amount))
end
def print_balance(account)
 report_line("Debits", account.debits)
 report_line("Credits", account.credits)
 report_line("Fees", account.fees)
 print_line("", "———-")
 report_line("Balance", account.balance)
end

```

## Not All Code Duplication is Knowledge Duplication

As part of your online wine ordering application you’re capturing and validating
your user’s age, along with the quantity they’re ordering. According to the
site owner, they should both be numbers, and both greater than zero. So you
code up the validations:

```c++
def validate_age(value):
 validate_type(value, :integer)
 validate_min_integer(value, 0)
```
```c++
def validate_quantity(value):
 validate_type(value, :integer)
 validate_min_integer(value, 0)
```

The above is not DRY violation. The code is the same, but the knowledge they represent is
different. The two functions validate two separate things that just happen to
have the same rules. That’s a coincidence, not a duplication

### More on this!

In the example, we have two different concepts being validated:

`validate_age(value)`

`validate_quantity(value)`

The code seems looks identical. 

But the domain knowledge behind them is different:

1. Age is a concept with meaning in terms of legality, user verification, etc.

2. Quantity is about ordering items, stock, and so on.

So even though the implementation overlaps now, they might diverge in the future — maybe we will need to:

1. Allow quantity to be 0 (meaning “save for later”), or

2. Add age restrictions based on region or drink type.

### Why This Isn’t a DRY Violation
DRY is about avoiding duplicated knowledge — not just avoiding copy-pasted code. If you DRY things up too early based on coincidental similarities, you risk coupling unrelated logic.
That makes future changes harder because tweaking validation for quantity might accidentally mess with age logic, or vice versa.

### ✅ Best Practice
If the rules are truly shared because of shared knowledge (like "all user input values must be positive integers"), then you abstract. 
Otherwise, keep them separate — even if the code looks the same.


## Representational Duplication?
When the code talks to external systems (like APIs, databases, or services), we end up duplicating the knowledge of how those systems represent their data or behavior.

For example:
1. We write code that knows an API needs a `user_id` and `email`.

2. The API docs also describe that. 
3. Now, both our code and the external API hold the same “knowledge” — that’s representational duplication.

If the API changes, our code breaks — because it was built on duplicated assumptions.

## Inter-developer Duplication
Perhaps the hardest type of duplication to detect and handle occurs between
different developers on a project. Entire sets of functionality may be inadvertently duplicated, and that duplication could go undetected for years, leading
to maintenance problems.

We feel that the best way to deal with this is to encourage active and frequent
communication between developers.
Maybe run a daily scrum standup meeting. Set up forums (such as Slack
channels) to discuss common problems. This provides a nonintrusive way of
• 10
• Click HERE to purchase this book now. discuss
communicating—even across multiple sites—while retaining a permanent
history of everything said.
Appoint a team member as the project librarian, whose job is to facilitate the
exchange of knowledge. Have a central place in the source tree where utility
routines and scripts can be deposited. And make a point of reading other
people’s source code and documentation, either informally or during code
reviews. You’re not snooping—you’re learning from them. And remember, the
access is reciprocal—don’t get twisted about other people poring (pawing?)
through your code, either.

<b>What you’re trying to do is foster an environment where it’s easier to find and
reuse existing stuff than to write it yourself. If it isn’t easy, people won’t do
it. And if you fail to reuse, you risk duplicating knowledge.
</b>


