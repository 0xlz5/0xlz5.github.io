---
layout: post
title: "Temporal Drift — Code Appendix (Design Notes + Full Implementation)"
date: 2026-04-10 03:36:00 +0200
categories: [tech, code]
tags: [temporal-drift, appendix, code, python, functional-programming, design]
---

# Code Appendix — Design Notes, Architecture & Full Implementation

## 5 — Code Appendix: Design Notes, Architecture & Intent

This appendix contains the full implementation of the base‑conversion engine referenced throughout the series.  
It is intentionally over‑engineered — not for performance, but for **aesthetic clarity**, **functional purity**, and **narrative coherence** with the themes of *Temporal Drift*.

The code is a small, self‑contained artifact that demonstrates how ideas from:

- functional programming  
- monadic error handling  
- point‑free composition  
- recursion as structure  
- caching as a semantic guarantee  
- object‑oriented facades  
- and playful over‑engineering  

can coexist inside a single, compact Python module.

It is not meant to be “the best” way to convert integers between bases.  
It is meant to be **a miniature philosophy of code** — a crystallized moment of technical memory.

---

## 5.1 — Why this design?

The converter is built around a few guiding principles:

### **1. Functional purity as a constraint**
The core converter (`convert_pf`) is:

- pure  
- deterministic  
- referentially transparent  
- free of side effects  

This allows the entire module to behave like a mathematical object rather than a procedural script.

### **2. Recursion as narrative**
Instead of iterative loops, the digit extraction uses a **recursive collector**:

- it mirrors the mathematical definition of base decomposition  
- it produces a natural tree‑like structure  
- it keeps the code visually minimal  

Recursion is not used for performance — it is used for **shape**.

### **3. Point‑free composition**
Helpers like `compose`, `join`, and `with_sign` allow the final expression:

```
with_sign(sign)(join(collect(n)))
```


to read like a pipeline rather than a sequence of steps.

This is a nod to:

- Haskell  
- combinatory logic  
- Unix pipelines  
- and the “flow” aesthetic of functional code  

### **4. A tiny Maybe monad**
The `Maybe` class is intentionally microscopic:

- no inheritance  
- no type machinery  
- no decorators  
- just `bind`, `map`, and `unwrap`  

It exists to show how **error handling can be expressive without exceptions**.

### **5. A thin OO façade**
`BaseConverter` wraps the functional core in a small object:

- for ergonomics  
- for readability  
- for users who prefer method calls over free functions  

This duality (FP core + OO façade) mirrors the duality explored in the article:  
**two paradigms, one system**.

### **6. Decorators as shape‑shifters**
The `tupify` decorator is a playful trick:

- it transforms string output into tuples  
- without modifying the underlying logic  
- demonstrating how decorators can “bend” functions without touching them  

It’s a reminder that Python’s metaprogramming tools can be elegant when used sparingly.

### **7. Caching as a semantic guarantee**
`@lru_cache` is not used for speed.  
It is used to assert:

> “This function is pure. Its output depends only on its inputs.”

Caching becomes a **declaration of intent**, not an optimization.

---

## 5.2 — What this code is *not*

- It is not optimized for speed.  
- It is not meant for production.  
- It is not meant to replace Python’s built‑ins.  
- It is not meant to be minimal.  

It is meant to be:

- expressive  
- playful  
- precise  
- over‑designed in a deliberate way  
- a small artifact of technical memory  

A piece of code that exists because it **felt right**.

---

## 5.3 — What to look for when reading the code

When you scroll into the raw block below, notice:

- how the recursive collector builds digits  
- how point‑free helpers remove noise  
- how the monad wraps unsafe operations  
- how the OO façade hides the functional machinery  
- how decorators reshape output  
- how the entire module remains compact despite its layers  

This is code as **miniature architecture**, not just implementation.

---

## 5.4 — Full Implementation  
*(wrapped in `{% raw %}` to prevent Jekyll from interpreting it)*

{% raw %}
```python
from functools import lru_cache, reduce, wraps
from dataclasses import dataclass

# ---------- digit table ----------

DIGITS = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"


# ---------- ultra-short Maybe monad ----------

class Maybe:
    def __init__(self, v=None, n=False): self.v, self.n = v, n
    def bind(self, f): return self if self.n else f(self.v)
    def map(self, f):  return self if self.n else Maybe(f(self.v))
    unwrap = lambda self, d=None: self.v if not self.n else d

Just    = lambda x: Maybe(x, False)
Nothing = lambda:   Maybe(None, True)


# ---------- point-free helpers ----------

compose = lambda f, g: lambda x: f(g(x))
join    = lambda seq: reduce(lambda a, b: a + b, seq, "")
with_sign = lambda s: lambda x: s + x


# ---------- pure functional converter (point-free) ----------

@lru_cache
def convert_pf(n: int, base: int) -> str:
    if n == 0:
        return "0"

    sign = "-" if n < 0 else ""
    n = abs(n)

    # recursive digit collector
    def collect(k):
        return [] if k == 0 else collect(k // base) + [DIGITS[k % base]]

    # point-free: join ∘ collect
    return with_sign(sign)(join(collect(n)))


# ---------- monadic safe converter ----------

def safe_convert(n, base):
    if not isinstance(n, int) or not isinstance(base, int):
        return Nothing()
    if not (2 <= base <= 36):
        return Nothing()
    return Just(convert_pf(n, base))


# ---------- OO façade encapsulating everything ----------

@dataclass(frozen=True)
class BaseConverter:
    base: int

    def convert(self, n: int) -> str:
        return convert_pf(n, self.base)

    def maybe(self, n: int) -> Maybe:
        return safe_convert(n, self.base)

    __call__ = convert


# ---------- tupify decorator ----------

def tupify(fn):
    @wraps(fn)
    def wrapper(n):
        return tuple(fn(n))
    return wrapper


# ---------- tuple-producing helpers ----------

@tupify
def bits(n):
    return BaseConverter(2)(n)

@tupify
def octals(n):
    return BaseConverter(8)(n)

@tupify
def hexed(n):
    return BaseConverter(16)(n)

@tupify
def base_10(n):
    return BaseConverter(10)(n)


# ---------- demo ----------

if __name__ == "__main__":
    print(bits(-20))
    print(octals(10))
    print(hexed(40))
    print(base_10(-123456789))

    bc = BaseConverter(16)
    print(bc.maybe(255))      # Just('FF')
    print(bc.maybe("nope"))   # Nothing
```
{% endraw %}
