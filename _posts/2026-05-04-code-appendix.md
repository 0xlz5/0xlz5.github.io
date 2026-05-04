---
layout: post
title: "Temporal Drift — Code Appendix"
date: 2026-05-04 03:36:00 +0200
categories: [tech, code, narrative]
tags: [temporal-drift, appendix, code, python, functional-programming, design]
---

# Code Appendix

---

# 5A — Intent, Philosophy & Context  
*Why this code exists at all*

This appendix is not a “utility module”.  
It is a **technical artifact**, a crystallized moment of how ideas, aesthetics, and habits converge into a single piece of code.

It exists because:

- over‑engineering can be a form of play  
- functional purity can be calming  
- recursion can be expressive  
- monads can be tiny and beautiful  
- Python can be bent into shapes it was never meant to hold  
- technical memory deserves artifacts  

This code is a **snapshot** of a particular way of thinking —  
a moment in the drift.

It is not optimized.  
It is not minimal.  
It is not industrial.

It is **intentional**.

---

# 5B — Architecture Overview  
*What the system is made of*

The module is composed of seven conceptual layers:

1. **Digit Table**  
   A simple string of characters representing digits for bases 2–36.

2. **Maybe Monad**  
   A microscopic error‑handling abstraction with `bind`, `map`, and `unwrap`.

3. **Point‑Free Helpers**  
   Small combinators (`compose`, `join`, `with_sign`) that remove noise.

4. **Functional Core**  
   `convert_pf` — a pure, recursive, cached base converter.

5. **Safe Converter**  
   `safe_convert` — wraps the functional core in a monadic safety layer.

6. **OO Façade**  
   `BaseConverter` — a thin ergonomic wrapper with `__call__`.

7. **Decorators & Tuple Helpers**  
   `tupify` + `bits`, `octals`, `hexed`, `base_10`.

8. **Demo Section**  
   A small runnable block showing usage.

This is not a “stack”.  
It is a **constellation** — each piece independent, but aligned.

---

# 5C — Functional Core  
*The heart of the machine*

The core function, `convert_pf`, is:

- pure  
- recursive  
- referentially transparent  
- cached  
- point‑free in its final expression  

The recursive collector:
```
def collect(k):
return [] if k == 0 else collect(k // base) + [DIGITS[k % base]]
```


is intentionally naïve —  
not for performance, but for **shape**.

The final expression:

```
with_sign(sign)(join(collect(n)))
```


is a miniature pipeline.

This is functional programming **as aesthetic discipline**.

---

# 5D — The Maybe Monad & Error Semantics  
*How the system handles uncertainty*

Instead of exceptions, the module uses a tiny `Maybe` monad:

- `Just(x)` represents success  
- `Nothing()` represents failure  

This allows:

- safe composition  
- predictable control flow  
- no try/except noise  
- no implicit state  

The monad is intentionally microscopic —  
a reminder that expressive error handling does not require heavy machinery.

`safe_convert` becomes a semantic wrapper:

- type checks  
- base range checks  
- returns `Just(result)` or `Nothing()`  

This is error handling as **explicit structure**, not side effect.

---

# 5E — OO Façade, Decorators & Shape‑Shifting  
*How the system presents itself to the outside world*

The `BaseConverter` class is a thin ergonomic wrapper:

- stores a base  
- exposes `convert`  
- exposes `maybe`  
- is callable (`__call__`)  

It exists for users who prefer method calls over free functions.

The `tupify` decorator is a playful shape‑shifter:

- it transforms string output into tuples  
- without modifying the underlying logic  

The tuple‑producing helpers (`bits`, `octals`, `hexed`, `base_10`) demonstrate how decorators can **bend** functions without touching them.

This is interface design as **lightweight metamorphosis**.

---

# 5F — Full Implementation  

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
