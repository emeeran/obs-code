---
title: Python Reference — Concepts & Commands
tags: [python, programming, reference, cheatsheet]
created: 2026-10-15
python_version: "3.9+ (modern features flagged with the version that introduced them)"
---

# Python Reference — Concepts & Commands

A comprehensive overview of Python: language concepts, the standard library, and CLI tooling. Tables list `name` — what it does. Version notes like **(3.10)** mark the release that introduced a feature; assume everything else is ≥3.9.

**Baseline assumption:** modern CPython 3.9+. Where a feature needs 3.10/3.11/3.12/3.13, it is called out — check `python3 -V` before relying on it.

---

## Table of Contents

- [[#1. Language Fundamentals|1. Language Fundamentals]]
- [[#2. Built-in Types|2. Built-in Types]]
    - [[#Type checking & conversion|Type checking & conversion]]
- [[#3. Numbers & Math|3. Numbers & Math]]
    - [[#Formatting numbers|Formatting numbers]]
- [[#4. Strings|4. Strings]]
    - [[#Methods|Methods]]
    - [[#Slicing|Slicing]]
    - [[#Formatting|Formatting]]
- [[#5. Lists|5. Lists]]
- [[#6. Tuples & Records|6. Tuples & Records]]
- [[#7. Dictionaries|7. Dictionaries]]
    - [[#Iteration patterns|Iteration patterns]]
- [[#8. Sets|8. Sets]]
- [[#9. Collections Module|9. Collections Module]]
- [[#10. Control Flow|10. Control Flow]]
    - [[#Structural pattern matching **(3.10)**|Structural pattern matching **(3.10)**]]
- [[#11. Comprehensions & Generators|11. Comprehensions & Generators]]
- [[#12. Functions|12. Functions]]
    - [[#Argument unpacking|Argument unpacking]]
- [[#13. Functional Built-ins|13. Functional Built-ins]]
- [[#14. Iterators & Itertools|14. Iterators & Itertools]]
- [[#15. functools|15. functools]]
- [[#16. Object-Oriented Python|16. Object-Oriented Python]]
    - [[#Dunder (magic) methods|Dunder (magic) methods]]
- [[#17. Typing & Type Hints|17. Typing & Type Hints]]
- [[#18. Exceptions & Error Handling|18. Exceptions & Error Handling]]
    - [[#Built-in exception hierarchy (excerpt)|Built-in exception hierarchy (excerpt)]]
- [[#19. Context Managers|19. Context Managers]]
- [[#20. Decorators|20. Decorators]]
- [[#21. Files, I/O & Pathlib|21. Files, I/O & Pathlib]]
    - [[#`open()`|`open()`]]
    - [[#`pathlib`|`pathlib`]]
    - [[#Other file tools|Other file tools]]
- [[#22. Standard Library Map|22. Standard Library Map]]
    - [[#Regex quick reference (`re`)|Regex quick reference (`re`)]]
    - [[#`datetime` essentials|`datetime` essentials]]
- [[#23. Concurrency & Parallelism|23. Concurrency & Parallelism]]
- [[#24. Testing|24. Testing]]
- [[#25. Packaging & Environments|25. Packaging & Environments]]
- [[#26. Python CLI Commands|26. Python CLI Commands]]
    - [[#Package & environment tooling|Package & environment tooling]]
    - [[#Quality & static analysis|Quality & static analysis]]
    - [[#Common third-party libraries|Common third-party libraries]]
- [[#27. Debugging & Introspection|27. Debugging & Introspection]]
- [[#28. Idioms & Common Recipes|28. Idioms & Common Recipes]]
- [[#29. Version Feature Notes|29. Version Feature Notes]]
- [[#30. Common Pitfalls|30. Common Pitfalls]]
- [[#Appendix A: Format Spec Mini-Language|Appendix A: Format Spec Mini-Language]]
- [[#Appendix B: Exit Codes & `sys.exit`|Appendix B: Exit Codes & `sys.exit`]]
- [[#Appendix C: `__main__` Package Layout|Appendix C: `__main__` Package Layout]]
- [[#Appendix D: The Zen of Python|Appendix D: The Zen of Python]]
- [[#Appendix E: Quick Sanity Commands|Appendix E: Quick Sanity Commands]]

---


## 1. Language Fundamentals

| Concept | Description |
|---|---|
| Dynamic typing | Types are attached to values, not names; a name can rebind to any type |
| Strong typing | No implicit coercion: `"1" + 1` raises `TypeError` |
| Indentation | Blocks are defined by indentation (4 spaces, PEP 8), not braces |
| Comments | `# line`, and docstrings `"""..."""` for documentation |
| Statements vs expressions | Everything is a statement unless it produces a value |
| Names & binding | `x = 1` binds a name to an object; no declaration needed |
| Truthiness | `0`, `0.0`, `""`, `[]`, `{}`, `()`, `set()`, `None`, `False` are falsy |
| Immutability | `int`, `float`, `str`, `tuple`, `frozenset`, `bytes` cannot be mutated |
| Reference semantics | Assignment and argument passing copy *references*, not objects |
| Everything is an object | Functions, classes, and modules are first-class values |
| Duck typing | Behaviour over inheritance: "if it walks like a duck…" |
| `__name__ == "__main__"` | Guard so a module's demo code only runs when executed directly |
| LEGB scope | Name lookup: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in |
| `global` / `nonlocal` | Declare assignment to an outer scope's name |
| Mutable default trap | Default args are evaluated once — never use `[]`/`{}` as defaults |

## 2. Built-in Types

| Type         | Description                                      |
| ------------ | ------------------------------------------------ |
| `int`        | Arbitrary-precision integer                      |
| `float`      | IEEE-754 double; use `decimal.Decimal` for money |
| `complex`    | Complex number, `3+4j`                           |
| `bool`       | Subclass of `int`; `True == 1`, `False == 0`     |
| `str`        | Immutable Unicode text sequence                  |
| `bytes`      | Immutable byte sequence (binary data)            |
| `bytearray`  | Mutable byte sequence                            |
| `memoryview` | Zero-copy view over a buffer                     |
| `list`       | Ordered, mutable, heterogeneous sequence         |
| `tuple`      | Ordered, immutable sequence                      |
| `dict`       | Hash map; ordered by insertion since 3.7         |
| `set`        | Unordered, unique, mutable hash set              |
| `frozenset`  | Immutable, hashable set                          |
| `range`      | Lazy arithmetic sequence                         |
| `NoneType`   | The single value `None`                          |
| `Ellipsis`   | The `...` literal, used in stubs and slicing     |
| `type`       | Metaclass: the type of a class                   |
|              |                                                  |
|              |                                                  |
```python
# list — ordered, mutable
nums = [1, 2, 3]
nums.append(4)          # [1, 2, 3, 4]
nums[0] = 10            # [10, 2, 3, 4]

# tuple — ordered, immutable
point = (3, 5)
x, y = point            # unpacking
# point[0] = 9          # TypeError

# dict — key→value, mutable, insertion-ordered
user = {"name": "Em", "age": 30}
user["age"] = 31
print(user.get("email", "n/a"))   # "n/a" if missing

# set — unordered, unique, mutable
tags = {"a", "b", "a"}  # {"a", "b"}
tags.add("c")
other = {"b", "d"}
print(tags & other)     # {"b"}

# frozenset — like set, but immutable
perms = frozenset({"read", "write"})
# perms.add("admin")    # AttributeError
print("read" in perms)  # True

# range — lazy sequence of integers, immutable
r = range(2, 10, 2)     # 2, 4, 6, 8
print(list(r))          # [2, 4, 6, 8]
print(r[1])             # 4
```

Rule of thumb: list = ordered collection you edit; tuple = fixed record; dict = lookup by key; set/frozenset = uniqueness/membership (mutable/immutable); range = counting loop (`for i in range(5):`).


### Type checking & conversion

| Expression | Result |
|---|---|
| `type(x)` | Exact class of `x` |
| `isinstance(x, T)` | Prefer this — respects subclasses and tuples of types |
| `issubclass(A, B)` | Class inheritance test |
| `int("42")` / `float("3.14")` / `str(42)` | Casting conversions |
| `bool(x)` | Truthiness |
| `list(x)` / `tuple(x)` / `set(x)` / `dict(pairs)` | Collection conversion |
| `bytes("hi", "utf-8")` | Encode to bytes |
| `ord('a')` / `chr(97)` | Codepoint ↔ character |
| `hex(n)` / `oct(n)` / `bin(n)` | Base conversion to string |
| `hash(x)` | Hash value, if hashable |
| `id(x)` | Identity (memory address-like) |
| `dir(x)` | Attribute names available on an object |
| `vars(x)` / `x.__dict__` | Instance attribute mapping |
| `callable(x)` | Is `x` callable? |

## 3. Numbers & Math

| Item | Description |
|---|---|
| `+ - * /` | Arithmetic; `/` always returns a float |
| `//` | Floor division |
| `%` | Modulo / remainder |
| `**` | Exponentiation |
| `divmod(a, b)` | Tuple `(a // b, a % b)` |
| `abs(x)` | Absolute value |
| `round(x, n)` | Round to `n` places (banker's rounding) |
| `min()` / `max()` | Extremes; accept `key=` |
| `sum(iterable, start)` | Total |
| `pow(a, b, m)` | Modular exponentiation |
| `math.sqrt/floor/ceil/trunc` | Math primitives |
| `math.pi` / `math.e` / `math.tau` | Constants |
| `math.isnan` / `math.isinf` | Float specials |
| `math.fsum` | Accurate float summation |
| `math.lcm` / `math.gcd` **(3.9)** | Least common multiple / greatest common divisor |
| `math.comb` / `math.perm` **(3.8)** | Combinatorics |
| `math.isqrt` **(3.8)** | Integer square root |
| `math.log/exp/sin/cos/tan` | Logarithms and trigonometry |
| `math.dist(p, q)` **(3.8)** | Euclidean distance |
| `decimal.Decimal` | Exact decimal arithmetic (money) |
| `fractions.Fraction` | Exact rational numbers |
| `statistics.mean/median/mode/stdev` | Descriptive statistics |
| `statistics.fmean` **(3.8)** | Fast float mean |
| `complex(x, y)` / `cmath` | Complex numbers and their functions |

### Formatting numbers

| Spec | Meaning |
|---|---|
| `f"{x:.2f}"` | 2 decimal places |
| `f"{x:,}"` | Thousands separators |
| `f"{x:,.2f}"` | Both |
| `f"{x:08.3f}"` | Zero-padded to width 8 |
| `f"{x:e}"` | Scientific notation |
| `f"{x:%}"` | Percentage |
| `f"{x:#x}"` | Hex with `0x` prefix |
| `f"{x:b}"` | Binary |
| `f"{x:+d}"` | Force sign |

## 4. Strings

### Methods

| Method | Description |
|---|---|
| `s.lower()` / `s.upper()` | Case conversion |
| `s.casefold()` | Aggressive casefold for comparisons |
| `s.strip(c)` | Trim whitespace or given chars from both ends |
| `s.lstrip()` / `s.rstrip()` | Trim one side |
| `s.split(sep, maxsplit)` | Split into a list |
| `s.rsplit()` | Split from the right |
| `s.splitlines()` | Split on line boundaries |
| `sep.join(iterable)` | Join strings |
| `s.replace(old, new, count)` | Replace occurrences |
| `s.find(sub)` / `s.rfind` | Index or `-1` |
| `s.index(sub)` | Index or raise `ValueError` |
| `s.count(sub)` | Occurrences |
| `s.startswith(p)` / `s.endswith(p)` | Prefix/suffix test (accepts tuples) |
| `s.removeprefix(p)` **(3.9)** | Strip leading prefix if present |
| `s.removesuffix(p)` **(3.9)** | Strip trailing suffix if present |
| `s.zfill(n)` | Zero-pad on the left |
| `s.center(w, c)` / `ljust` / `rjust` | Padding |
| `s.title()` / `s.capitalize()` | Title-case / capitalise |
| `s.swapcase()` | Invert case |
| `s.isdigit()` / `isalpha()` / `isalnum()` | Character-class tests |
| `s.isspace()` / `isupper()` / `islower()` | More predicates |
| `s.partition(sep)` | `(before, sep, after)` tuple |
| `s.expandtabs(n)` | Expand tab characters |
| `s.encode(enc)` / `b.decode(enc)` | Encoding boundary |
| `s.format(**kw)` | Legacy formatting |
| `s.translate(table)` / `str.maketrans` | Character translation |

### Slicing

| Slice | Meaning |
|---|---|
| `s[0]` / `s[-1]` | First / last character |
| `s[2:5]` | Index 2 up to (not including) 5 |
| `s[:5]` / `s[5:]` | Prefix / suffix |
| `s[::2]` | Every second character |
| `s[::-1]` | Reversed string |
| `s[start:stop:step]` | General slice form |

### Formatting

| Style | Example |
|---|---|
| f-string **(3.6)** | `f"{name} is {age}"` — preferred |
| Nested f-string | `f"{value:{width}.2f}"` |
| Conversion flags | `f"{x!r}"` repr, `f"{x!s}"` str, `f"{x!a}"` ascii |
| Debug specifier **(3.8)** | `f"{x=}"` → `x=42` |
| `str.format` | `"{0} and {1}".format(a, b)` |
| `%`-formatting | `"%s → %d" % (a, b)` (legacy) |
| Multi-line f-string **(3.12)** | Same quote reuse / multi-line expressions allowed |
| Raw string | `r"C:\path"` — no escape processing |
| Byte string | `b"data"` |
| Triple-quoted | `"""multi\nline"""` |

## 5. Lists

| Operation | Description |
|---|---|
| `lst[i]` / `lst[-1]` | Index, negatives from the end |
| `lst[a:b]` | Slice (returns a new list) |
| `lst.append(x)` | Add one item to the end |
| `lst.extend(it)` | Append all items from an iterable |
| `lst.insert(i, x)` | Insert at index |
| `lst.remove(x)` | Remove first equal value (raises if absent) |
| `lst.pop(i)` | Remove and return item (default: last) |
| `lst.clear()` | Remove all items |
| `lst.index(x)` | First index of a value |
| `lst.count(x)` | Occurrences |
| `lst.sort(key=, reverse=)` | In-place sort (stable) |
| `sorted(lst)` | Return a new sorted list |
| `lst.reverse()` | In-place reverse |
| `reversed(lst)` | Reverse iterator |
| `lst.copy()` | Shallow copy |
| `lst + lst2` | Concatenate (new list) |
| `lst * 3` | Repeat |
| `x in lst` | Membership (linear time) |
| `len(lst)` | Length |
| `zip(a, b)` | Pairwise iteration |
| `enumerate(lst, start=0)` | Index/value pairs |
| `list(range(5))` | Build from a range |
| `copy.deepcopy(lst)` | Nested-safe copy |
| `lst[i:j] = [...]` | Slice assignment (changes length) |
| `del lst[i]` / `del lst[i:j]` | Delete by index/slice |

**Copy semantics:** `a = b` aliases; `b = a[:]` or `a.copy()` is a shallow copy; nested objects are still shared until `copy.deepcopy`.

## 6. Tuples & Records

| Item | Description |
|---|---|
| `t = (1, 2, 3)` | Tuple literal (parens optional in many places) |
| `t = 1,` | Single-element tuple — the trailing comma matters |
| `a, b = b, a` | Swap via tuple unpacking |
| `a, *rest = seq` | Starred unpacking |
| `t[0]` | Indexing, as with lists |
| `t.count(x)` / `t.index(x)` | The only two methods |
| `tuple(iterable)` | Convert to tuple |
| `hash(t)` | Hashable if all elements are hashable |
| `namedtuple` | `collections.namedtuple("P", "x y")` — lightweight record |
| `NamedTuple` | Typed, class-based variant from `typing` |
| Use as dict key | Tuples are hashable and can key a dict |

## 7. Dictionaries

| Operation | Description |
|---|---|
| `d[key]` | Lookup (raises `KeyError` if missing) |
| `d.get(key, default)` | Safe lookup |
| `d[key] = value` | Insert or overwrite |
| `d.setdefault(key, default)` | Get, inserting a default if absent |
| `d.update(other)` | Merge in another mapping |
| `d.pop(key, default)` | Remove and return |
| `d.popitem()` | Remove and return the last `(k, v)` |
| `d.clear()` | Empty the dict |
| `d.keys()` / `.values()` / `.items()` | Views over contents |
| `key in d` | Membership test on keys |
| `len(d)` | Number of entries |
| `d \| other` **(3.9)** | Merge, right side wins |
| `d \|= other` **(3.9)** | In-place merge |
| `{**a, **b}` | Merge via unpacking (older idiom) |
| `dict.fromkeys(keys, v)` | Build from a key list |
| `copy.deepcopy(d)` | Nested-safe copy |
| `del d[key]` | Delete an entry |

**Insertion order** is guaranteed from 3.7 onward; `Counter`, `defaultdict`, and `OrderedDict` live in `collections`.

### Iteration patterns

```python
for k, v in d.items(): ...           # both
for k in d: ...                       # keys
for v in d.values(): ...              # values
{k: v for k, v in pairs}              # comprehension
{k: v for k, v in d.items() if v > 0} # filtered
sorted(d.items(), key=lambda kv: kv[1])   # by value
max(d, key=d.get)                     # key with largest value
```

## 8. Sets

| Operation | Description |
|---|---|
| `s = {1, 2, 3}` | Set literal |
| `set(iterable)` | Build a set (deduplicates) |
| `frozenset(iterable)` | Immutable, hashable set |
| `s.add(x)` | Add an element |
| `s.discard(x)` | Remove if present (no error) |
| `s.remove(x)` | Remove, raising if absent |
| `s.pop()` | Remove an arbitrary element |
| `a \| b` | Union |
| `a & b` | Intersection |
| `a - b` | Difference |
| `a ^ b` | Symmetric difference |
| `a.union/intersection/difference` | Method forms (accept any iterable) |
| `a.issubset(b)` / `a.issuperset(b)` | Containment |
| `a.isdisjoint(b)` | No common elements |
| `a.update(b)` | In-place union |
| Set comprehension | `{x for x in seq if cond}` |

Sets give **O(1)** membership — the standard fix for "is this in a big list?".

## 9. Collections Module

| Class | Description |
|---|---|
| `Counter` | Multiset / tally; `.most_common(n)` |
| `defaultdict(factory)` | Dict with auto-created missing values |
| `OrderedDict` | Explicitly order-aware dict (rarely needed now) |
| `deque` | Double-ended queue; `appendleft`, `popleft`, `maxlen` |
| `namedtuple` | Tuple subclass with named fields |
| `ChainMap` | Layered view over multiple mappings |
| `UserDict` / `UserList` / `UserString` | Cleaner subclassing bases |

```python
from collections import Counter, defaultdict, deque
Counter("abracadabra").most_common(2)        # [('a', 5), ('b', 2)]
g = defaultdict(list); g["a"].append(1)      # no KeyError
q = deque(maxlen=3)                          # bounded ring buffer
```

## 10. Control Flow

| Statement | Description |
|---|---|
| `if` / `elif` / `else` | Conditional branching |
| Ternary | `x if cond else y` |
| `while` | Loop while a condition holds |
| `for x in iterable` | Iterate any iterable |
| `while … else` / `for … else` | `else` runs if the loop wasn't `break`-ed |
| `break` | Exit the innermost loop |
| `continue` | Skip to the next iteration |
| `pass` | No-op placeholder |
| `match` / `case` **(3.10)** | Structural pattern matching |
| Walrus `:=` **(3.8)** | Assign inside an expression |
| `raise` | Raise an exception |
| `try` / `except` / `else` / `finally` | Exception handling |
| `with` | Context manager (resource cleanup) |
| `assert cond, msg` | Debug assertion (stripped under `-O`) |
| `del` | Delete a name or item |
| `yield` | Produce a value from a generator |
| `import` / `from … import` | Module imports |

### Structural pattern matching **(3.10)**

```python
match command.split():
    case ["go", direction]:              # sequence pattern
        move(direction)
    case ["drop", *items]:               # starred
        drop(*items)
    case {"action": "quit"}:             # mapping pattern
        quit()
    case Point(x=0, y=0):                # class pattern
        origin()
    case str() | bytes() as text:         # or-pattern + capture
        print(text)
    case _:                              # wildcard (default)
        unknown()
```

## 11. Comprehensions & Generators

| Form | Description |
|---|---|
| `[f(x) for x in it]` | List comprehension |
| `[f(x) for x in it if cond]` | With a filter |
| `[f(x) if c else g(x) for x in it]` | Conditional expression |
| `[f(x, y) for x in a for y in b]` | Nested loops (cartesian) |
| `{k: v for k, v in pairs}` | Dict comprehension |
| `{f(x) for x in it}` | Set comprehension |
| `(f(x) for x in it)` | Generator expression — lazy |
| `sum(f(x) for x in it)` | Genexp in a call (no extra parens) |

```python
def gen(n):           # generator function
    for i in range(n):
        yield i * i

g = gen(3)
next(g)               # 0

def delegate():       # `yield from` is only valid inside a function
    yield from gen(3) # delegate to a sub-generator
```

Generators are single-pass and memory-lazy — the right tool for large or streaming data.

## 12. Functions

| Feature | Description |
|---|---|
| `def f(a, b=1, *args, **kw)` | Positional, default, var-positional, var-keyword |
| Keyword-only | `def f(*, x)` — must be passed by name |
| Positional-only **(3.8)** | `def f(a, /, b)` — cannot be passed by name |
| `return` | Return a value (or `None`) |
| Multiple returns | `return a, b` packs into a tuple |
| `lambda` | Anonymous single-expression function |
| `functools.partial` | Pre-bind arguments |
| Closures | Nested functions capture enclosing scope |
| `global` / `nonlocal` | Assign to outer scopes |
| Docstrings | `"""Summary."""` on the first line |
| Annotations | `def f(x: int) -> str:` — metadata, not enforcement |
| Recursion limit | `sys.setrecursionlimit(n)` (default 1000) |

### Argument unpacking

```python
def f(a, b, c): ...
args = [1, 2, 3]
f(*args)                    # → f(1, 2, 3)
f(**{"a": 1, "b": 2, "c": 3})
```

## 13. Functional Built-ins

| Function | Description |
|---|---|
| `map(f, it)` | Apply `f` lazily |
| `filter(pred, it)` | Keep items where `pred` is true |
| `functools.reduce(f, it, init)` | Fold left into a single value |
| `sorted(it, key=, reverse=)` | Sorted list (stable) |
| `zip(a, b, strict=True)` **(3.10)** | Pair iterables; `strict` checks equal length |
| `enumerate(it, start)` | Index/value pairs |
| `all(it)` / `any(it)` | Boolean reductions |
| `iter(callable, sentinel)` | Two-arg iterator form |
| `next(it, default)` | Advance an iterator |
| `reversed(seq)` | Reverse iterator |
| `slice(a, b, c)` | Reusable slice object |
| `functools.cmp_to_key` | Adapt a comparator to `key=` |
| `operator.itemgetter` | Fast field extraction, `key=` friendly |
| `operator.attrgetter` | Attribute-based extraction |
| `operator.methodcaller` | Call a named method |

```python
from operator import itemgetter
rows.sort(key=itemgetter(1), reverse=True)   # by column 1
```

## 14. Iterators & Itertools

| Tool | Description |
|---|---|
| `itertools.count(a, step)` | Infinite arithmetic sequence |
| `itertools.cycle(it)` | Repeat forever |
| `itertools.repeat(x, n)` | Repeat a value |
| `itertools.chain(*its)` | Concatenate iterables |
| `itertools.islice(it, n)` | Slice a lazy iterable |
| `itertools.takewhile` / `dropwhile` | Conditional prefix/skip |
| `itertools.groupby(it, key)` | Group **adjacent** items (sort first) |
| `itertools.product(*its)` | Cartesian product |
| `itertools.permutations` / `combinations` | Combinatorics |
| `itertools.combinations_with_replacement` | Multiset combinations |
| `itertools.accumulate(it, op)` | Running total |
| `itertools.pairwise(it)` **(3.10)** | Overlapping consecutive pairs |
| `itertools.batched(it, n)` **(3.12)** | Fixed-size chunks |
| `itertools.zip_longest(*its, fillvalue)` | Zip, padding shorter inputs |
| `itertools.tee(it, n)` | Fork an iterator |
| `itertools.filterfalse` | Keep items where predicate is false |
| `itertools.starmap(f, its)` | `map` with unpacked args |

```python
from itertools import pairwise, batched, groupby, chain
list(pairwise("ABC"))            # [('A','B'), ('B','C')]
list(batched(range(5), 2))       # [(0,1), (2,3), (4,)]
```

## 15. functools

| Tool | Description |
|---|---|
| `functools.lru_cache` | Memoisation with a bounded cache |
| `functools.cache` **(3.9)** | Unbounded memoisation |
| `functools.cached_property` **(3.8)** | Compute an attribute once |
| `functools.partial` | Bind arguments ahead of time |
| `functools.partialmethod` | `partial` for methods |
| `functools.reduce` | Fold |
| `functools.wraps` | Preserve metadata in decorators |
| `functools.total_ordering` | Derive comparison methods |
| `functools.singledispatch` | Type-based dispatch |
| `functools.cmp_to_key` | Comparator → key function |

```python
from functools import cache

@cache
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
```

## 16. Object-Oriented Python

| Concept | Description |
|---|---|
| `class C: ...` | Class definition |
| `__init__` | Initialiser (constructor) |
| `self` | The instance, passed explicitly |
| Instance attributes | Set on `self`; per-object |
| Class attributes | Shared by all instances |
| `super()` | Call the next method in the MRO |
| Inheritance | `class D(B):` — multiple bases allowed |
| MRO | `C.__mro__` — resolution order for lookups |
| `@property` | Getter that looks like an attribute |
| `@x.setter` / `@x.deleter` | Define write/delete behaviour |
| `@staticmethod` | Function in the class namespace |
| `@classmethod` | Receives the class, not the instance |
| `@dataclass` **(3.7)** | Auto-generate `__init__`/`__repr__`/`__eq__` |
| `field(default_factory=...)` | Safe mutable defaults for dataclasses |
| `@dataclass(frozen=True)` | Immutable, hashable dataclass |
| `@dataclass(slots=True)` **(3.10)** | Memory-efficient dataclass |
| `@dataclass(kw_only=True)` **(3.10)** | Keyword-only fields |
| `ABC` / `@abstractmethod` | Abstract base classes |
| `Protocol` **(3.8)** | Structural (duck-typed) interfaces |
| `__slots__` | Restrict instance attributes; saves memory |
| Descriptors | `__get__`/`__set__` for reusable attribute logic |
| `@override` **(3.12)** | Assert a method overrides a base member |

```python
from dataclasses import dataclass, field

@dataclass(frozen=True, slots=True)
class Point:
    x: float
    y: float
    tags: tuple[str, ...] = field(default_factory=tuple)
```

### Dunder (magic) methods

| Method | Triggered by |
|---|---|
| `__init__` | Construction |
| `__repr__` | `repr()`, REPL display — be unambiguous |
| `__str__` | `str()`, `print()` — be readable |
| `__eq__` / `__ne__` | `==` / `!=` |
| `__lt__ __le__ __gt__ __ge__` | Ordering comparisons |
| `__hash__` | `hash()`, dict/set keys |
| `__len__` | `len()` |
| `__getitem__` / `__setitem__` / `__delitem__` | `obj[k]` |
| `__contains__` | `x in obj` |
| `__iter__` / `__next__` | Iteration protocol |
| `__enter__` / `__exit__` | `with` blocks |
| `__call__` | `obj()` |
| `__bool__` | Truthiness |
| `__add__ __sub__ __mul__` | Arithmetic operators |
| `__iadd__` etc. | In-place operators (`+=`) |
| `__getattr__` / `__getattribute__` | Attribute access fallbacks |
| `__setattr__` / `__delattr__` | Attribute assignment/deletion |
| `__slots__` | Declared attribute names |
| `__format__` | f-string/`format()` behaviour |
| `__copy__` / `__deepcopy__` | Copy hooks |
| `__reduce__` | Pickle support |
| `__await__` / `__aiter__` / `__anext__` | Async protocols |

## 17. Typing & Type Hints

| Item | Description |
|---|---|
| `x: int = 0` | Variable annotation |
| `def f(x: int) -> str:` | Function annotations |
| `list[int]`, `dict[str, int]` **(3.9)** | Built-in generic aliases |
| `X \| None` **(3.10)** | Union syntax (replaces `Optional[X]`) |
| `Optional[X]` | `X | None` |
| `Union[A, B]` | Either type |
| `Any` | Opt out of checking |
| `Callable[[int], str]` | Function signature |
| `Iterable[T]` / `Iterator[T]` | Iteration protocols |
| `Sequence[T]` / `Mapping[K, V]` | Read-only abstract collections |
| `Literal["a", "b"]` **(3.8)** | Specific allowed values |
| `Final` | Non-reassignable |
| `TypedDict` | Dict with a fixed key schema |
| `Protocol` | Structural interface |
| `Generic[T]` | Parameterised class |
| `Self` **(3.11)** | Returns the enclosing class type |
| `type X = int` **(3.12)** | `type` alias statement |
| `TypeVar` / `ParamSpec` | Type variables for generics |
| `@overload` | Alternate signatures for one implementation |
| `@runtime_checkable` | Allow `isinstance` on a Protocol |
| `cast(T, x)` | Assert a type to the checker |
| `TYPE_CHECKING` | Import guard for type-only imports |
| `from __future__ import annotations` | Lazy (string) annotations |

**Tools:** `mypy`, `pyright`, `pyre`, `ty` — all read these hints statically. Hints are not enforced at runtime.

## 18. Exceptions & Error Handling

| Statement | Description |
|---|---|
| `try:` / `except E:` | Catch a specific exception |
| `except (A, B):` | Catch several types |
| `except E as e:` | Bind the exception instance |
| `else:` | Runs if no exception was raised |
| `finally:` | Always runs (cleanup) |
| `raise E("msg") from e` | Raise with explicit chaining |
| `raise` (bare) | Re-raise the current exception |
| `try` / `except*` **(3.11)** | Handle `ExceptionGroup`s |
| `ExceptionGroup(msg, [e1, e2])` **(3.11)** | Bundle concurrent failures |
| `assert` | Debug-only sanity check |

### Built-in exception hierarchy (excerpt)

| Exception | Raised when |
|---|---|
| `BaseException` | Root of all exceptions |
| `Exception` | Root of normal, catchable errors |
| `KeyboardInterrupt` | `Ctrl-C` (not an `Exception`) |
| `SystemExit` | `sys.exit()` / interpreter shutdown |
| `ArithmeticError` | `ZeroDivisionError`, `OverflowError` |
| `ValueError` | Right type, wrong value |
| `TypeError` | Wrong type for an operation |
| `KeyError` | Missing dict key |
| `IndexError` | Sequence index out of range |
| `AttributeError` | Missing attribute |
| `NameError` | Unknown local/global name |
| `ImportError` / `ModuleNotFoundError` | Import failure |
| `FileNotFoundError` / `PermissionError` | OS errors (`OSError` subclasses) |
| `StopIteration` / `StopAsyncIteration` | Iterator exhaustion |
| `RuntimeError` / `RecursionError` | Runtime problems |
| `TimeoutError` | Operation timed out |
| `NotImplementedError` | Abstract method placeholder |
| `Warning` family | Non-fatal advisories |

**Rule of thumb:** catch the narrowest exception you can handle; never `except Exception: pass`, which hides real bugs.

## 19. Context Managers

| Item | Description |
|---|---|
| `with open(f) as fh:` | Automatic cleanup on exit |
| `with A() as a, B() as b:` | Multiple managers |
| `contextlib.contextmanager` | Build one from a generator |
| `contextlib.suppress(E)` | Ignore a specific exception |
| `contextlib.ExitStack` | Dynamic set of managers |
| `contextlib.closing(x)` | Ensure `x.close()` |
| `contextlib.nullcontext` | No-op placeholder |
| `contextlib.asynccontextmanager` | Async generator form |
| `__enter__` / `__exit__` | The protocol methods |

```python
import os, time
from contextlib import contextmanager, suppress

@contextmanager
def timing(label):
    start = time.perf_counter()
    try:
        yield
    finally:
        print(f"{label}: {time.perf_counter() - start:.3f}s")

with suppress(FileNotFoundError):
    os.remove("missing.txt")
```

## 20. Decorators

| Item | Description |
|---|---|
| `@decorator` | Sugar for `f = decorator(f)` |
| `functools.wraps` | Preserve `__name__`/`__doc__` in wrappers |
| Stacked decorators | Applied bottom-up |
| Decorators with arguments | Return a configured decorator (3 layers) |
| Class-based decorator | Implement `__call__` |
| Standard decorators | `@property`, `@staticmethod`, `@classmethod`, `@cache`, `@dataclass`, `@abstractmethod` |

```python
import functools, time

def timed(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kw):
        t0 = time.perf_counter()
        try:
            return fn(*args, **kw)
        finally:
            print(f"{fn.__name__} took {time.perf_counter()-t0:.4f}s")
    return wrapper
```

## 21. Files, I/O & Pathlib

### `open()`

| Mode | Meaning |
|---|---|
| `"r"` | Read text (default) |
| `"w"` | Write, truncating |
| `"a"` | Append |
| `"x"` | Exclusive create (fails if present) |
| `"b"` | Binary suffix, e.g. `"rb"`, `"wb"` |
| `"r+"` | Read and write |
| `encoding=` | Text encoding, e.g. `"utf-8"` |
| `newline=` | Line-ending translation control |
| `errors=` | `"strict"`, `"ignore"`, `"replace"` |

| File method | Description |
|---|---|
| `f.read()` / `f.read(n)` | Whole file / `n` chars |
| `f.readline()` / `f.readlines()` | One line / all lines |
| `f.write(s)` / `f.writelines(it)` | Write |
| `f.seek(pos)` / `f.tell()` | Position control |
| `f.flush()` | Push buffers to the OS |
| `f.close()` | Close (implicit in `with`) |
| `for line in f:` | Memory-efficient line iteration |
| `f.name` / `f.mode` / `f.closed` | File metadata |

### `pathlib`

| Expression | Description |
|---|---|
| `Path("dir") / "file.txt"` | Join with `/` |
| `p.exists()` | Path exists |
| `p.is_file()` / `p.is_dir()` | Type predicates |
| `p.read_text()` / `p.write_text(s)` | Quick text I/O |
| `p.read_bytes()` / `p.write_bytes(b)` | Quick binary I/O |
| `p.name` / `p.stem` / `p.suffix` | Filename components |
| `p.parent` / `p.parents` | Ancestors |
| `p.parts` | Path components tuple |
| `p.resolve()` | Absolute, symlinks resolved |
| `p.absolute()` | Absolute without resolving |
| `p.mkdir(parents=True, exist_ok=True)` | Create directories |
| `p.iterdir()` / `p.glob("*.py")` / `p.rglob("*.py")` | Iterate / match |
| `p.unlink()` / `p.rmdir()` | Delete file / empty dir |
| `p.rename(t)` / `p.replace(t)` | Move |
| `p.stat()` | Metadata |
| `p.touch()` | Create empty / update mtime |
| `p.chmod(mode)` | Change permissions |
| `p.samefile(q)` | Same inode? |
| `p.is_relative_to(q)` **(3.9)** | Ancestry test |
| `Path.home()` / `Path.cwd()` | Common anchors |

### Other file tools

| Module | Purpose |
|---|---|
| `os` | `getcwd`, `chdir`, `listdir`, `remove`, `rename`, `walk`, `environ`, `path.*` |
| `shutil` | `copy`, `copy2`, `copytree`, `move`, `rmtree`, `which`, `disk_usage` |
| `glob` | Shell-style path matching |
| `tempfile` | `NamedTemporaryFile`, `TemporaryDirectory` |
| `io` | `StringIO`, `BytesIO` in-memory streams |
| `csv` / `json` | Structured text formats |
| `zipfile` / `tarfile` / `gzip` / `bz2` / `lzma` | Archives and compression |
| `pickle` | Serialise Python objects (trusted data only) |
| `shelve` / `dbm` | Simple on-disk mappings |
| `sqlite3` | Embedded relational database |
| `mmap` | Memory-mapped files |
| `fileinput` | Iterate lines across many files |

## 22. Standard Library Map

| Module | What it gives you |
|---|---|
| `sys` | Interpreter state: `argv`, `path`, `stdin/out/err`, `exit`, `version` |
| `os` | OS interface: env vars, processes, paths, file ops |
| `os.path` | Legacy path manipulation (`join`, `basename`, `splitext`) |
| `pathlib` | Object-oriented paths (preferred) |
| `io` | In-memory and abstract streams |
| `re` | Regular expressions |
| `json` | JSON encode/decode |
| `csv` | CSV reader/writer/dict access |
| `configparser` | INI-style config files |
| `tomllib` **(3.11)** | Read-only TOML parsing |
| `dataclasses` | Boilerplate-free data classes |
| `enum` | Symbolic constants (`Enum`, `IntEnum`, `Flag`, `auto`) |
| `typing` | Type-hint machinery |
| `abc` | Abstract base classes |
| `collections` | `Counter`, `defaultdict`, `deque`, `namedtuple` |
| `itertools` | Iterator algebra building blocks |
| `functools` | Higher-order helpers |
| `operator` | Function forms of operators |
| `heapq` | Heap queue / priority queue |
| `bisect` | Binary search on sorted lists |
| `array` | Compact homogeneous numeric arrays |
| `struct` | Pack/unpack C-style binary layouts |
| `copy` | `copy` and `deepcopy` |
| `datetime` / `time` / `calendar` | Dates, times, durations, calendars |
| `zoneinfo` **(3.9)** | IANA timezone database |
| `random` | Pseudo-random numbers, sampling, shuffling |
| `secrets` | Cryptographically secure randomness/tokens |
| `statistics` | Mean, median, variance, correlation |
| `hashlib` | MD5/SHA/BLAKE2 digests |
| `hmac` | Keyed message authentication |
| `base64` / `binascii` | Binary↔text encodings |
| `uuid` | UUID generation |
| `subprocess` | Run external commands |
| `shlex` | Shell-style tokenising/quoting |
| `signal` | Unix signal handling |
| `threading` | Threads, locks, events |
| `multiprocessing` | Process-based parallelism |
| `concurrent.futures` | Thread/process pool executors |
| `asyncio` | Async I/O event loop, tasks, streams |
| `queue` | Thread-safe FIFO/LIFO/priority queues |
| `socket` | Low-level networking |
| `ssl` | TLS wrappers and context configuration |
| `http.client` / `http.server` | HTTP client and simple servers |
| `urllib.request/parse/error` | Higher-level URL fetching |
| `ipaddress` | IP address/network objects |
| `smtplib` / `imaplib` / `poplib` | Email sending and retrieval |
| `email` | Parse and build MIME messages |
| `xml.etree.ElementTree` | XML parsing and generation |
| `html` / `html.parser` | HTML escaping and parsing |
| `textwrap` | Wrap, fill, dedent text |
| `string` | Constants and `Template` substitution |
| `pprint` | Pretty-printing nested structures |
| `reprlib` | Truncating `repr` for large objects |
| `logging` | Structured, levelled application logging |
| `warnings` | Non-fatal advisory messages |
| `traceback` | Programmatic traceback formatting |
| `argparse` | Command-line argument parsing |
| `getpass` | Hidden password input |
| `platform` | OS/hardware/version identification |
| `sysconfig` | Interpreter build configuration paths |
| `importlib` | Dynamic imports and reloading |
| `importlib.metadata` | Read installed distribution metadata |
| `pkgutil` | Package discovery helpers |
| `inspect` | Introspect live objects, signatures, source |
| `ast` | Parse Python into syntax trees |
| `tokenize` | Python tokenizer |
| `dis` | Disassemble to bytecode |
| `gc` | Garbage-collector interface |
| `weakref` | Non-owning references |
| `ctypes` | Call C libraries (also third-party `cffi`) |
| `select` / `selectors` | I/O multiplexing |
| `fcntl` / `termios` / `tty` | Unix file/terminal control |
| `pty` | Pseudo-terminal handling |
| `atexit` | Run code at interpreter exit |
| `contextlib` | Context-manager utilities |
| `graphlib` **(3.9)** | Topological sorting |
| `difflib` | Sequence diffing and fuzzy matching |
| `unittest` / `doctest` | Built-in testing frameworks |
| `timeit` | Micro-benchmarking |
| `cProfile` / `profile` / `pstats` | Deterministic profiling |
| `tracemalloc` | Python-level memory allocation tracking |
| `resource` | Process resource usage limits |
| `locale` / `gettext` | Internationalisation |
| `filecmp` / `fnmatch` | File comparison / name matching |
| `dataclasses` | (also listed above) record types |
| `concurrent` | Parallel-execution namespace |
| `sqlite3` | Embedded SQL database |
| `zipapp` | Build executable zip applications |

### Regex quick reference (`re`)

| Pattern | Matches |
|---|---|
| `.` | Any character except newline |
| `\d` / `\w` / `\s` | Digit / word char / whitespace |
| `\D` / `\W` / `\S` | Negations |
| `^` / `$` | Start / end of string (or line with `MULTILINE`) |
| `\b` | Word boundary |
| `*` / `+` / `?` | Zero+ / one+ / zero-or-one |
| `{m,n}` | Between m and n repetitions |
| `*?` `+?` | Lazy (non-greedy) variants |
| `[...]` / `[^...]` | Character class / negation |
| `(a\|b)` | Alternation |
| `(...)` | Capturing group |
| `(?:...)` | Non-capturing group |
| `(?P<name>...)` | Named group |
| `(?=...)` / `(?!...)` | Lookahead / negative lookahead |
| `(?<=...)` / `(?<!...)` | Lookbehind / negative lookbehind |

| `re` function | Description |
|---|---|
| `re.search(p, s)` | First match anywhere |
| `re.match(p, s)` | Match at the start only |
| `re.fullmatch(p, s)` | Must consume the whole string |
| `re.findall(p, s)` | All matches as a list |
| `re.finditer(p, s)` | Iterator of match objects |
| `re.sub(p, rep, s, count)` | Substitute |
| `re.split(p, s)` | Split by pattern |
| `re.compile(p, flags)` | Pre-compile for reuse |
| `m.group(n)` / `m.groups()` / `m.groupdict()` | Captures |
| `m.start()` / `m.end()` / `m.span()` | Positions |
| Flags | `IGNORECASE`, `MULTILINE`, `DOTALL`, `VERBOSE`, `ASCII` |
| Replacement | `\1` backrefs, `\g<name>`, callable replacement |

### `datetime` essentials

| Item | Description |
|---|---|
| `datetime.now()` / `datetime.now(timezone.utc)` | Current local / aware UTC time |
| `datetime.now(timezone.utc)` | Aware UTC time (preferred) |
| `date.today()` | Today's date |
| `datetime.fromisoformat(s)` **(3.11** parses most ISO-8601**)** | Parse ISO strings |
| `dt.isoformat()` | Serialise to ISO-8601 |
| `dt.strftime(fmt)` / `strptime(s, fmt)` | Format / parse |
| `dt.timestamp()` / `fromtimestamp()` | POSIX epoch boundary |
| `timedelta(days=1, hours=2)` | Duration arithmetic |
| `dt + timedelta(...)` | Date arithmetic |
| `dt.replace(...)` | Copy with changes |
| `dt.astimezone(tz)` | Convert timezone |
| `ZoneInfo("Europe/London")` **(3.9)** | IANA timezone |
| `dt.date()` / `dt.time()` | Extract components |
| `dt.weekday()` / `isoweekday()` | Day of week |

**Rule:** always work in **timezone-aware UTC** internally; convert to local only for display.

## 23. Concurrency & Parallelism

| Approach | Best for | Notes |
|---|---|---|
| `threading` | I/O-bound work | Subject to the GIL for CPU work |
| `multiprocessing` | CPU-bound work | Separate interpreters; pickling required |
| `asyncio` | High-concurrency I/O | Single-threaded cooperative multitasking |
| `concurrent.futures` | Pool abstractions | `ThreadPoolExecutor`, `ProcessPoolExecutor` |

| Tool | Description |
|---|---|
| `Thread(target=, args=)` | Create a thread |
| `t.start()` / `t.join()` | Run / wait |
| `Lock` / `RLock` / `Semaphore` / `Event` | Synchronisation primitives |
| `threading.local` | Thread-local storage |
| `queue.Queue` | Thread-safe work queue |
| `Process(target=, args=)` | Spawn a process |
| `Pool.map(func, it)` | Parallel map over a process pool |
| `Pool.imap` / `imap_unordered` | Lazy parallel map |
| `Pool.starmap` | Parallel map with unpacked args |
| `Manager()` | Shared state between processes |
| `ThreadPoolExecutor(max_workers)` | High-level thread pool |
| `ProcessPoolExecutor` | High-level process pool |
| `executor.submit(fn, *a)` | Schedule one call, returns a `Future` |
| `future.result(timeout)` | Retrieve or raise the result |
| `asyncio.run(main())` | Run an async entry point |
| `async def` / `await` | Coroutine definition / suspension |
| `asyncio.gather(*aws)` | Run concurrently, collect results |
| `asyncio.TaskGroup()` **(3.11)** | Structured concurrency with cleanup |
| `asyncio.create_task(coro)` | Fire off a background task |
| `asyncio.sleep(n)` | Non-blocking sleep |
| `asyncio.Queue` | Async producer/consumer queue |
| `asyncio.StreamReader/Writer` | Async TCP streams |
| `asyncio.timeout(n)` **(3.11)** | Cancel a block after a deadline |
| `asyncio.to_thread(fn)` | Run blocking code off the loop |
| `loop.run_in_executor` | Bridge to threads/processes |

**The GIL:** CPython serialises pure-Python bytecode execution per process, so threads help mainly with blocking I/O (many C extensions — `hashlib`, `zlib`, NumPy — release the GIL during their work). The **free-threaded build** (`python3.13t`, PEP 703) removes it, but is experimental.

```python
import asyncio

async def fetch(n):
    await asyncio.sleep(n)
    return n

async def main():
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(i)) for i in (1, 2, 3)]
    print([t.result() for t in tasks])

asyncio.run(main())
```

## 24. Testing

| Tool | Description |
|---|---|
| `unittest.TestCase` | Built-in xUnit-style framework |
| `assertEqual` / `assertRaises` / `assertIn` | Built-in assertions |
| `setUp` / `tearDown` | Per-test fixtures |
| `setUpClass` / `tearDownClass` | Per-class fixtures |
| `unittest.mock` | `Mock`, `MagicMock`, `patch`, `autospec` |
| `pytest` | Convention-based runner; plain `assert` |
| `pytest -k pat` | Select tests by name |
| `pytest -x` / `--maxfail=1` | Stop at first failure |
| `pytest -q` / `-v` | Quieter / more verbose |
| `pytest --lf` | Re-run last-failed tests |
| `@pytest.fixture` | Dependency-injected setup |
| `@pytest.mark.parametrize` | Table-driven cases |
| `pytest.raises(E)` | Assert an exception |
| `pytest.approx(x)` | Float comparison with tolerance |
| `capsys` / `monkeypatch` | Capture output / patch environment |
| `tmp_path` | Isolated temp directory fixture |
| `conftest.py` | Shared fixtures and hooks |
| `pytest-cov` | Coverage measurement |
| `hypothesis` | Property-based / generative testing |
| `pytest-benchmark` | Micro-benchmarking as tests |
| `tox` / `nox` | Test across multiple environments |
| `doctest` | Tests embedded in docstrings |

```python
import pytest

@pytest.mark.parametrize("n,expected", [(0, 1), (1, 1), (5, 120)])
def test_factorial(n, expected):
    assert factorial(n) == expected

def test_raises():
    with pytest.raises(ValueError):
        parse("")
```

## 25. Packaging & Environments

| Concept | Description |
|---|---|
| Virtual environment | Isolated `site-packages` per project |
| `requirements.txt` | Flat dependency list for `pip` |
| `pyproject.toml` | Modern project metadata + build config (PEP 621) |
| Build backends | `setuptools`, `hatchling`, `flit-core`, `poetry-core`, `pdm-backend` |
| `src/` layout | Code under `src/pkg/` — avoids import shadowing |
| Wheel (`.whl`) | Built binary distribution |
| sdist (`.tar.gz`) | Source distribution |
| `__init__.py` | Marks a directory as a regular package |
| Namespace package | Import package without `__init__.py` |
| Entry points | `[project.scripts]` console commands |
| Lock files | `uv.lock`, `poetry.lock`, `pdm.lock`, `requirements.txt` |
| Semantic versioning | `MAJOR.MINOR.PATCH`; specifiers `>=1.2,<2` |
| Extras | Optional dependency groups, `pkg[dev]` |
| Editable install | `pip install -e .` — live source linking |

```
project/
├── pyproject.toml
├── README.md
├── src/
│   └── mypkg/
│       ├── __init__.py
│       └── core.py
└── tests/
    └── test_core.py
```

## 26. Python CLI Commands

| Command | Description |
|---|---|
| `python3 script.py` | Run a script |
| `python3 -c "code"` | Run inline code |
| `python3 -m module` | Run a module as `__main__` |
| `python3 -i script.py` | Run then drop into the REPL |
| `python3 -` | Read the program from stdin |
| `python3 -O` | Optimise (strips asserts; `-OO` also strips docstrings) |
| `python3 -B` | Don't write `.pyc` files |
| `python3 -u` | Unbuffered stdout/stderr |
| `python3 -X dev` | Development mode (extra warnings) |
| `python3 -W error` | Turn warnings into errors |
| `python3 -V` / `--version` | Interpreter version |
| `python3 -m venv .venv` | Create a virtual environment |
| `python3 -m pydoc name` | Render documentation |
| `python3 -m pydoc -b` | Serve docs in a browser |
| `python3 -m json.tool f.json` | Pretty-print JSON |
| `python3 -m http.server 8000` | Serve the current directory over HTTP |
| `python3 -m zipapp pkg -o app.pyz` | Build a zipapp |
| `python3 -m timeit "expr"` | Quick micro-benchmark |
| `python3 -m cProfile -s cumtime s.py` | Profile a script |
| `python3 -m trace --trace s.py` | Line-by-line trace |
| `python3 -m compileall dir` | Byte-compile a tree |
| `python3 -m unittest discover` | Run all tests |
| `python3 -m pdb script.py` | Debugger entry |
| `python3 -m asyncio` | Async REPL |
| `python3 -m tkinter` | Verify tkinter is available |

> On 3.13+, bare `python` opens the new interactive REPL with multi-line editing and colour.

### Package & environment tooling

| Command | Description |
|---|---|
| `pip install pkg` | Install a package |
| `pip install pkg==1.2.3` | Pin a version |
| `pip install -r requirements.txt` | Install from a list |
| `pip install -e .` | Editable install of the current project |
| `pip uninstall pkg` | Remove a package |
| `pip list --outdated` | Show upgradable packages |
| `pip freeze > req.txt` | Export the exact environment |
| `pip show pkg` | Package metadata |
| `pip download pkg` | Fetch without installing |
| `pip cache purge` | Clear the wheel cache |
| `pip-audit` | Scan dependencies for CVEs |
| `pipx install app` | Install an isolated CLI app |
| `pipx list` / `pipx upgrade-all` | Manage pipx applications |
| `uv venv` / `uv pip install` | Fast Rust-based resolver |
| `uv sync` | Install from `uv.lock` |
| `uv run script.py` | Run inside the project env |
| `uv add` / `uv remove` | Manage dependencies |
| `poetry init` / `poetry add` | Poetry project management |
| `poetry install --with dev` | Install with a dependency group |
| `poetry build` / `poetry publish` | Build and upload |
| `pdm install` / `hatch build` | Alternative managers |
| `conda create -n env pkg` | Conda environment |
| `mamba` | Faster conda front-end |
| `source .venv/bin/activate` | Activate a venv (Unix) |
| `.venv\Scripts\activate` | Activate a venv (Windows) |
| `deactivate` | Leave the venv |
| `python -m build` | Build wheel + sdist |
| `twine upload dist/*` | Publish to a package index |
| `pipenv install` | Pipenv-based workflow |

### Quality & static analysis

| Command | Description |
|---|---|
| `ruff check .` | Extremely fast linter (flake8 replacement) |
| `ruff format` | Black-compatible formatter |
| `black .` | Opinionated code formatter |
| `isort .` | Sort imports |
| `flake8` | Classic style/error linter |
| `pylint` | Comprehensive linter |
| `mypy .` | Static type checker |
| `pyright` / `basedpyright` | Alternative type checker |
| `bandit -r .` | Security-focused linter |
| `vulture` | Find dead code |
| `radon cc .` | Cyclomatic-complexity report |
| `interrogate` | Docstring coverage |
| `pre-commit run --all-files` | Run configured git hooks |
| `pyupgrade` | Modernise syntax for a target version |
| `autoflake` | Remove unused imports/variables |
| `py-spy top --pid N` | Sampling profiler for live processes |
| `scalene` / `memray` | CPU/memory profilers |
| `pytest --cov=pkg` | Coverage report |
| `coverage html` | Browseable coverage |
| `pdb` / `breakpoint()` | Interactive debugger |
| `ipdb` / `pudb` | Enhanced debuggers |
| `rich` / `textual` | Terminal formatting / TUIs |
| `tqdm` | Progress bars |
| `logging` / `structlog` | Application logging |
| `hypothesis` | Property-based testing |

### Common third-party libraries

| Area | Libraries |
|---|---|
| Numeric | `numpy`, `scipy`, `pandas`, `polars` |
| Visualisation | `matplotlib`, `seaborn`, `plotly`, `altair` |
| Web frameworks | `django`, `flask`, `fastapi`, `starlette`, `aiohttp` |
| HTTP clients | `requests`, `httpx`, `urllib3`, `aiohttp` |
| Data validation | `pydantic`, `attrs`, `marshmallow`, `cerberus` |
| ORM | `sqlalchemy`, `django` (built-in ORM), `peewee`, `tortoise-orm` |
| CLI | `click`, `typer`, `rich`, `fire` |
| Testing | `pytest`, `hypothesis`, `faker`, `tox`, `responses` |
| Async | `asyncio`, `anyio`, `trio`, `uvloop` |
| ML | `scikit-learn`, `pytorch`, `tensorflow`, `transformers` |
| Scraping | `beautifulsoup4`, `lxml`, `scrapy`, `playwright` |
| Serialisation | `orjson`, `msgspec`, `pyyaml`, `tomli-w` |
| Cloud | `boto3`, `azure-*`, `google-cloud-*` |
| Dates | `pendulum`, `arrow`, `dateutil` |
| Templating | `jinja2`, `mako` |
| Config | `python-dotenv`, `dynaconf`, `hydra` |

## 27. Debugging & Introspection

| Tool | Description |
|---|---|
| `breakpoint()` **(3.7)** | Drop into a debugger (honours `PYTHONBREAKPOINT`) |
| `pdb` commands | `n` next, `s` step, `c` continue, `l` list, `p expr`, `b` breakpoint, `q` quit |
| `python -m pdb script.py` | Debug from the first line |
| `pdb.post_mortem(tb)` | Debug a caught traceback |
| `sys.excepthook` | Global uncaught-exception handler |
| `traceback.print_exc()` | Print the current traceback |
| `traceback.format_exc()` | Capture it as a string |
| `traceback.print_stack()` | Show the call stack |
| `inspect.signature(fn)` | Parameter list of a callable |
| `inspect.getsource(fn)` | Source text |
| `inspect.getmembers(obj)` | All attributes |
| `vars(obj)` / `obj.__dict__` | Instance state |
| `dir(obj)` | Attribute names |
| `help(obj)` | Interactive documentation |
| `pprint.pprint(obj)` | Readable nested output |
| `dis.dis(fn)` | Bytecode disassembly |
| `sys.settrace(fn)` | Set a tracing function |
| `tracemalloc.start()` | Begin allocation tracking |
| `gc.get_objects()` | List of live tracked objects |
| `sys.getsizeof(obj)` | Shallow memory size |
| `weakref.ref(obj)` | Non-owning reference |
| `assert` + `pytest` | Fast feedback loop |
| `logging.debug` / `%r` | Prefer logging over `print` debugging |

## 28. Idioms & Common Recipes

```python
# Guard clause / default
value = d.get("k", default)

# Count occurrences
from collections import Counter
Counter(words).most_common(5)

# Group into a dict of lists
from collections import defaultdict
by_key = defaultdict(list)
for k, v in pairs: by_key[k].append(v)

# Flatten one level
flat = [x for row in matrix for x in row]

# Unique while preserving order (3.7+: dicts are ordered)
unique = list(dict.fromkeys(seq))

# Read a file line by line, safely
with open(path, encoding="utf-8") as fh:
    for line in fh:
        process(line.rstrip("\n"))

# Atomic write
import os, tempfile
fd, tmp = tempfile.mkstemp(dir=os.path.dirname(path))
with os.fdopen(fd, "w") as fh:
    fh.write(data)
os.replace(tmp, path)          # atomic on POSIX

# Chunk an iterable
from itertools import islice
def chunk(it, n):
    it = iter(it)
    while (b := list(islice(it, n))):   # walrus, 3.8
        yield b

# Top-N by value
import heapq
heapq.nlargest(3, d.items(), key=lambda kv: kv[1])

# Merge dicts (right wins)
merged = a | b                     # 3.9+

# Safe filename from arbitrary text
import re
safe = re.sub(r"[^\w.-]+", "_", title).strip("_")

# Time a block
from time import perf_counter
t0 = perf_counter(); work(); elapsed = perf_counter() - t0

# Retry with backoff
import time
for attempt in range(5):
    try:
        result = flaky(); break
    except TransientError:
        time.sleep(2 ** attempt)
else:
    raise RuntimeError("gave up after 5 attempts")
```

## 29. Version Feature Notes

| Version | Highlights |
|---|---|
| 3.6 | f-strings, variable annotations, ordered dicts (impl detail) |
| 3.7 | `dataclasses`, ordered dicts guaranteed, `breakpoint()`, `from __future__ import annotations` |
| 3.8 | Walrus `:=`, positional-only `/`, `Protocol` |
| 3.9 | `list[int]` generics, `dict \|` merge, `removeprefix/suffix`, `zoneinfo`, `math.lcm`, `functools.cache` |
| 3.10 | `match`/`case`, `X \| Y` unions, `zip(strict=)`, `pairwise`, `dataclass(slots=)` |
| 3.11 | `except*` / `ExceptionGroup`, `TaskGroup`, `asyncio.timeout`, `Self`, `tomllib`, much faster startup |
| 3.12 | PEP 695 `type`/generics syntax, multi-line f-strings, `itertools.batched`, `sys.monitoring`, `@override` |
| 3.13 | New interactive REPL, experimental free-threaded build (PEP 703), experimental JIT, `pathlib` improvements |
| 3.14 | (current-era release) t-strings / annotations work continues — verify against your interpreter |

## 30. Common Pitfalls

| Pitfall | Why it bites | Fix |
|---|---|---|
| Mutable default arg | Defaults are created once and shared | Use `None` + `if x is None: x = []` |
| Modifying a list while iterating | Skips elements silently | Iterate a copy or build a new list |
| `is` vs `==` | `is` compares identity, not value | Use `==` for values; `is` only for `None`/sentinels |
| Integer caching | `a is b` is True for small ints, False for large | Never use `is` on numbers |
| Shallow copy surprise | Nested objects are still shared | `copy.deepcopy` |
| Late-binding closures | Loop variables captured by reference | Bind with `functools.partial` or a default arg |
| Bare `except:` | Swallows `KeyboardInterrupt`/`SystemExit` | Catch specific exceptions |
| `except: pass` | Hides real bugs | Log or re-raise |
| String concatenation in a loop | O(n²) memory churn | `"".join(parts)` |
| Mutable class attribute | Shared across all instances | Assign in `__init__` |
| Comparing floats with `==` | Binary rounding | `math.isclose` / `pytest.approx` |
| `datetime.now()` naive | Ambiguous across timezones | Use `now(timezone.utc)` |
| `pickle` on untrusted data | Arbitrary code execution | Use JSON, or `pickle` only on trusted input |
| Reading a whole huge file | Memory blow-up | Iterate line by line |
| Shadowing builtins | `list = [1]`, `id = 5` break code | Rename the variable |
| Circular imports | Import-time deadlock | Restructure or import inside functions |
| Slicing copies | Large slices duplicate memory | Use `memoryview`/indices for big buffers |
| `float` for money | Precision drift | `decimal.Decimal` |
| Reusing a consumed generator | Second loop yields nothing | Materialise into a list |
| GIL assumption | Threads don't speed up CPU work | Use `multiprocessing` |

## Appendix A: Format Spec Mini-Language

```
[[fill]align][sign][#][0][width][,][.precision][type]

align:  <  left     >  right     ^  centre     =  pad after sign
sign:   +  always   -  negative only   (space)  space for positive
type:   s str  d int  f float  e sci  % pct  x hex  o oct  b bin  c char
```

Examples: `f"{x:>10}"`, `f"{x:<10.2f}"`, `f"{x:^8}"`, `f"{x:,.0f}"`, `f"{x:#010x}"`.

## Appendix B: Exit Codes & `sys.exit`

| Call | Effect |
|---|---|
| `sys.exit()` / `sys.exit(0)` | Success |
| `sys.exit(1)` | General error |
| `sys.exit("message")` | Prints to stderr, exits 1 |
| Uncaught exception | Traceback to stderr, exit 1 |
| `raise SystemExit(n)` | Same as `sys.exit(n)` |

Convention: `0` = success, non-zero = failure. Shells surface this as `$?`.

## Appendix C: `__main__` Package Layout

```python
# src/mypkg/__main__.py
from .cli import main

if __name__ == "__main__":
    raise SystemExit(main())
```

Then `python -m mypkg` works. For a console script, declare in `pyproject.toml`:

```toml
[project.scripts]
mytool = "mypkg.cli:main"
```

## Appendix D: The Zen of Python

`python3 -c "import this"` — 19 aphorisms. The load-bearing ones:

- Beautiful is better than ugly; explicit is better than implicit.
- Simple is better than complex; complex is better than complicated.
- Readability counts.
- Errors should never pass silently.
- In the face of ambiguity, refuse the temptation to guess.
- There should be one — and preferably only one — obvious way to do it.
- If the implementation is hard to explain, it's a bad idea.

**Style authority:** PEP 8 (style), PEP 257 (docstrings), PEP 484/585/604 (typing), PEP 517/518/621 (packaging).

## Appendix E: Quick Sanity Commands

```bash
python3 -V                              # interpreter version
python3 -m venv .venv && . .venv/bin/activate
python3 -m pip install -U pip
python3 -c "import sys; print(sys.path)"  # module search path
python3 -c "import this"                 # the Zen
python3 -m pydoc -b                      # browse the docs
python3 -m json.tool data.json           # pretty-print JSON
python3 -m cProfile -s tottime s.py      # profile
python3 -X importtime -c "import pkg"    # slow-import diagnosis
```
