---
layout: post
title: "Testing - Part 1 - From Regression Tests to TDD"
date: 2026-08-18 04:05:00 +0200
categories:
  - software-engineering
  - testing
  - programming-languages
tags:
  - perl
  - eiffel
  - ada
  - python
  - rust
  - tdd
  - testing
  - contracts
  - tap
  - quickcheck
---
# From Regression Tests to TDD

## Perl, Eiffel, Ada, Python, Rust and the Rise of Executable Specifications

There is a common tendency to tell the history of software testing as if it naturally led to modern Test-Driven Development:

> unit tests → TDD → BDD → property testing → modern software engineering.

The actual history is much more interesting.

Long before **TDD** became a named methodology, programmers were already writing regression tests, assertions, test harnesses and executable specifications. At roughly the same time, other programming-language traditions were asking a different question:

> Can correctness be expressed *inside the program itself*?

Eiffel answered with **Design by Contract**.

Ada answered with strong typing, assertions and eventually contracts.

Perl developed a remarkably pragmatic automated testing culture.

Python combined dynamic programming with annotations, decorators, assertions and powerful testing frameworks.

Rust moved even more correctness into the type system.

And modern property-based testing asks yet another question:

> Instead of testing a few examples, can we describe a property that should hold for an entire class of inputs?

These aren't competing inventions so much as different points in a large design space.

---

# 1. Testing existed before TDD

Testing software is considerably older than the term **Test-Driven Development**.

The basic idea of a regression test is simple:

```text
change program
     ↓
run known tests
     ↓
did previously working behavior break?
```

Unix development already had test scripts, build systems and regression testing traditions before TDD acquired its modern meaning.

The important distinction is that **testing** and **TDD** are not synonymous.

Testing asks:

> Does this program behave correctly?

TDD adds a development discipline:

```text
write failing test
       ↓
write minimal implementation
       ↓
make test pass
       ↓
refactor
       ↓
repeat
```

The test therefore isn't merely a verification artifact.

It becomes part of the **design process**.

---

# 2. Perl: a particularly interesting pre-TDD culture

Perl is one of the best examples of a language whose ecosystem developed a strong testing culture without that culture necessarily being TDD in the modern sense.

Perl modules conventionally developed a test suite alongside the module:

```text
MyModule/
├── lib/
│   └── MyModule.pm
├── t/
│   ├── 01-basic.t
│   ├── 02-edge-cases.t
│   └── 03-errors.t
└── Makefile.PL
```

The `t/` directory became almost a cultural convention.

Tests could emit **TAP — the Test Anything Protocol**:

```text
1..4
ok 1 - object can be created
ok 2 - object has a name
ok 3 - name can be changed
not ok 4 - invalid name is rejected
```

A test harness could consume the result without needing to know the implementation language.

That is a remarkably important idea:

> **The test result becomes a language-independent protocol.**

The Perl testing ecosystem therefore established a pattern that looks surprisingly modern:

```text
module
   ↓
tests
   ↓
test harness
   ↓
machine-readable result
   ↓
continuous integration / distribution
```

This was fundamentally about **automated regression and executable examples**.

It wasn't necessarily TDD.

---

# 3. TDD adds a temporal dimension

The difference is subtle but fundamental.

A conventional test suite can be written:

```text
implementation
      ↓
tests
```

TDD deliberately reverses the order:

```text
test
 ↓
failure
 ↓
implementation
 ↓
success
 ↓
refactoring
```

The test becomes a **design instrument**.

This gives us a useful distinction:

| Practice            | Main question                                          |
| ------------------- | ------------------------------------------------------ |
| Regression testing  | Did I break something?                                 |
| Unit testing        | Does this component behave correctly?                  |
| TDD                 | Can tests drive the design?                            |
| BDD                 | Does the implementation realize the intended behavior? |
| Property testing    | Does a general property hold?                          |
| Formal verification | Can the property be proved?                            |

They overlap heavily, but they aren't the same thing.

---

# 4. Eiffel: correctness as a contract

Eiffel takes a very different route.

With **Design by Contract**, the programmer can express:

* preconditions
* postconditions
* invariants

Consider a bank account:

```eiffel
deposit (amount: INTEGER)
    require
        amount > 0
    do
        balance := balance + amount
    ensure
        balance = old balance + amount
end
```

The method isn't merely accompanied by a test.

It contains a **behavioral contract**.

The precondition says:

```text
the caller must provide a positive amount
```

The postcondition says:

```text
the operation must increase the balance by exactly that amount
```

An invariant might say:

```text
balance >= 0
```

The difference from a unit test is profound.

A test says:

```text
deposit(100)
→ balance becomes 100
```

A postcondition describes a general relationship:

```text
for every valid call,
balance_after = balance_before + amount
```

The contract is therefore closer to a **property** than to an example.

---

# 5. Invariants are not really TDD

An invariant such as:

```text
balance >= 0
```

is not equivalent to:

```python
def test_balance_is_never_negative():
    ...
```

The test samples executions.

The invariant describes a condition that should hold whenever the object is in a valid state.

This puts Eiffel closer to:

```text
runtime verification
       +
executable specification
```

than to TDD.

A useful conceptual distinction is:

```text
TDD:
"Show me an execution where this works."

Contract:
"State the condition that must hold."

Formal verification:
"Prove that the condition always holds."
```

---

# 6. Ada: types and contracts

Ada comes from a related but somewhat different tradition.

Its strong static type system already allows programmers to express many constraints at the type level.

Modern Ada also supports preconditions, postconditions and type invariants.

Conceptually:

```ada
procedure Withdraw
  (Amount : Money)
with
  Pre  => Amount > 0,
  Post => Balance = Balance'Old - Amount;
```

This gives us another axis of software correctness:

```text
ordinary code
     ↓
type constraints
     ↓
contracts
     ↓
runtime checks
     ↓
tests
```

Ada therefore demonstrates an important idea:

> **Some things that we test in a dynamic language can instead become constraints enforced by the language or compiler.**

---

# 7. Python: an unusually flexible middle ground

Python doesn't impose one philosophy.

Instead it provides mechanisms that can be combined.

## Type annotations

```python
def deposit(account: Account, amount: int) -> None:
    ...
```

Annotations primarily describe intended types.

Static tools such as type checkers can then inspect them.

Conceptually:

```text
annotation
    ↓
static constraint
    ↓
tool verification
```

But Python remains dynamically executable even when an annotation is violated.

---

# 8. Decorators as executable metadata

Python decorators make things more interesting.

A function can be annotated with behavior:

```python
@cached
@validated
@transactional
def deposit(account, amount):
    ...
```

A decorator can intercept the call and implement:

* caching
* validation
* authorization
* logging
* tracing
* transactions
* retries
* contracts

This makes decorators a convenient mechanism for building a lightweight Design-by-Contract system.

Conceptually:

```python
@pre(lambda account, amount: amount > 0)
@post(lambda account, amount, result: account.balance >= 0)
def deposit(account, amount):
    ...
```

Now the function has an executable behavioral specification surrounding it.

Python therefore has an interesting combination:

```text
annotations → structural specification

decorators → behavioral metadata

assertions → local runtime constraints

pytest → executable examples

Hypothesis → executable properties
```

---

# 9. pytest: the TDD side of Python

A normal pytest test is an executable example:

```python
def test_deposit():
    account = Account()

    account.deposit(100)

    assert account.balance == 100
```

This is extremely close to classic TDD.

The test describes a concrete example of desired behavior.

But pytest can also move toward broader specifications.

For example:

```python
@pytest.mark.parametrize(
    "amount",
    [1, 10, 100, 1000]
)
def test_deposit(amount):
    account = Account()

    account.deposit(amount)

    assert account.balance == amount
```

Now a family of examples is being described.

That leads naturally to property-based testing.

---

# 10. Property-based testing

QuickCheck introduced a powerful alternative to example-based testing.

Instead of:

```text
for this input:
    expected output = X
```

you describe:

```text
for a whole class of inputs:
    property P must hold
```

Python's Hypothesis, Rust's `proptest`, and JavaScript's `fast-check` bring this idea into their respective ecosystems.

Conceptually:

```python
@given(st.integers(min_value=1))
def test_deposit(amount):
    account = Account()

    account.deposit(amount)

    assert account.balance == amount
```

The important transition is:

```text
example
   ↓
many examples
   ↓
general property
```

This is much closer to Eiffel's postconditions than ordinary unit testing is.

---

# 11. Rust pushes correctness toward the type system

Rust takes yet another approach.

A normal Rust test looks familiar:

```rust
#[test]
fn deposit_increases_balance() {
    let mut account = Account::new();

    account.deposit(100);

    assert_eq!(account.balance(), 100);
}
```

That's ordinary unit testing and can naturally be used with TDD.

But Rust's type system allows another strategy.

Instead of repeatedly testing:

```text
amount > 0
```

you might introduce a type representing a valid amount:

```rust
struct PositiveAmount(u64);
```

Now the program architecture can make invalid states harder to represent.

This gives a useful Rust principle:

> **Move correctness from runtime checks into compile-time constraints whenever practical.**

The resulting stack looks something like:

```text
Rust types
     ↓
compile-time guarantees

#[test]
     ↓
examples

proptest / quickcheck
     ↓
properties

assert!
     ↓
runtime contracts
```

Rust therefore combines several traditions rather than choosing just one.

---

# 12. JavaScript and TypeScript

JavaScript is more dynamic, so its ecosystem relies heavily on runtime testing.

A typical TypeScript project might combine:

```text
TypeScript
     ↓
static type information
     ↓
Vitest / Jest
     ↓
unit + integration tests
     ↓
fast-check
     ↓
property testing
```

A test might look like:

```typescript
test("deposit increases balance", () => {
    const account = new Account();

    account.deposit(100);

    expect(account.balance).toBe(100);
});
```

TypeScript gives:

```typescript
function deposit(
    account: Account,
    amount: number
): void
```

But `number` does not express:

```text
amount > 0
```

Consequently, runtime validation and testing remain important.

---

# 13. The big picture

We can now see several different mechanisms for expressing software meaning:

```text
                 SOFTWARE MEANING
                        │
        ┌───────────────┼────────────────┐
        │               │                │
       TYPES          CONTRACTS        TESTS
        │               │                │
        ↓               ↓                ↓
   compile-time     runtime rules     examples
        │               │                │
        │               │          ┌─────┴─────┐
        │               │          │           │
        │               │       examples   properties
        │               │          │           │
        └───────────────┴──────────┴───────────┘
```

There is no single "best" level.

Each expresses a different kind of knowledge.

---

# 14. A useful spectrum

One way to understand the whole landscape is:

```text
MORE FORMAL / MACHINE-CHECKABLE
            ↑
            │
      static types
            │
      dependent/refined types
            │
        contracts
            │
    property-based tests
            │
       unit tests
            │
          BDD
            │
      acceptance scenarios
            ↓
MORE HUMAN / DOMAIN-ORIENTED
```

This isn't a strict hierarchy.

A unit test can be extremely valuable.

A contract can be poorly designed.

A type system can encode the wrong model.

A Cucumber scenario can be useless ceremony.

The real question is:

> **Which part of the intended behavior belongs at which level?**

---

# 15. The important distinction: examples vs properties

This distinction explains much of the evolution.

### Example

```text
deposit(100)
→ balance = 100
```

### Property

```text
for every amount > 0:
    balance_after = balance_before + amount
```

### Contract

```text
postcondition:
balance = old balance + amount
```

### Type-level constraint

```text
PositiveAmount
```

### Formal specification

```text
prove that every valid execution
preserves the invariant
```

These are different degrees and forms of **executable or machine-checkable meaning**.

---

# 16. So where does TDD actually fit?

TDD isn't really another type of assertion.

It is primarily a **development workflow**.

You can use:

```text
TDD + pytest
TDD + Rust tests
TDD + Jest
TDD + property testing
TDD + contracts
```

TDD tells you *when and why* to write the test.

The testing framework tells you *how* to execute it.

The contract system tells you *which invariant* to enforce.

The type system tells you *which states should be impossible*.

This distinction is easy to lose in modern terminology.

---

# 17. A compact taxonomy

| Concept             | What it expresses         | Typical mechanism     |
| ------------------- | ------------------------- | --------------------- |
| Type                | Valid structure/state     | compiler/type checker |
| Assertion           | Local condition           | runtime               |
| Contract            | Behavioral obligation     | pre/post/invariant    |
| Unit test           | Concrete behavior         | test framework        |
| Regression test     | Previously known behavior | test suite            |
| TDD                 | Development process       | tests first           |
| Property test       | General behavior          | generated inputs      |
| BDD                 | Domain behavior           | scenarios             |
| Cucumber            | Executable scenario DSL   | Gherkin + bindings    |
| Formal verification | Mathematical correctness  | proof system          |

The important observation is that **TDD, BDD, contracts and property testing aren't mutually exclusive**.

They operate at different dimensions.

---

# 18. From Perl to modern testing

The historical progression can therefore be represented approximately as:

```text
Unix regression testing
        │
        ▼
Perl automated test culture
        │
        ├── t/*.t
        ├── Test::Harness
        └── TAP
        │
        ▼
unit-testing ecosystems
        │
        ▼
XP / TDD
        │
        ├── red
        ├── green
        └── refactor
        │
        ├───────────────┐
        ▼               ▼
property testing       BDD
        │               │
        ▼               ▼
 QuickCheck          Gherkin
 Hypothesis          Cucumber
 proptest            scenarios
 fast-check
```

Meanwhile, another lineage developed alongside it:

```text
Eiffel
  │
  ▼
Design by Contract
  │
  ├── preconditions
  ├── postconditions
  └── invariants
  │
  ▼
runtime contracts
```

And another:

```text
Ada
  │
  ▼
strong typing
  │
  ▼
contracts / invariants
```

And another:

```text
ML / Haskell / Rust
       │
       ▼
strong static type systems
       │
       ▼
make invalid states harder to represent
```

These traditions eventually converge in modern software engineering.

---

# 19. The deeper common idea

Despite their differences, all of these techniques attack the same fundamental problem:

> **How can we turn the programmer's intention into something a machine can check?**

There are several answers.

```text
"I know what this value is."
        ↓
type

"I know what must be true here."
        ↓
assertion

"I know what this operation promises."
        ↓
contract

"I know what this example should do."
        ↓
test

"I know what must hold for every generated input."
        ↓
property

"I know what the user expects."
        ↓
BDD scenario

"I can mathematically establish it."
        ↓
formal proof
```

Seen this way, software testing isn't merely about finding bugs.

It is part of a much larger movement:

> **making software semantics explicit.**

---

# 20. Practical conclusion

For a modern project, there is no need to choose one ideology.

A very pragmatic stack might be:

```text
TypeScript / Python / Rust
          │
          ▼
       types
          │
          ▼
     small contracts
          │
          ▼
     unit tests
          │
          ▼
   property tests where useful
          │
          ▼
 integration tests
          │
          ▼
BDD/Cucumber only where
domain communication benefits
```

The important rule is **KISS**:

Don't write a Cucumber scenario for something that a three-line unit test explains better.

Don't write a runtime test for something the type system can make impossible.

Don't introduce a contract framework when an assertion is enough.

Don't write fifty examples when a property captures the invariant.

And don't use TDD mechanically if the design is still exploratory.

The strongest approach is therefore not:

> **TDD vs contracts vs types vs BDD**

but:

> **Use the cheapest mechanism that expresses the intended invariant clearly enough.**

That is the common thread connecting Perl's tiny `t/*.t` files, Eiffel's contracts, Python decorators, Rust's types, Hypothesis/proptest, and Cucumber's Given/When/Then scenarios.
