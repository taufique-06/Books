# Implementing DBC
Simply enumerating at design time what the input domain range is, what the boundary conditions are, and what the routine promises to deliver—or, more importantly, what it does not promise to deliver is a huge leap forward in
writing better software. By not stating these things, you are back to programming by coincidence, which is where many projects start, finish, and fail.
In languages that do not support DBC in the code, this might be as far as you can go—and that’s not too bad. DBC is, after all, a design
technique. Even without automatic checking, you can put the contract in the code as comments and still get a very real benefit. If nothing else,
the commented contracts give you a place to start looking when trouble strikes.

## Assertions
While documenting these assumptions is a great start, you can get much greater benefit by having the compiler check your contract for
you. You can partially emulate this in some languages by using assertions. To begin with, in Object-Oriented language there is no support
for propagating assertions down an inheritance hierarchy. This means that if you override a base class method that has a contract, the assertions that implement that contract will not be called correctly.

## DBC and Crashing Early

DBC fits nicely with our concept of crashing early. By using an assert or DBC mechanisms to validate the preconditions,
post-conditions and invariants, you can crash early and report more accurate information about the problem.
