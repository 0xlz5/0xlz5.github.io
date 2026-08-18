---
layout: post
title: "Testing - Part 1 - From TDD to BDD"
date: 2026-08-18 04:03:00 +0200
categories:
  - software-engineering
  - architecture
  - testing
tags:
  - tdd
  - bdd
  - cucumber
  - gherkin
  - contracts
  - types
  - property-testing
  - python
  - rust
  - javascript
---
# From TDD to BDD

## Types, Contracts, Properties, Tests and Cucumber as Layers of Software Meaning

There is a useful way to look at modern software engineering that is slightly different from the usual "unit tests vs integration tests" taxonomy.

Instead of asking:

> What kind of test is this?

ask:

> **At what level are we expressing the meaning of the program?**

A modern application can express its intended behavior through:

* types
* annotations
* assertions
* contracts
* unit tests
* property-based tests
* integration tests
* BDD scenarios
* formal specifications

These aren't simply competing testing frameworks.

They are **layers of executable or machine-checkable specification**.

---

# 1. The semantic ladder

Consider a simple requirement:

> A bank account cannot have a negative balance.

There are many ways to express it.

### Test

```python
def test_balance_never_negative():
    ...
```

### Assertion

```python
assert account.balance >= 0
```

### Contract

```text
invariant:
    balance >= 0
```

### Type/model

```text
NonNegativeMoney
```

### Property

```text
for every valid sequence of operations:
    balance >= 0
```

### Formal specification

```text
prove that every reachable valid state
satisfies balance >= 0
```

They all concern the same domain fact.

The difference is **where the fact lives and how strongly it is expressed**.

---

# 2. TDD: tests as design feedback

TDD is often summarized as:

```text
RED → GREEN → REFACTOR
```

But the deeper idea is that the test becomes a feedback mechanism for design.

Suppose we start with:

```python
def test_deposit():
    account = Account()

    account.deposit(100)

    assert account.balance == 100
```

The test forces us to decide:

* What is an `Account`?
* How is it created?
* What does `deposit` mean?
* Is `balance` observable?
* What happens with invalid amounts?

Testing therefore becomes a form of **API design**.

This is one of TDD's greatest strengths.

---

# 3. TDD isn't synonymous with testing

This distinction is important.

You can have:

```text
excellent test suite
+
no TDD
```

and:

```text
TDD process
+
poorly designed tests
```

TDD describes **how development proceeds**.

Testing describes **how behavior is checked**.

Contracts describe **what must remain true**.

BDD describes **how behavior is communicated and specified**.

These are orthogonal concepts.

---

# 4. BDD: changing the vocabulary

BDD emerged partly from asking:

> What are we actually trying to test?

Instead of talking primarily about objects and methods, BDD talks about **behavior**.

The canonical structure is:

```text
Given
When
Then
```

For example:

```gherkin
Feature: Bank account

Scenario: Withdraw money
  Given an account with balance 100
  When I withdraw 30
  Then the balance should be 70
```

This is much closer to a domain specification than:

```python
account.withdraw(30)
assert account.balance == 70
```

The two may ultimately execute the same assertion.

But they communicate with different audiences.

---

# 5. Gherkin is a DSL

Gherkin is not itself the application.

It is a domain-oriented language for describing scenarios.

Conceptually:

```text
Gherkin
   ↓
Given / When / Then
   ↓
step definitions
   ↓
application
   ↓
assertions
```

This makes Cucumber interesting from a language-design perspective.

It is effectively a **small external DSL for behavioral specifications**.

The business vocabulary becomes executable.

---

# 6. But Cucumber isn't always a good idea

Cucumber is most valuable when multiple groups need to share a vocabulary:

```text
product
  ↕
domain experts
  ↕
developers
  ↕
test automation
```

For example:

```gherkin
Given a customer is eligible for a discount
When the customer purchases three qualifying products
Then the discount should be applied
```

That may communicate something important to people who don't care about the implementation.

But this:

```gherkin
Scenario: Addition

Given a calculator
When I add 2 and 2
Then the result should be 4
```

is probably pointless ceremony.

A unit test is clearer.

This leads to a useful rule:

> **BDD earns its complexity when the behavior itself is a shared domain language.**

---

# 7. BDD and TDD are not opposites

A common misconception is:

```text
TDD → developers
BDD → business
```

The reality is more continuous.

A team can use:

```text
BDD scenario
      ↓
acceptance criterion
      ↓
TDD unit tests
      ↓
implementation
```

For example:

```text
BDD:
Given an account with €100
When €30 is withdrawn
Then the balance is €70
```

can lead to:

```python
def test_withdrawal_reduces_balance():
    ...
```

and eventually:

```python
account.withdraw(30)
```

The BDD scenario describes **external behavior**.

The unit test describes **internal design behavior**.

---

# 8. Property testing sits between them

Property-based testing provides another fascinating bridge.

Example-based testing:

```text
deposit(100)
→ balance = 100
```

Property testing:

```text
for every amount > 0:
    deposit(amount)
    → balance increases by amount
```

BDD:

```text
Given an account
When money is deposited
Then the balance increases
```

Contract:

```text
postcondition:
balance = old balance + amount
```

These can all describe essentially the same business rule.

But their abstraction levels differ.

---

# 9. A four-way comparison

| Technique    | Unit of meaning       | Typical audience    |
| ------------ | --------------------- | ------------------- |
| Type         | value/state           | compiler/programmer |
| Contract     | invariant/obligation  | programmer/runtime  |
| Property     | general behavior      | programmer/tester   |
| Unit test    | concrete behavior     | programmer          |
| BDD scenario | domain behavior       | team/domain         |
| Formal proof | mathematical property | verifier/theorist   |

This suggests a broader architecture:

```text
             DOMAIN MODEL
                  │
        ┌─────────┴─────────┐
        │                   │
     DOMAIN               CODE
     LANGUAGE                │
        │                   │
      BDD                types/contracts
        │                   │
        └─────────┬─────────┘
                  │
              properties
                  │
                tests
                  │
             implementation
```

---

# 10. Python's unusual position

Python can support nearly every layer.

### Type layer

```python
def deposit(account: Account, amount: int) -> None:
    ...
```

### Assertion layer

```python
assert amount > 0
```

### Decorator layer

```python
@validate
@cached
def deposit(...):
    ...
```

### Unit-testing layer

```python
def test_deposit():
    ...
```

### Property-testing layer

```python
@given(st.integers(min_value=1))
def test_deposit(amount):
    ...
```

### BDD layer

With a BDD framework, the same behavior can become:

```gherkin
Given an account with balance 100
When I deposit 50
Then the balance should be 150
```

Python is therefore less a single methodology than a **toolkit for constructing methodologies**.

---

# 11. Rust takes a different path

Rust shifts more responsibility toward compile-time guarantees.

Instead of:

```text
test that an object cannot be in state X
```

you can sometimes design the types so that:

```text
state X cannot be constructed
```

This is a profound change.

Compare:

```text
dynamic language:

create invalid state
       ↓
run program
       ↓
test detects problem
```

with:

```text
Rust:

attempt to create invalid state
       ↓
compiler rejects program
```

Of course, not everything can be encoded in types.

That's why Rust still needs:

```text
#[test]
assert!
proptest
integration tests
```

But the balance shifts.

---

# 12. JavaScript/TypeScript takes another route

TypeScript gives JavaScript a static layer:

```typescript
function withdraw(
    account: Account,
    amount: number
): void
```

but the type system doesn't generally express domain invariants such as:

```text
amount > 0
balance >= 0
```

Therefore:

```text
TypeScript
    +
runtime validation
    +
unit tests
    +
property testing
```

is common.

The testing ecosystem compensates for the language's dynamic runtime.

---

# 13. Perl's lesson is still relevant

Perl's testing culture demonstrates something that remains useful:

> **A test does not need to be sophisticated to be valuable.**

A tiny:

```text
t/basic.t
```

file can prevent a regression.

A simple test harness can turn a collection of scripts into a reliable software component.

TAP is particularly elegant because the test output becomes a protocol:

```text
program
   ↓
TAP
   ↓
harness
   ↓
CI
```

This is an early and very pragmatic form of **machine-readable executable specification**.

---

# 14. The "executable specification" idea

The phrase that connects all these traditions is:

> **Executable specification.**

Consider:

```text
documentation
```

A human can read it, but the machine can't verify it.

Then:

```text
test
```

The machine can execute it.

Then:

```text
property
```

The machine can generate many cases.

Then:

```text
contract
```

The rule becomes part of the program.

Then:

```text
type
```

Some invalid states become unrepresentable.

Then:

```text
formal proof
```

The system attempts to establish the property mathematically.

So:

```text
human prose
    ↓
examples
    ↓
tests
    ↓
properties
    ↓
contracts
    ↓
types
    ↓
proof
```

This is not a strict progression.

It is a **spectrum of explicitness and enforceability**.

---

# 15. The best architecture is usually layered

A mature system might look like:

```text
                 USER / DOMAIN
                       │
                 BDD scenarios
                       │
                 acceptance tests
                       │
              ┌────────┴────────┐
              │                 │
        integration tests   domain tests
              │                 │
              └────────┬────────┘
                       │
                  unit tests
                       │
                properties
                       │
                 contracts
                       │
                    types
                       │
                    code
```

But you don't necessarily need every layer.

The art is deciding **where each invariant belongs**.

---

# 16. A KISS decision rule

A practical decision table is:

| If you need to express...  | Prefer...           |
| -------------------------- | ------------------- |
| Shape of data              | Type/annotation     |
| Simple local assumption    | Assertion           |
| Function obligation        | Contract            |
| Concrete example           | Unit test           |
| Many input combinations    | Property test       |
| Component interaction      | Integration test    |
| User-visible workflow      | BDD                 |
| Shared business vocabulary | Cucumber/Gherkin    |
| Impossible states          | Strong type system  |
| Mathematical guarantee     | Formal verification |

This avoids a common modern problem:

> using the biggest testing technology available simply because it exists.

---

# 17. One requirement, many representations

Take:

> "A customer cannot spend more than their available credit."

We could express it as:

### Type

```text
CreditLimit
```

### Contract

```text
postcondition:
remaining_credit >= 0
```

### Unit test

```python
def test_customer_cannot_exceed_credit():
    ...
```

### Property

```text
for every sequence of valid purchases:
    credit_used <= credit_limit
```

### BDD

```gherkin
Scenario: Customer exceeds credit limit
  Given a customer with €100 credit
  When the customer attempts to spend €120
  Then the transaction is rejected
```

### Formal specification

```text
∀ reachable states:
    credit_used ≤ credit_limit
```

The domain requirement hasn't changed.

Only its **representation** has changed.

---

# 18. This gives us a better definition of "testing"

Testing isn't necessarily:

> Run the program and see whether it breaks.

It can be understood more broadly as:

> **Make an intended property of a program explicit enough that some mechanism can challenge, check, enforce or prove it.**

Under that definition:

```text
type checking
assertions
contracts
unit tests
property tests
BDD
formal verification
```

are all related activities.

They simply have different levels of automation, abstraction and assurance.

---

# 19. The deeper trade-off

There is a recurring trade-off:

```text
more expressive specification
          ↑
          │
          │       formal proof
          │
          │     contracts
          │
          │   properties
          │
          │ unit tests
          │
          │ BDD scenarios
          │
          └────────────────────→
             implementation
             proximity
```

Highly formal specifications can be expensive.

Very concrete tests can be cheap but incomplete.

BDD can communicate beautifully but become verbose.

Types can be powerful but difficult to model.

Contracts can be elegant but require runtime or verification infrastructure.

The engineering problem is therefore not:

> "Which testing methodology wins?"

It is:

> **Which representation gives us the best correctness-to-complexity ratio for this particular invariant?**

---

# 20. Final synthesis

Perl, Eiffel, Ada, Python, Rust, JavaScript, TDD, BDD, Cucumber and property-based testing can initially look like unrelated pieces of software history.

They aren't.

They represent different attempts to answer one recurring question:

> **How do we express what a program is supposed to mean, and how much of that meaning can we make mechanically checkable?**

Perl emphasized pragmatic executable regression suites.

Eiffel emphasized contracts and invariants.

Ada emphasized strong typing and correctness.

TDD made tests part of the design loop.

BDD moved specification toward domain behavior.

Cucumber made those specifications executable through a DSL.

Python provides flexible annotations, decorators, assertions and testing frameworks.

Rust moves many correctness constraints into the type system.

Property-based testing generalizes examples into rules.

Formal methods push the same idea toward mathematical proof.

The resulting picture is not a ladder where one technology replaces another.

It is a **stack of semantic tools**:

```text
             WHAT DOES THE SYSTEM MEAN?
                        │
              ┌─────────┴─────────┐
              │                   │
           DOMAIN              PROGRAM
              │                   │
           BDD/Gherkin       types/contracts
              │                   │
              └─────────┬─────────┘
                        │
                    properties
                        │
                      tests
                        │
                       code
```

And perhaps the most useful engineering principle is the simplest one:

> **Put each piece of knowledge at the cheapest level that can express it reliably.**

If a type can guarantee it, use a type.

If a contract expresses it naturally, use a contract.

If it is an example, write a test.

If it is a general rule, use a property.

If it is a business workflow, consider BDD.

If it truly requires mathematical certainty, consider formal verification.

That makes testing less a collection of frameworks and more a **language for expressing software meaning**.
