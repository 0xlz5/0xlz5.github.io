---
layout: post
title: "Temporal Drift — Code Appendix (Sample Code)"
date: 2026-04-10 03:36:00 +0200
categories: [tech, code]
tags: [temporal-drift, appendix, code, python, functional-programming]
---

# Code Appendix — Sample Code  
*Untouched, exactly as written*

<!-- Copy button -->
<button id="copy-btn" style="
  padding:6px 12px;
  font-size:14px;
  cursor:pointer;
  margin-bottom:10px;
">
Copy Code
</button>

<!-- Code block container -->
<div id="code-block">

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
