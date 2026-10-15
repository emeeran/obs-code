---
title: JavaScript Reference — Concepts, APIs & Tooling
tags: [javascript, js, nodejs, browser, reference, cheatsheet]
created: 2026-10-15
scope: Language core · Async & event loop · Browser DOM/Web APIs · Node.js · Tooling · TypeScript primer
---

# JavaScript Reference

A comprehensive overview of JavaScript: language concepts, built-in APIs, browser and Node environments, and tooling.

**Version notation:** ECMAScript features are marked with the edition that introduced them, e.g. **(ES2020)**. Runtime features are marked **(Node 20+)** or **(browser)**. ECMAScript editions map roughly one per year from 2015 (ES6/ES2015) onward.

> **Verify before relying on it:** Node LTS lines advance every October. Run `node -v` rather than trusting a version claim here, and check the [node.green](https://node.green) ES compatibility tables for a specific runtime.

---

## Table of Contents

- [[#1. Language Fundamentals|1. Language Fundamentals]]
- [[#2. Variables & Scope|2. Variables & Scope]]
- [[#3. Data Types & Coercion|3. Data Types & Coercion]]
    - [[#Primitives (7) plus Object|Primitives (7) plus Object]]
    - [[#The falsy set (everything else is truthy)|The falsy set (everything else is truthy)]]
    - [[#Coercion rules|Coercion rules]]
- [[#4. Operators|4. Operators]]
    - [[#Short-circuit and nullish behaviour|Short-circuit and nullish behaviour]]
    - [[#Equality and bitwise gotchas|Equality and bitwise gotchas]]
- [[#5. Strings|5. Strings]]
    - [[#Template literals **(ES2015)**|Template literals **(ES2015)**]]
    - [[#Conversion|Conversion]]
- [[#6. Numbers & Math|6. Numbers & Math]]
- [[#7. Arrays|7. Arrays]]
    - [[#Mutating methods|Mutating methods]]
    - [[#Non-mutating methods|Non-mutating methods]]
    - [[#Iteration & aggregation|Iteration & aggregation]]
- [[#8. Objects|8. Objects]]
    - [[#Property descriptors|Property descriptors]]
    - [[#Getters, setters, computed keys, shorthand|Getters, setters, computed keys, shorthand]]
- [[#9. Destructuring, Spread & Rest|9. Destructuring, Spread & Rest]]
- [[#10. Functions|10. Functions]]
- [[#11. Control Flow|11. Control Flow]]
- [[#12. Iterators & Generators|12. Iterators & Generators]]
- [[#13. Map, Set, WeakMap, WeakSet|13. Map, Set, WeakMap, WeakSet]]
- [[#14. Classes, Prototypes & Inheritance|14. Classes, Prototypes & Inheritance]]
- [[#15. `this` and Explicit Binding|15. `this` and Explicit Binding]]
- [[#16. Modules|16. Modules]]
    - [[#ES Modules (ESM, the standard)|ES Modules (ESM, the standard)]]
- [[#17. Async JavaScript|17. Async JavaScript]]
    - [[#The three generations|The three generations]]
    - [[#Promises|Promises]]
    - [[#Async/await patterns|Async/await patterns]]
    - [[#Cancellation|Cancellation]]
- [[#18. Error Handling|18. Error Handling]]
- [[#19. JSON|19. JSON]]
- [[#20. Regular Expressions|20. Regular Expressions]]
- [[#21. Dates & Times|21. Dates & Times]]
- [[#22. DOM & Browser APIs|22. DOM & Browser APIs]]
    - [[#Events|Events]]
    - [[#Web platform APIs|Web platform APIs]]
- [[#23. Node.js|23. Node.js]]
    - [[#Common Node snippets|Common Node snippets]]
- [[#24. The Event Loop & Concurrency|24. The Event Loop & Concurrency]]
- [[#25. Modern Syntax Cheat-Sheet|25. Modern Syntax Cheat-Sheet]]
- [[#26. Tooling & Ecosystem|26. Tooling & Ecosystem]]
    - [[#npm commands|npm commands]]
    - [[#`package.json` fields that matter|`package.json` fields that matter]]
    - [[#Semver ranges|Semver ranges]]
- [[#27. CLI Commands|27. CLI Commands]]
- [[#28. Common Pitfalls|28. Common Pitfalls]]
- [[#Appendix A: Array & Object Method Quick Reference|Appendix A: Array & Object Method Quick Reference]]
- [[#Appendix B: Promise Combinators at a Glance|Appendix B: Promise Combinators at a Glance]]
- [[#Appendix C: Environment Differences|Appendix C: Environment Differences]]
- [[#Appendix D: TypeScript Primer|Appendix D: TypeScript Primer]]
- [[#Appendix E: Quick Reference Card|Appendix E: Quick Reference Card]]

---


## 1. Language Fundamentals

| Concept | Description |
|---|---|
| Dynamically typed | Types belong to values; variables are untyped bindings |
| Weakly typed | Implicit coercion happens freely (`"1" + 1 === "11"`) |
| Multi-paradigm | Supports imperative, OO (prototypal), and functional styles |
| Single-threaded | One call stack; concurrency is achieved via the event loop |
| JIT-compiled | Engines optimise hot code at runtime (V8, JavaScriptCore, SpiderMonkey) |
| Prototype-based | Inheritance is delegation to a prototype object, not classes at runtime |
| Garbage-collected | A generational tracing GC reclaims unreachable objects |
| First-class functions | Functions are values: assignable, passable, returnable |
| Strict mode | `"use strict"` opts into stricter semantics; **modules are always strict** |
| Semicolon insertion | ASI inserts semicolons at newlines — a frequent source of surprises |
| Nullish vs falsy | `undefined`/`null` are nullish; `0`/`""`/`false` are falsy but not nullish |
| `NaN` quirks | `NaN !== NaN`; use `Number.isNaN` to test it |
| Hoisting | Declarations move to the top of their scope, with different rules per keyword |
| TDZ | The "temporal dead zone": `let`/`const` throw if touched before initialisation |
| Run-to-completion | A synchronous block cannot be interrupted by other JS |
| `Symbol` identity | Symbols are unique and usable as non-colliding property keys |

## 2. Variables & Scope

| Keyword | Scope | Hoisted | Reassignable | Redeclarable |
|---|---|---|---|---|
| `var` | Function (or global) | Yes, as `undefined` | Yes | Yes |
| `let` | Block | Yes, but in TDZ | Yes | No |
| `const` | Block | Yes, but in TDZ | No (binding) | No |

| Concept | Description |
|---|---|
| `const` is not immutable | Prevents *rebinding*, not mutation: `const a = []` then `a.push(1)` is fine |
| Block scope | `{}`, `if`, `for`, `try` bodies create scope for `let`/`const` |
| Function scope | `var` leaks out of blocks to the enclosing function |
| Global scope | `var`/function declarations at top level attach to `globalThis` |
| Module scope | Each module has its own scope; nothing leaks implicitly |
| Closure capture | Functions capture variables by reference, not by value |
| IIFE | `(function(){ … })()` — the pre-module scoping idiom |
| Loop capture trap | `var` in a loop shares one binding; `let` creates one per iteration |

**Default to `const`; use `let` only when reassignment is required; avoid `var` entirely.**

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i));   // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i));   // 0 1 2
```

## 3. Data Types & Coercion

### Primitives (7) plus Object

| Type | Notes |
|---|---|
| `string` | Immutable UTF-16 sequence |
| `number` | IEEE-754 double; one numeric type for ints and floats |
| `bigint` **(ES2020)** | Arbitrary precision, written `123n`; cannot mix with `number` |
| `boolean` | `true` / `false` |
| `undefined` | Declared but unassigned |
| `null` | Intentional absence of a value |
| `symbol` **(ES2015)** | Unique, non-string property key |
| `object` | Everything else: objects, arrays, functions, dates, regexes, maps |

| Check | Result |
|---|---|
| `typeof "s"` | `"string"` |
| `typeof 1` / `typeof 1n` | `"number"` / `"bigint"` |
| `typeof undefined` | `"undefined"` |
| `typeof true` | `"boolean"` |
| `typeof Symbol()` | `"symbol"` |
| `typeof function(){}` | `"function"` |
| `typeof {}` / `typeof []` / `typeof null` | `"object"` (all three — a historic bug) |
| `Array.isArray(x)` | Correct array test |
| `x === null` | Correct null test |
| `Number.isNaN(x)` | Correct NaN test |
| `Object.is(a, b)` | `SameValue`: distinguishes `-0` from `0` and treats `NaN === NaN` |

### The falsy set (everything else is truthy)

| Falsy value | Notes |
|---|---|
| `false` | |
| `0` / `-0` / `0n` | Zero in all numeric forms |
| `""` | Empty string only — `" "` and `"0"` are truthy |
| `null` | |
| `undefined` | |
| `NaN` | |

**Traps:** `[]`, `{}`, `"0"`, `"false"`, and `new Boolean(false)` are all **truthy**.

### Coercion rules

| Expression | Result | Why |
|---|---|---|
| `1 + "2"` | `"12"` | `+` prefers string concatenation |
| `1 - "2"` | `-1` | `-` forces numeric conversion |
| `"3" * "4"` | `12` | `*` forces numeric conversion |
| `[] + []` | `""` | Both stringify to `""` |
| `[] + {}` | `"[object Object]"` | |
| `{} + []` | `"[object Object]"` or `0` | Parsed as a block, not an expression |
| `[1,2] + [3]` | `"1,23"` | Array `toString` then concat |
| `null + 1` | `1` | `null` → `0` |
| `undefined + 1` | `NaN` | `undefined` → `NaN` |
| `"5" == 5` | `true` | Loose equality coerces |
| `null == undefined` | `true` | Special-cased pair |
| `null == 0` | `false` | `null` only equals `undefined` |
| `NaN == NaN` | `false` | |
| `"" == 0` | `true` | Both → `0` |
| `[] == false` | `true` | Both → `""` / `0` |

**Rule: always use `===` / `!==`.** The only defensible `==` use is `x == null`, which tests for both `null` and `undefined` in one go.

## 4. Operators

| Category | Operators |
|---|---|
| Arithmetic | `+` `-` `*` `/` `%` `**` |
| Assignment | `=` `+=` `-=` `*=` `/=` `%=` `**=` |
| Comparison | `===` `!==` `==` `!=` `>` `<` `>=` `<=` |
| Logical | `&&` `||` `!` |
| Nullish | `??` **(ES2020)** |
| Bitwise | `&` `^` `~` `<<` `>>` `>>>` |
| Ternary | `cond ? a : b` |
| Comma | `a, b` (evaluates left to right) |
| Optional chaining | `?.` **(ES2020)** |
| Spread / rest | `...` **(ES2015)** |
| `typeof` / `instanceof` | Type and prototype checks |
| `in` | Own or inherited property exists |
| `delete` | Remove a property |
| `void` | Evaluate then yield `undefined` |

### Short-circuit and nullish behaviour

```js
a || b       // b if a is falsy  (0, "", false all trigger the fallback)
a ?? b       // b only if a is null or undefined  (ES2020)
a?.b.c       // undefined instead of TypeError if a is nullish
a?.[i]       // optional element access
fn?.()       // optional call

// Logical assignment (ES2021)
x ||= y      // x = x || y
x &&= y      // x = x && y
x ??= y      // x = x ?? y    <- the useful one for defaults
```

### Equality and bitwise gotchas

| Expression | Result |
|---|---|
| `0.1 + 0.2 === 0.3` | `false` — binary float rounding |
| `-0 === 0` | `true` (but `Object.is(-0, 0)` is `false`) |
| `1e21 + ""` | `"1e+21"` — exponential string above 21 digits |
| `(0.1 + 0.2).toFixed(2)` | `"0.30"` |
| `~x` | `-(x + 1)` |
| `1 << 31` | Negative — bitwise ops are 32-bit signed |

**Precision guard:** compare floats with a tolerance (`Math.abs(a - b) < 1e-9`) or work in integer units (cents, milliseconds).

## 5. Strings

| Method | Description |
|---|---|
| `s.length` | UTF-16 code-unit count (not characters — see `Intl.Segmenter`) |
| `s[i]` / `s.at(-1)` **(ES2022)** | Character or code unit at an index |
| `s.includes(sub)` | Substring test |
| `s.startsWith(p)` / `s.endsWith(p)` | Prefix/suffix test |
| `s.indexOf(sub, from)` / `s.lastIndexOf` | Position or `-1` |
| `s.search(re)` | Index of a regex match |
| `s.slice(a, b)` | Substring; negatives count from the end |
| `s.substring(a, b)` | Like `slice` but swaps/clamps negatives |
| `s.substr(a, len)` | Deprecated — avoid |
| `s.split(sep, limit)` | Split into an array |
| `s.replace(p, r)` | Replace first match (string or regex) |
| `s.replaceAll(p, r)` **(ES2021)** | Replace all matches |
| `s.match(re)` | Match array or `null` |
| `s.matchAll(re)` **(ES2020)** | Iterator of all matches (needs the `g` flag) |
| `s.trim()` / `trimStart()` / `trimEnd()` | Strip whitespace |
| `s.padStart(n, c)` / `padEnd(n, c)` | Pad to a width |
| `s.repeat(n)` | Repeat |
| `s.toUpperCase()` / `toLowerCase()` | Case conversion |
| `s.normalize(form)` | Unicode normalisation (NFC, NFD, …) |
| `s.localeCompare(o, loc)` | Locale-aware ordering |
| `s.charCodeAt(i)` / `s.codePointAt(i)` | UTF-16 unit / full codepoint |
| `String.raw` | Tagged template with no escape processing |
| `String.fromCharCode(...)` / `fromCodePoint(...)` | Build from codepoints |

### Template literals **(ES2015)**

```js
const msg = `Hello ${name}, you have ${n} item${n === 1 ? "" : "s"}`;

// Tagged template
const sql = String.raw`SELECT * FROM t WHERE x = ${x}`;   // note: raw ≠ safe
function tag(strings, ...values) { /* strings.raw available */ }
```

### Conversion

| Expression | Result |
|---|---|
| `String(123)` / `(123).toString()` | `"123"` |
| `Number("3.14")` | `3.14` |
| `Number("")` | `0` |
| `Number("abc")` | `NaN` |
| `parseInt("42px", 10)` | `42` — **always pass the radix** |
| `parseFloat("3.14abc")` | `3.14` |
| `"5" * 1` | `5` |
| `num.toFixed(2)` | String, 2 decimals |
| `(255).toString(16)` | `"ff"` |
| `(255).toString(2)` | `"11111111"` |

## 6. Numbers & Math

| Item | Description |
|---|---|
| `Number.MAX_SAFE_INTEGER` | 2^53 − 1 — integers above this lose precision |
| `Number.EPSILON` | Smallest representable difference from 1 |
| `Number.isInteger(x)` | Integer test (no coercion) |
| `Number.isFinite(x)` | Finite test (no coercion) |
| `Number.isNaN(x)` | NaN test (no coercion — prefer over global `isNaN`) |
| `Number.isSafeInteger(x)` | Within the safe integer range |
| `Number.parseInt` / `Number.parseFloat` | Namespaced versions |
| `n.toFixed(d)` / `toPrecision(p)` / `toExponential(d)` | Formatting to a string |
| `Math.abs/ceil/floor/round/trunc/sign` | Rounding and sign |
| `Math.min/max(...xs)` | Extremes (spreading a huge array risks a stack limit) |
| `Math.pow(a, b)` / `Math.sqrt/cbrt` | Powers and roots |
| `Math.hypot(...xs)` | Euclidean norm |
| `Math.exp/log/log2/log10` | Exponentials and logs |
| `Math.random()` | Pseudo-random in [0, 1) — **never for security** |
| `Math.imul` / `clz32` | 32-bit multiply / leading-zero count |
| `Math.PI` / `E` / `SQRT2` | Constants |
| `crypto.getRandomValues()` / `crypto.randomUUID()` | Cryptographically secure randomness |
| `Intl.NumberFormat` | Locale-aware currency/percent/decimal formatting |
| `1_000_000` **(ES2021)** | Numeric separator for readability |

```js
new Intl.NumberFormat("en-GB", { style: "currency", currency: "GBP" })
  .format(1234.5);                        // "£1,234.50"
crypto.randomUUID();                      // v4 UUID (browser & Node 14.17+)
```

## 7. Arrays

### Mutating methods

| Method | Description |
|---|---|
| `push(...x)` / `pop()` | Add / remove at the end |
| `unshift(...x)` / `shift()` | Add / remove at the start (O(n)) |
| `splice(start, del, ...items)` | Remove and/or insert anywhere; returns removed |
| `sort(cmp)` | In-place, stable **(ES2019)**; default is lexicographic |
| `reverse()` | In-place reversal |
| `fill(v, a, b)` | Fill a range |
| `copyWithin(t, a, b)` | Copy a range within the array |

### Non-mutating methods

| Method | Description |
|---|---|
| `slice(a, b)` | Shallow copy of a range |
| `concat(...xs)` | Join arrays/values |
| `join(sep)` | Stringify with a separator |
| `flat(depth)` **(ES2019)** | Flatten nested arrays |
| `flatMap(f)` **(ES2019)** | Map then flatten one level |
| `includes(v)` | Membership (`NaN`-aware, unlike `indexOf`) |
| `indexOf(v)` / `lastIndexOf(v)` | Position or `-1` |
| `at(i)` **(ES2022)** | Index with negative support |
| `find(fn)` / `findIndex(fn)` | First match |
| `findLast(fn)` **(ES2023)** / `findLastIndex` | Last match |
| `filter(fn)` | Keep matching elements |
| `toSorted(cmp)` **(ES2023)** | Copy, sorted |
| `toReversed()` **(ES2023)** | Copy, reversed |
| `toSpliced(...)` **(ES2023)** | Copy with a splice applied |
| `with(i, v)` **(ES2023)** | Copy with one element replaced |
| `keys()` / `values()` / `entries()` | Iterators |
| `toString()` | Comma-joined |

### Iteration & aggregation

| Method | Description |
|---|---|
| `forEach(fn)` | Side effects; no return value, cannot break |
| `map(fn)` | Transform each element |
| `reduce(fn, init)` / `reduceRight` | Fold to a single value |
| `every(fn)` / `some(fn)` | Universal / existential test |
| `Array.from(iterable, mapFn)` | Build from any iterable |
| `Array.of(...xs)` | Build from arguments (fixes `Array(3)` ambiguity) |
| `Array.isArray(x)` | Correct array test |
| `Array(n).fill(v)` | n-length pre-filled array |
| `Array.from({length: n}, (_, i) => i)` | Safe range builder |

```js
const nums = [3, 1, 2];
nums.sort();                 // [1, 2, 3]  (lexicographic — fine for 1-digit)
[10, 9, 1].sort();           // [1, 10, 9]  <- WRONG for numbers
[10, 9, 1].sort((a, b) => a - b);   // [1, 9, 10]

Array(3)                     // [ <3 empty items> ]  sparse, not [undefined x3]
Array(3).fill(0)             // [0, 0, 0]
[...Array(3)]                // [undefined, undefined, undefined]

// Holes behave inconsistently across methods
[1, , 3].length              // 3
[1, , 3].map(x => x * 2)     // [2, <empty>, 6]  — the map is skipped
```

**Copy warning:** `map`, `filter`, `slice`, `concat`, `flat`, and spread are all **shallow**. Nested objects remain shared.

## 8. Objects

| Operation | Description |
|---|---|
| `o.key` / `o["key"]` | Property access (bracket form for dynamic keys) |
| `o[key] = v` | Assign |
| `delete o.key` | Remove a property (slow — deoptimises the shape) |
| `"key" in o` | Exists as own or inherited |
| `Object.keys(o) / values(o) / entries(o)` | Own enumerable arrays |
| `Object.assign(t, ...srcs)` | Shallow merge (triggers setters) |
| `{...a, ...b}` **(ES2018)** | Shallow merge (does **not** trigger setters) |
| `Object.fromEntries(it)` **(ES2019)** | Build from `[k, v]` pairs |
| `Object.hasOwn(o, k)` **(ES2022)** | Safe own-property test |
| `Object.freeze(o)` | Prevent mutation (shallow) |
| `Object.seal(o)` | Prevent add/remove, allow writes |
| `Object.preventExtensions(o)` | Prevent adding properties |
| `Object.isFrozen/isSealed/isExtensible` | State queries |
| `Object.getPrototypeOf` / `setPrototypeOf` | Prototype access |
| `Object.create(proto, descs)` | Create with a chosen prototype |
| `Object.defineProperty(o, k, desc)` | Fine-grained property control |
| `Object.getOwnPropertyDescriptor(o, k)` | Read a descriptor |
| `Object.groupBy(it, fn)` **(ES2024)** | Group into an object of arrays |
| `Object.is(a, b)` | `SameValue` comparison |
| `Object.entries(o).forEach(([k, v]) => …)` | Enumerate pairs |

### Property descriptors

```js
Object.defineProperty(obj, "id", {
  value: 42,
  writable: false,
  enumerable: false,
  configurable: false,
});
```

### Getters, setters, computed keys, shorthand

```js
const key = "dyn";
const obj = {
  shorthand,                       // from a variable of the same name
  [key]: 1,                        // computed key
  method() { return 1; },          // method shorthand (no `function`)
  get double() { return this.n * 2; },
  set double(v) { this.n = v / 2; },
  ...spreadIn,                     // object spread
};
```

**Prototype pollution** is the big object-shaped vulnerability: merging untrusted data with keys like `__proto__` or `constructor` can poison every object in the process. Guard merges with `Object.hasOwn`, an allowlist of keys, or `Object.create(null)` for pure maps.

## 9. Destructuring, Spread & Rest

```js
// Array destructuring
const [a, b = 2, ...rest] = [1];
const [, second] = arr;                       // skip the first
;[x, y] = [y, x];                             // swap (assignment — `const` would redeclare; leading `;` guards ASI)

// Object destructuring
const { name, age = 0, addr: { city } = {} } = user;
const { a: renamed } = obj;
function f({ x, y }, [p, q]) { … }             // in parameters

// Spread (expand)
const merged = [...arr1, ...arr2];
const copy = { ...obj };
Math.max(...nums);
fn(...args);

// Rest (collect)
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
const { keep, ...dropped } = obj;              // object rest (ES2018)
```

**Gotcha:** `{ ...obj }` and `[...arr]` are **shallow**. Use `structuredClone(x)` (browser & Node 17+) for a deep copy of structured-cloneable data, or a library for anything exotic (class instances, functions, symbols).

## 10. Functions

| Form | Notes |
|---|---|
| Declaration | `function f() {}` — hoisted entirely |
| Expression | `const f = function () {}` — not hoisted |
| Named expression | `const f = function g() {}` — `g` visible only inside |
| Arrow | `const f = () => {}` **(ES2015)** — no own `this`/`arguments` |
| Method | `{ f() {} }` — has its own `this` |
| Generator | `function* g() {}` — yields values |
| Async | `async function f() {}` — returns a Promise |
| Async generator | `async function* g() {}` — async iteration |
| IIFE | `(() => {})()` — immediate invocation |
| Constructor | `new F()` — legacy; prefer `class` |

| Concept | Description |
|---|---|
| Default parameters | `function f(a = 1) {}` — evaluated per call |
| Rest parameters | `f(...args)` collects into a real array |
| `arguments` | Array-like, function-only — absent from arrows |
| Arity | `f.length` counts parameters before the first default/rest |
| `f.name` | The function's name |
| Closures | Inner functions retain access to outer bindings |
| Pure function | Same input → same output, no side effects |
| Higher-order | Accepts and/or returns functions |
| Currying | `f(a)(b)` via nested arrows |
| Memoisation | Cache results by argument key |
| Recursion | A stack frame per call; deep recursion can overflow |

**Arrow functions differ in four ways:** no own `this`, no `arguments`, no `new.target`, and they cannot be used as constructors.

```js
const memo = (fn) => {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (!cache.has(key)) cache.set(key, fn(...args));
    return cache.get(key);
  };
};
```

## 11. Control Flow

| Statement | Description |
|---|---|
| `if` / `else if` / `else` | Branching |
| Ternary | `cond ? a : b` |
| `switch` / `case` / `default` | Multi-branch; **falls through without `break`** |
| `switch (true)` | Pattern-matching idiom using boolean cases |
| `while` | Pre-test loop |
| `do…while` | Post-test loop (runs at least once) |
| `for` | Classic three-clause loop |
| `for…of` **(ES2015)** | Iterate values of any iterable |
| `for…in` | Iterate **enumerable string keys**, including inherited |
| `for await…of` **(ES2018)** | Async iteration |
| `break` / `continue` | Exit / skip, with optional labels |
| Label | `outer: for (…) { break outer; }` |
| `try` / `catch` / `finally` | Exception handling |
| `throw` | Raise any value (use `Error` instances) |
| `return` | Exit a function |
| `catch {}` **(ES2019)** | Optional catch binding |

**`for…in` is almost always wrong** for arrays — it includes inherited and non-index keys, and order is not fully guaranteed. Use `for…of`, or `Object.entries()` for objects.

## 12. Iterators & Generators

| Item | Description |
|---|---|
| Iterable protocol | An object with `Symbol.iterator` returning an iterator |
| Iterator protocol | An object with a `next()` returning `{value, done}` |
| Built-in iterables | Array, string, Map, Set, NodeList, generator results |
| Generators | `function*` — a resumable function |
| `yield` | Pause and emit a value |
| `yield*` | Delegate to another iterable |
| `gen.next()` | Advance; `{value, done}` |
| `gen.return(v)` | Finish early (runs `finally` blocks) |
| `gen.throw(e)` | Inject an exception at the pause point |
| Lazy evaluation | Values are produced on demand — good for large/infinite sequences |
| Async generators | `async function*` plus `for await…of` |

```js
function* range(start, end, step = 1) {
  for (let i = start; i < end; i += step) yield i;
}
[...range(0, 5)];                     // [0, 1, 2, 3, 4]

// Infinite sequence, consumed lazily
function* naturals() { let i = 0; while (true) yield i++; }
const take = (it, n) => { const out = []; for (const v of it) { if (out.length >= n) break; out.push(v); } return out; };
take(naturals(), 3);                  // [0, 1, 2]
```

## 13. Map, Set, WeakMap, WeakSet

| | Map | Object |
|---|---|---|
| Key types | Any value | Strings and symbols only |
| Key order | Insertion order, guaranteed | Insertion order for string keys |
| Size | `.size` | `Object.keys(o).length` |
| Prototype keys | None by default | Inherited keys pollute lookups |
| Iteration | Directly iterable | Needs `Object.entries` |
| JSON | Not directly | Native |

| Collection | Description |
|---|---|
| `new Map([[k, v], …])` | Key-value store with typed keys |
| `map.set(k, v)` / `get(k)` / `has(k)` / `delete(k)` / `clear()` | Operations |
| `map.size` | Entry count |
| `map.keys() / values() / entries()` | Iterators |
| `map.forEach((v, k) => …)` | Note the **value-first** argument order |
| `new Set(iterable)` | Unique values; insertion-ordered |
| `set.add(v)` / `has(v)` / `delete(v)` / `clear()` | Operations |
| `set.size` | Element count |
| `new WeakMap()` / `new WeakSet()` | Keys/elements must be objects (or non-registered symbols, ES2023) |
| Weak collections | Hold keys **weakly** — eligible for GC, not enumerable |

**Use cases:** `Map` for caches and frequency tables with object keys; `Set` for deduplication and O(1) membership; `WeakMap` for attaching private data to objects without leaking them.

```js
const unique = [...new Set([1, 1, 2, 3])];        // [1, 2, 3]
const freq = new Map();
for (const c of "abracadabra") freq.set(c, (freq.get(c) ?? 0) + 1);
```

## 14. Classes, Prototypes & Inheritance

| Feature | Description |
|---|---|
| `class C {}` **(ES2015)** | Syntactic sugar over prototype chains |
| `constructor()` | Initialiser |
| Instance fields | `count = 0` — declared in the class body **(ES2022)** |
| Static members | `static create() {}` — on the class, not instances |
| Private fields | `#secret` — truly inaccessible outside **(ES2022)** |
| Static private | `static #registry` |
| Static init block | `static { … }` **(ES2022)** |
| `get` / `set` | Accessors |
| `extends` | Inheritance |
| `super()` / `super.method()` | Parent constructor / method |
| `#x in obj` **(ES2022)** | Private-field brand check |
| `new.target` | Detection of `new` invocation |
| `instanceof` | Prototype-chain test |
| `Object.getPrototypeOf` | The actual prototype object |
| Mixins | Composing behaviour via function application |
| `Symbol.hasInstance` | Customise `instanceof` |

```js
class Shape {
  #id = crypto.randomUUID();              // private instance field
  static #count = 0;                       // private static

  constructor(name) {
    this.name = name;
    Shape.#count++;
  }

  static get count() { return Shape.#count; }
  get label() { return `${this.name} [${this.#id}]`; }

  area() { throw new Error("abstract"); }
}

class Circle extends Shape {
  constructor(r) { super("circle"); this.r = r; }
  area() { return Math.PI * this.r ** 2; }
}

Object.hasOwn(Circle.prototype, "area");    // true
```

**Class gotcha:** class declarations are **not hoisted** for use (TDZ), and a class body is always in strict mode. Methods are **not** bound — passing `obj.method` as a callback loses `this` unless you arrow-bind or wrap.

## 15. `this` and Explicit Binding

**Binding rules, in precedence order:**

| Rule | Example | `this` is |
|---|---|---|
| `new` | `new F()` | the newly created object |
| Explicit | `f.call(o)`, `f.apply(o)`, `f.bind(o)` | `o` |
| Implicit | `o.f()` | `o` |
| Default | `f()` | `undefined` in strict/modules, `globalThis` in sloppy mode |
| Arrow | `() => this` | the enclosing lexical `this` (no own binding) |

| Method | Description |
|---|---|
| `f.call(thisArg, ...args)` | Invoke with explicit `this` |
| `f.apply(thisArg, argsArray)` | Same, arguments as an array |
| `f.bind(thisArg, ...partial)` | Return a permanently bound copy |
| `f.bind()` partial application | Pre-fill leading arguments |

```js
const obj = { x: 1, get() { return this.x; } };
const { get } = obj;
get();                    // sloppy mode: undefined (this is globalThis); strict/module: TypeError
get.call(obj);            // 1
const bound = obj.get.bind(obj);
bound();                  // 1
[1, 2, 3].map(obj.get, obj);   // second arg to map is the thisArg
```

**Callbacks are the classic trap:** array methods, timers, and event handlers all invoke your function with their own `this` (or `undefined`). Use an arrow function, or `bind`.

## 16. Modules

### ES Modules (ESM, the standard)

```js
// named exports
export const PI = 3.14;
export function area(r) { return PI * r ** 2; }
export class Circle {}

// default export
export default function main() {}
export { thing as alias };
export * from "./other.js";              // re-export

// imports
import main, { PI, area as a } from "./math.js";
import * as math from "./math.js";
import "./side-effects.js";              // for side effects only
const mod = await import("./lazy.js");   // dynamic import (ES2020) — code splitting
```

| Property | ESM | CommonJS |
|---|---|---|
| Syntax | `import` / `export` | `require` / `module.exports` |
| Timing | Static, hoisted, async-resolved | Runtime, synchronous |
| Bindings | **Live** references | Copied values at import time |
| Top-level `await` | Yes **(ES2022)** | No |
| `this` at top level | `undefined` | `module.exports` |
| File extension | `.mjs` or `.js` with `"type":"module"` | `.cjs` or `.js` default |
| Node dirname | `import.meta.dirname` **(Node 20.11+)** | `__dirname` |
| Tree shaking | Yes | No |

**Key facts:** ESM bindings are **live** — if the exporting module reassigns, importers see the new value. ESM is always strict, and imports are hoisted above other statements. In Node, `import.meta.url` (a `file:` URL) replaces `__dirname`; convert with `fileURLToPath`.

## 17. Async JavaScript

### The three generations

| Style | Shape |
|---|---|
| Callbacks | `fs.readFile(path, (err, data) => {})` — error-first convention |
| Promises **(ES2015)** | `read(path).then(d => …).catch(e => …)` |
| `async`/`await` **(ES2017)** | `const d = await read(path)` inside an `async` function |

### Promises

| Member | Description |
|---|---|
| `new Promise((resolve, reject) => {})` | Create |
| `.then(onFulfilled, onRejected)` | Chain on success |
| `.catch(onRejected)` | Handle failure |
| `.finally(fn)` | Always runs |
| `Promise.resolve(v)` / `reject(e)` | Pre-settled promises |
| `Promise.all(iterable)` | All must succeed; **fails fast** on first rejection |
| `Promise.allSettled(iterable)` **(ES2020)** | Wait for all; never rejects |
| `Promise.race(iterable)` | Settles with the first to settle |
| `Promise.any(iterable)` **(ES2021)** | First to **fulfil**; rejects only if all fail |
| `Promise.withResolvers()` **(ES2024)** | Expose `{promise, resolve, reject}` |
| `await` | Unwrap; rejection throws |

**States:** pending → fulfilled or rejected, and **settled is permanent** — a promise never changes state twice.

```js
const [user, posts] = await Promise.all([getUser(), getPosts()]);   // parallel
const results = await Promise.allSettled(tasks);                    // never throws
const first = await Promise.race([fetchA(), fetchB()]);             // first to settle

// Sequential vs parallel — a common performance bug
for (const id of ids) await fetch(id);            // slow: one at a time
await Promise.all(ids.map(id => fetch(id)));      // fast: concurrent
```

**Unhandled rejections crash Node** (and log prominently in browsers). Always attach a handler, or use `process.on("unhandledRejection")` as a last-resort net.

### Async/await patterns

```js
async function load() {
  try {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);   // fetch does NOT throw on 4xx/5xx
    return await res.json();
  } catch (err) {
    if (err instanceof TypeError) { /* network/parse failure */ }
    throw err;
  } finally {
    cleanup();
  }
}

// Sequential with early exit
for (const url of urls) {
  if (await check(url)) break;
}

// Async iteration
for await (const chunk of stream) process(chunk);

// Concurrency limit — batching avoids flooding the target
async function pool(items, limit, fn) {
  const out = [];
  for (let i = 0; i < items.length; i += limit) {
    out.push(...await Promise.all(items.slice(i, i + limit).map(fn)));
  }
  return out;
}
```

### Cancellation

```js
const ac = new AbortController();
setTimeout(() => ac.abort(), 5000);
const res = await fetch(url, { signal: ac.signal });
const check = await fetch(url, { signal: AbortSignal.timeout(5000) });  // simpler
```

## 18. Error Handling

| Statement | Description |
|---|---|
| `try { } catch (e) { } finally { }` | Handle and clean up |
| `throw new Error("msg")` | Raise (always throw `Error` instances, not strings) |
| `err.message` | Human-readable text |
| `err.name` | `"TypeError"`, `"RangeError"`, … |
| `err.stack` | Stack trace (non-standard but universal) |
| `err.cause` **(ES2022)** | Wrap an underlying error |
| `AggregateError` **(ES2021)** | Bundle several errors (from `Promise.any`) |
| `instanceof` | Discriminate error types |
| `err.code` | Node's machine-readable error code (`"ENOENT"`) |
| Global handlers | `window.onerror`, `unhandledrejection`, `process.on("uncaughtException")` |

| Built-in error | Raised when |
|---|---|
| `TypeError` | Wrong type or missing property/method call |
| `ReferenceError` | Undefined variable referenced |
| `RangeError` | Value out of allowed range (e.g. `new Array(-1)`) |
| `SyntaxError` | Invalid syntax or malformed `JSON.parse` input |
| `URIError` | Malformed URI passed to encode/decode |
| `EvalError` | Legacy; from `eval` misuse |
| `AggregateError` | Multiple errors grouped |

**Patterns:** validate inputs at boundaries, wrap third-party errors with `cause`, and never `catch` and silently swallow. For resource cleanup use `using`/`await using` (Explicit Resource Management) where your runtime supports it, or a `try/finally`.

## 19. JSON

| Function | Description |
|---|---|
| `JSON.stringify(v)` | Serialise to a string |
| `JSON.stringify(v, null, 2)` | Pretty-print with 2-space indent |
| `JSON.stringify(v, replacer, n)` | `replacer` filters keys or transforms values |
| `JSON.stringify(v, (k, val) => val, 2)` | Function replacer |
| `JSON.parse(s)` | Parse to a value |
| `JSON.parse(s, reviver)` | Transform on parse (e.g. revive dates) |

**What `JSON.stringify` drops or changes:**

| Input | Result |
|---|---|
| `undefined`, functions, symbols (as values) | Omitted from objects; become `null` in arrays |
| `NaN`, `Infinity`, `-Infinity` | `null` |
| `BigInt` | **Throws** `TypeError` |
| `Date` | ISO string via `toJSON()` |
| `Map` / `Set` | `{}` (unless a `toJSON` exists) |
| Circular references | **Throws** `TypeError` |
| Class instances | Own enumerable properties only — prototype is lost |

```js
const reviver = (k, v) =>
  typeof v === "string" && /^\d{4}-\d{2}-\d{2}T/.test(v) ? new Date(v) : v;
const obj = JSON.parse(text, reviver);          // revive dates
```

For anything more demanding — `Map`/`Set`, `BigInt`, dates, circular structures — use `structuredClone` (in-process) or a library like `superjson`/`devalue` (over the wire).

## 20. Regular Expressions

| Syntax | Meaning |
|---|---|
| `/\d+/` | Literal |
| `new RegExp(pattern, flags)` | Constructed (escaped dynamically) |
| `.` | Any char except newline (with `s`, includes it) |
| `\d` `\w` `\s` | Digit, word char, whitespace |
| `\D` `\W` `\S` | Negations |
| `\b` | Word boundary |
| `^` `$` | Start / end (per line with `m`) |
| `*` `+` `?` | Zero+, one+, zero-or-one |
| `{n}` `{n,}` `{n,m}` | Explicit quantifiers |
| `*?` `+?` `??` | Lazy variants |
| `[...]` `[^...]` | Character class / negation |
| `(a)` | Capturing group |
| `(?:a)` | Non-capturing group |
| `(?<name>a)` **(ES2018)** | Named capture |
| `(?=a)` `(?!a)` | Lookahead / negative lookahead |
| `(?<=a)` `(?<!a)` **(ES2018)** | Lookbehind / negative lookbehind |
| `\|` | Alternation |
| `\p{L}` `\p{Emoji}` | Unicode property escape (needs `u`) |

| Flag | Meaning |
|---|---|
| `g` | Global — find all matches |
| `i` | Case-insensitive |
| `m` | `^`/`$` match line boundaries |
| `s` | Dotall — `.` matches newlines |
| `u` | Full Unicode mode (correct for astral characters) |
| `y` | Sticky — match exactly at `lastIndex` |
| `d` | Indices for match positions **(ES2022)** |

```js
const re = /(?<year>\d{4})-(?<month>\d{2})/;
const { groups: { year, month } } = re.exec("2026-10");

// matchAll requires the g flag and returns an iterator
for (const m of "a1b2".matchAll(/([a-z])(\d)/g)) console.log(m[1], m[2]);

// Named groups in replacement
"2026-10".replace(re, "$<month>/$<year>");       // "10/2026"

// Escape user input before building a regex
const esc = (s) => s.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
```

**Two traps:** a regex with `g` carries `lastIndex` state, so reusing it across calls (`re.test()` in a loop) yields alternating results — create a fresh regex or reset `lastIndex`. And **catastrophic backtracking** on nested quantifiers (`(a+)+$`) can hang the single-threaded runtime; prefer explicit, bounded patterns.

## 21. Dates & Times

| Item | Description |
|---|---|
| `Date.now()` | Milliseconds since the epoch (UTC) — cheapest option |
| `new Date()` | Current time |
| `new Date(ms)` / `new Date("2026-10-15")` | From a timestamp or ISO string |
| `new Date(y, m, d, …)` | Local components — **month is 0-indexed** |
| `d.getTime()` | Epoch milliseconds |
| `d.toISOString()` | UTC ISO-8601 string |
| `d.toJSON()` | Same as `toISOString` |
| `d.getFullYear/Month/Date/Day/Hours/…` | Local components |
| `d.getUTCFullYear/…` | UTC components |
| `d.setTime/setFullYear/…` | Mutating setters |
| `d.toLocaleDateString(loc, opts)` | Locale-formatted |
| `Intl.DateTimeFormat` | Full locale/timezone formatting control |
| `Intl.RelativeTimeFormat` | "3 days ago" |
| `Temporal` | The successor API — **not yet baseline**; check availability |
| `performance.now()` | High-resolution monotonic timer for durations |

**Date is a minefield:**

- Month is **0-based** (`new Date(2026, 0)` is January), but `getDate()` is 1-based.
- `new Date("2026-10-15")` parses as **UTC midnight**; `new Date("2026-10-15T00:00")` parses as **local** time. A classic off-by-one-day bug.
- `new Date("10/15/2026")` is implementation-defined; **avoid non-ISO strings entirely**.
- Dates are **mutable** — passing one around hands out write access. Prefer epoch numbers or strings at boundaries.
- No built-in duration arithmetic: `+ 1 day` means `+ 86_400_000` ms, which breaks across DST boundaries.

```js
// Duration measurement — use performance.now(), not Date.now()
const t0 = performance.now();
work();
console.log(`${(performance.now() - t0).toFixed(1)}ms`);

// Format for a specific zone and locale
new Intl.DateTimeFormat("en-GB", {
  dateStyle: "medium", timeStyle: "short", timeZone: "Europe/London",
}).format(new Date());
```

## 22. DOM & Browser APIs

| Task | API |
|---|---|
| Select one | `document.querySelector(sel)` |
| Select many | `document.querySelectorAll(sel)` → static NodeList |
| By id | `document.getElementById(id)` |
| Create | `document.createElement(tag)` |
| Insert | `parent.append(el)`, `.prepend()`, `.before()`, `.after()` |
| Remove | `el.remove()` |
| Replace | `el.replaceWith(other)` |
| Clone | `el.cloneNode(true)` |
| Text | `el.textContent` (safe) |
| HTML | `el.innerHTML` (**XSS risk** with untrusted data) |
| HTML fragment | `el.insertAdjacentHTML(pos, html)` |
| Attributes | `el.getAttribute/setAttribute/removeAttribute` |
| Data attributes | `el.dataset.userId` ↔ `data-user-id` |
| Classes | `el.classList.add/remove/toggle/contains` |
| Styles | `el.style.color`, `getComputedStyle(el)` |
| Dimensions | `el.getBoundingClientRect()`, `offsetWidth/Height` |
| Traverse | `.parentElement`, `.children`, `.closest(sel)`, `.matches(sel)` |
| Traverse (nodes) | `firstChild`, `nextSibling` (vs `firstElementChild`, `nextElementSibling`) |
| Focus | `el.focus()`, `document.activeElement` |
| Scroll | `el.scrollIntoView()`, `window.scrollTo()` |
| Batch DOM writes | `document.createDocumentFragment()` |

### Events

| API | Description |
|---|---|
| `el.addEventListener(type, fn, opts)` | Subscribe |
| `el.removeEventListener(type, fn)` | Unsubscribe (same function reference required) |
| `event.target` | The element that dispatched the event |
| `event.currentTarget` | The element whose listener is running |
| `event.preventDefault()` | Cancel default behaviour |
| `event.stopPropagation()` | Stop bubbling to ancestors |
| `event.stopImmediatePropagation()` | Also stop remaining listeners on this element |
| `{ once: true }` | Auto-remove after firing |
| `{ capture: true }` | Listen in the capture phase |
| `{ passive: true }` | Promise not to `preventDefault()` — enables scroll performance |
| Event delegation | One listener on a parent, dispatched via `target.closest()` |
| `CustomEvent` | User-defined events with a `detail` payload |
| `AbortSignal` in `addEventListener` | Remove many listeners at once |

**Phases:** capture (top-down) → target → bubble (bottom-up). Most listeners default to the bubble phase.

```js
// Event delegation — one listener handles all current and future rows
table.addEventListener("click", (e) => {
  const row = e.target.closest("tr[data-id]");
  if (!row) return;
  select(row.dataset.id);
});

// Custom events
el.dispatchEvent(new CustomEvent("selected", { detail: { id }, bubbles: true }));
```

**Never** build HTML by concatenating untrusted strings into `innerHTML`. Use `textContent`, or sanitise with a maintained library (DOMPurify). The same rule applies to `element.setAttribute("href", userInput)` — `javascript:` URLs execute.

### Web platform APIs

| Area | API |
|---|---|
| Network | `fetch`, `Request`, `Response`, `Headers`, `AbortController` |
| Storage (sync) | `localStorage`, `sessionStorage` — strings only, ~5 MB |
| Storage (async) | `IndexedDB`, Cache API |
| Cookies | `document.cookie` — awkward; prefer `Set-Cookie` server-side |
| URL | `new URL(s, base)`, `URLSearchParams` |
| Clipboard | `navigator.clipboard.writeText()` |
| Notifications | `Notification.requestPermission()` |
| Geolocation | `navigator.geolocation` |
| Observers | `IntersectionObserver`, `ResizeObserver`, `MutationObserver` |
| Animation | `requestAnimationFrame`, Web Animations API |
| Workers | `new Worker()`, Shared Worker, Service Worker |
| Messaging | `postMessage`, `MessageChannel`, `BroadcastChannel` |
| Media | `navigator.mediaDevices.getUserMedia()` |
| Canvas | `getContext("2d")`, `getContext("webgl2")` |
| File | `File`, `Blob`, `FileReader`, drag-and-drop |
| Crypto | `crypto.subtle` (async), `crypto.getRandomValues` (sync) |
| Streams | `ReadableStream`, `WritableStream`, `TransformStream` |

```js
// IntersectionObserver — the correct way to lazy-load or dock on scroll
const io = new IntersectionObserver((entries) => {
  for (const e of entries) if (e.isIntersecting) load(e.target);
}, { rootMargin: "200px" });
document.querySelectorAll("img[data-src]").forEach(el => io.observe(el));

// Build a URL safely
const u = new URL("/search", "https://example.com");
u.searchParams.set("q", userInput);
```

## 23. Node.js

| Built-in module | Purpose |
|---|---|
| `node:fs` / `node:fs/promises` | Filesystem, sync and async |
| `node:path` | Cross-platform path manipulation |
| `node:os` | Platform, CPUs, memory, home dir |
| `node:process` | `argv`, `env`, `exit`, `cwd`, streams |
| `node:url` | URL parsing, `fileURLToPath` |
| `node:http` / `node:https` | HTTP servers and clients |
| `node:crypto` | Hashing, HMAC, random, ciphers |
| `node:stream` | Streaming with backpressure |
| `node:buffer` | Binary data |
| `node:events` | EventEmitter |
| `node:util` | `promisify`, `inspect`, `parseArgs` |
| `node:child_process` | Spawn and exec external processes |
| `node:worker_threads` | Real parallelism within one process |
| `node:readline` | Line-by-line terminal/file input |
| `node:zlib` | Compression |
| `node:timers/promises` | Promise-based `setTimeout`, `setInterval` |
| `node:test` | Built-in test runner |
| `node:assert` | Assertions with `assert/strict` |
| `node:net` / `node:dgram` | TCP / UDP |
| `node:dns` | DNS resolution |
| `node:cluster` | Multi-core server scaling |
| `node:v8` | Heap statistics and tuning |
| `node:perf_hooks` | High-resolution timing, performance entries |
| `node:module` | Module system hooks |
| `node:async_hooks` / `node:diagnostics_channel` | Context tracing |

### Common Node snippets

```js
import fs from "node:fs/promises";              // or: const fs = require("node:fs/promises")
import path from "node:path";
import { fileURLToPath } from "node:url";

// Read a file
const text = await fs.readFile("data.txt", "utf8");
const data = JSON.parse(text);

// Write & read JSON safely
await fs.writeFile("out.json", JSON.stringify(data, null, 2) + "\n");
const cfg = JSON.parse(await fs.readFile("package.json", "utf8"));

// Directory listing with metadata
for (const entry of await fs.readdir(".", { withFileTypes: true })) {
  if (entry.isFile()) console.log(entry.name);
}

// ESM-safe __dirname
const __dirname = path.dirname(fileURLToPath(import.meta.url));
// Node 20.11+: import.meta.dirname  and  import.meta.filename

// Atomic write
const tmp = "out.json.tmp";
await fs.writeFile(tmp, json);
await fs.rename(tmp, "out.json");

// Subprocess without a shell (no injection risk)
import { execFile } from "node:child_process";
import { promisify } from "node:util";
const run = promisify(execFile);
const { stdout } = await run("git", ["rev-parse", "HEAD"]);

// Streaming a large file line by line
import { createReadStream } from "node:fs";
import { createInterface } from "node:readline";
const rl = createInterface({ input: createReadStream("big.log"), crlfDelay: Infinity });
for await (const line of rl) if (line.includes("ERROR")) console.log(line);

// Parallel CPU work without blocking the loop
import { Worker } from "node:worker_threads";
const worker = new Worker(new URL("./worker.js", import.meta.url));
worker.postMessage(data);
worker.on("message", (result) => console.log(result));
```

| Node global | Description |
|---|---|
| `process.argv` | Command-line arguments (`argv[2]` is the first user arg) |
| `process.env` | Environment variables (all strings) |
| `process.exitCode` | Preferred over `process.exit()` — lets output flush |
| `process.cwd()` | Current working directory |
| `process.platform` / `arch` | OS and CPU architecture |
| `process.version` | Node version string |
| `process.hrtime.bigint()` | Monotonic nanoseconds |
| `process.nextTick(cb)` | Starvation-prone microtask-like queue |
| `setImmediate(cb)` | Run in the check phase |
| `Buffer.from(...)` | Binary data |
| `globalThis` | The global object |
| `console.table/log/time/timeEnd` | Diagnostics |
| `structuredClone(x)` | Deep clone **(Node 17+)** |
| `fetch` | Global fetch **(Node 18+)** |
| `AbortSignal.timeout(ms)` | Timeout signal |
| Event loop args | `--max-old-space-size=4096` raises the heap |

**`Buffer` vs string:** `Buffer.concat`, `buf.subarray`, and `buf.toString("utf8")`. Note that reading a partially-encoded UTF-8 multibyte sequence per chunk can produce replacement characters — use a `StringDecoder` or a transform stream for correctness.

## 24. The Event Loop & Concurrency

**Phases of the Node event loop, in order:**

| Phase | Handles |
|---|---|
| Timers | `setTimeout`, `setInterval` callbacks that are due |
| Pending callbacks | Deferred I/O errors |
| Idle / prepare | Internal |
| Poll | Retrieve new I/O events; run I/O callbacks |
| Check | `setImmediate` callbacks |
| Close callbacks | Socket close handlers |

**Between every phase, and after every macrotask, the microtask queue drains completely.** Microtasks are: promise reactions, `queueMicrotask`, `await` continuations, `MutationObserver`.

```js
console.log("1");
setTimeout(() => console.log("2"));        // macrotask
Promise.resolve().then(() => console.log("3"));  // microtask
queueMicrotask(() => console.log("4"));
console.log("5");
// Output: 1 5 3 4 2
```

**Consequences that matter in practice:**

- A long synchronous loop **blocks** timers, rendering, and I/O — split work with `await` yields or a Worker.
- Microtasks can **starve** the loop: an infinite chain of resolved promises never yields to timers.
- Recursive `process.nextTick` has the same starvation hazard, with higher priority than promises.
- CPU-bound work belongs in `worker_threads` (Node) or a Web Worker (browser), never in the main thread.
- `await` inside a loop is sequential; `Promise.all` is concurrent.

## 25. Modern Syntax Cheat-Sheet

| Feature | Since | Example |
|---|---|---|
| Optional chaining | ES2020 | `a?.b?.[i]?.()` |
| Nullish coalescing | ES2020 | `x ?? fallback` |
| Logical assignment | ES2021 | `x ??= y` |
| `BigInt` | ES2020 | `9007199254740993n` |
| `globalThis` | ES2020 | Uniform global reference |
| Dynamic `import()` | ES2020 | `await import("./m.js")` |
| `String.replaceAll` | ES2021 | `s.replaceAll("a", "b")` |
| Numeric separators | ES2021 | `1_000_000` |
| `Promise.any` | ES2021 | First fulfilment wins |
| `Array.at` | ES2022 | `arr.at(-1)` |
| `Object.hasOwn` | ES2022 | Safer than `hasOwnProperty` |
| Top-level `await` | ES2022 | In modules only |
| Class private fields | ES2022 | `#secret` |
| Static class blocks | ES2022 | `static { … }` |
| `err.cause` | ES2022 | Error chaining |
| `findLast` / `findLastIndex` | ES2023 | Search from the end |
| `toSorted` / `toReversed` / `with` | ES2023 | Immutable array ops |
| Hashbang `#!` | ES2023 | Executable scripts |
| `Object.groupBy` / `Map.groupBy` | ES2024 | Grouping without a reduce |
| `Promise.withResolvers` | ES2024 | Deferred pattern |
| `Array.fromAsync` | ES2024 | Async iterable to array |
| `structuredClone` | Runtime | Deep clone |
| `Array.prototype.includes` | ES2016 | `NaN`-aware membership |
| `Object.entries` | ES2017 | Pair iteration |
| `flat` / `flatMap` | ES2019 | Flattening |
| `Object.fromEntries` | ES2019 | Pairs to object |
| `Intl.Segmenter` | ES2022 | Grapheme-aware splitting |
| `Set` operations | ES2025 | `union`, `intersection`, `difference` |

## 26. Tooling & Ecosystem

| Category | Options |
|---|---|
| Package managers | npm, yarn (classic/berry), **pnpm**, bun, deno |
| Bundlers | Vite, esbuild, Rollup, webpack, rspack, Parcel, Turbopack |
| Transpilers | Babel, SWC, esbuild, tsc |
| Linters | ESLint, oxlint, Biome, StandardJS |
| Formatters | Prettier, Biome, dprint |
| Type checkers | TypeScript (tsc), JSDoc + `checkJs` |
| Test runners | **Vitest**, Jest, Mocha, `node:test` |
| E2E testers | Playwright, Cypress, WebdriverIO |
| Reactivity | React, Vue, Svelte, Solid, Angular |
| Meta-frameworks | Next.js, Nuxt, SvelteKit, Remix, Astro |
| Server runtimes | Node, Deno, Bun, Cloudflare Workers, Vercel Edge |
| Validation | Zod, Valibot, Yup, AJV |
| Logging | Pino, Winston, consola |
| Date/time | date-fns, Day.js, Luxon |
| HTTP clients | fetch, axios, ky, undici |
| Build/test infra | Turborepo, Nx, Changesets, Volta, corepack |

### npm commands

| Command | Description |
|---|---|
| `npm init -y` | Create `package.json` with defaults |
| `npm install` | Install from `package.json` |
| `npm install pkg` | Add a dependency |
| `npm install -D pkg` | Add a dev dependency |
| `npm install -g pkg` | Install globally |
| `npm ci` | Clean install **exactly** from the lockfile — use in CI |
| `npm uninstall pkg` | Remove |
| `npm update` | Update within semver ranges |
| `npm outdated` | Show what's behind |
| `npm run <script>` | Run a `package.json` script |
| `npm run` | List available scripts |
| `npm test` | Run the `test` script |
| `npm exec pkg` / `npx pkg` | Run a package binary |
| `npm ls pkg` | Show the dependency tree |
| `npm why pkg` | Explain why a package is installed |
| `npm audit` | Report known vulnerabilities |
| `npm audit fix --force` | Attempt fixes (**may break things**) |
| `npm dedupe` | Flatten duplicate versions |
| `npm pack` | Produce a publishable tarball |
| `npm publish --access public` | Publish |
| `npm version minor` | Bump the version and tag |
| `npm view pkg versions` | Query the registry |
| `npm link` | Symlink a local package globally |
| `npm cache clean --force` | Clear the cache |
| `npm pkg set scripts.dev="vite"` | Edit `package.json` from the CLI |

### `package.json` fields that matter

| Field | Purpose |
|---|---|
| `name` / `version` | Identity (npm requires semver) |
| `private: true` | Prevent accidental publishing |
| `type: "module"` | Treat `.js` as ESM |
| `main` | CommonJS entry |
| `module` | ESM entry (bundler convention) |
| `exports` | Conditional entry map — the modern, authoritative field |
| `browser` | Browser-specific entry |
| `types` | TypeScript declarations entry |
| `bin` | CLI executables |
| `files` | Publish allowlist (smaller, safer packages) |
| `scripts` | Task runner entries |
| `dependencies` | Runtime requirements |
| `devDependencies` | Build/test-only requirements |
| `peerDependencies` | Host-provided (plugins) |
| `optionalDependencies` | Failure tolerated |
| `engines` | Runtime version constraints |
| `workspaces` | Monorepo package globs |
| `sideEffects` | Enables tree shaking |
| `overrides` | Force transitive versions |
| `packageManager` | Pin the PM + version for corepack |

### Semver ranges

| Range | Meaning |
|---|---|
| `1.2.3` | Exact |
| `^1.2.3` | Compatible with 1.x.x (no breaking change) |
| `~1.2.3` | Patch-level only: 1.2.x |
| `>=1.2.3 <2` | Explicit range |
| `1.2.x` | Wildcard patch |
| `*` / `latest` | Any — avoid |
| `workspace:*` | Monorepo-local reference (pnpm/yarn) |

**Lockfiles** (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lock`) pin the full resolved tree. **Commit them**, and use `npm ci` in CI so builds are reproducible.

## 27. CLI Commands

| Command | Description |
|---|---|
| `node script.js` | Run a script |
| `node -e "code"` | Evaluate inline |
| `node -p "expr"` | Evaluate and print the result |
| `node --check file.js` | Syntax-check without running |
| `node --watch script.js` | Auto-restart on change **(since 18.11, stable in 22+)** |
| `node --inspect-brk script.js` | Debugger, breaking on the first line |
| `node --test` | Run the built-in test runner **(Node 20+)** |
| `node --test --experimental-test-coverage` | Tests with coverage |
| `node --env-file=.env app.js` | Load env vars from a file **(Node 20.6+)** |
| `node --max-old-space-size=4096 app.js` | Raise the heap limit |
| `node --input-type=module -e "…"` | Force ESM for inline code |
| `node -r ./register.js app.js` | Preload a module |
| `node --experimental-strip-types app.ts` | Run TS by stripping types **(Node 22.6+)** |
| `node --conditions=development` | Resolve a custom `exports` condition |
| `node --trace-warnings app.js` | Show warning origins |
| `tsc --init` / `tsc --noEmit` | Initialise / type-check without emitting |
| `tsc --watch` | Incremental type checking |
| `eslint . --fix` | Lint and autofix |
| `eslint --init` | Create a config (flat config) |
| `prettier --write .` | Format in place |
| `prettier --check .` | Verify formatting |
| `biome check --write .` | Lint + format in one tool |
| `vitest run` / `vitest` | Test once / watch mode |
| `vitest --coverage` | Coverage report |
| `jest --watch` / `jest -u` | Watch / update snapshots |
| `playwright test` / `--ui` | Run E2E / interactive UI |
| `vite` / `vite build` / `vite preview` | Dev server / production build / preview |
| `esbuild in.js --bundle --minify --outfile=out.js` | Bundle and minify |
| `bun run script.ts` / `bun install` | Bun runtime and PM |
| `deno run --allow-net app.ts` | Deno with explicit permissions |
| `corepack enable` / `pnpm install` | Manage PM versions / install |
| `pm2 start app.js -i max` | Process manager with clustering |
| `nodemon app.js` | Restart-on-change dev server |

**Debugging tip:** `--inspect-brk` plus Chrome DevTools (`chrome://inspect`) gives breakpoints, step-through, and profiling for server code. For a one-off, `node -p` is faster than writing a file.

## 28. Common Pitfalls

| Pitfall | Why it bites | Fix |
|---|---|---|
| `this` in callbacks | Callbacks get their own `this` | Arrow function or `bind` |
| `==` coercion | Surprising equality (`"" == 0`) | Use `===` |
| `typeof null === "object"` | Historic bug | `x === null` |
| `NaN !== NaN` | Equality always fails | `Number.isNaN` |
| `[10,9,1].sort()` | Default sort is lexicographic | `sort((a,b)=>a-b)` |
| `arr.sort()` mutates | Shared arrays reorder silently | `toSorted()` |
| Mutable default-shaped objects | Spread/assign/`map` are shallow | `structuredClone` |
| `0.1 + 0.2 !== 0.3` | Binary float rounding | Tolerance or integer units |
| `JSON.stringify` on `Map`/`Set` | Yields `{}` | Convert to entries first |
| `JSON.stringify` on `BigInt` | Throws | Convert to string |
| Circular JSON | Throws | Replacer, or a graph-aware serialiser |
| `fetch` on HTTP 404 | Does not reject | Check `response.ok` |
| Missing `await` | Silent unresolved promise | `no-floating-promises` lint rule |
| Sequential `await` in a loop | Far slower than necessary | `Promise.all` |
| Unhandled rejection | Crashes Node | Attach `.catch` |
| `var` in loops | Shared binding across iterations | `let` |
| Regex with `g` reused | `lastIndex` state leaks | Fresh regex per call |
| Regex catastrophic backtracking | Hangs the single thread | Bounded patterns |
| `innerHTML` with user input | XSS | `textContent` or sanitise |
| `for…in` on arrays | Includes inherited/prototype keys | `for…of` |
| `delete` on an array | Leaves holes, deoptimises | `splice` or `filter` |
| Missing radix in `parseInt` | Legacy octal surprises | `parseInt(s, 10)` |
| `new Date("YYYY-MM-DD")` | Parses as UTC; off-by-one day | Parse ISO with a library |
| Blocking the event loop | Freezes UI / stalls I/O | Chunk work or use workers |
| Float accumulation in money | Rounding drift | Integer cents or a decimal library |
| Prototype pollution | Unfiltered deep merge | Validate keys, `Object.create(null)` |
| Comparing objects with `===` | Compares references | Compare fields or a deep-equal |
| Leaking listeners/timers | Memory growth and duplicate work | `removeEventListener`, `clearInterval` |
| `process.exit()` before flush | Truncated stdout | Set `process.exitCode` instead |

## Appendix A: Array & Object Method Quick Reference

```
MUTATING       push pop shift unshift splice sort reverse fill copyWithin
NON-MUTATING   slice concat flat flatMap join includes indexOf at
NEW-COPY       toSorted toReversed toSpliced with
SEARCH         find findIndex findLast findLastIndex filter some every
TRANSFORM      map flatMap reduce reduceRight
ITERATE        forEach entries keys values
BUILD          Array.from Array.of Array.isArray spread

OBJECT STATICS keys values entries assign create defineProperty
               fromEntries hasOwn freeze seal isFrozen getPrototypeOf
               groupBy is
```

## Appendix B: Promise Combinators at a Glance

| Combinator | Resolves when | Rejects when | Returns |
|---|---|---|---|
| `all` | All fulfil | Any rejects (fail fast) | Array of values |
| `allSettled` | All settle | Never | Array of status objects |
| `race` | First settles | First settles if rejected | The winning value |
| `any` | First fulfils | All reject | The first value, or `AggregateError` |
| `withResolvers` | — | — | `{promise, resolve, reject}` |

| Status object | Shape |
|---|---|
| Fulfilled | `{status: "fulfilled", value}` |
| Rejected | `{status: "rejected", reason}` |

## Appendix C: Environment Differences

| Concern | Browser | Node |
|---|---|---|
| Global object | `window` / `self` | `globalThis` |
| Module system | ESM only | ESM and CommonJS |
| File access | None (sandboxed) | `node:fs` |
| DOM | Yes | No |
| `__dirname` | N/A | `__dirname` (CJS), `import.meta.dirname` (ESM) |
| Env vars | Not available | `process.env` |
| Entry guards | N/A | `import.meta.url === pathToFileURL(process.argv[1]).href` |
| Timers | Clamped in background tabs | Full precision |
| Blocking the loop | Freezes the UI | Stalls all requests |
| Crypto | `crypto.subtle` (async) | `node:crypto` |
| `require` | No | Yes (CJS) |
| Network | CORS-restricted | Unrestricted |

```js
// Detect the runtime
const isNode = typeof process !== "undefined" && process.versions?.node;
const isBrowser = typeof window !== "undefined" && typeof document !== "undefined";
```

## Appendix D: TypeScript Primer

TypeScript is a typed superset that compiles to JavaScript. It adds nothing at runtime — types are erased — but catches a large class of errors before execution.

| Concept | Description |
|---|---|
| `x: number` | Type annotation |
| `f(x: string): void` | Parameter and return types |
| `interface I {}` / `type T = {}` | Object shapes (interface is extendable/mergeable) |
| `type A = B \| C` | Union |
| `type A = B & C` | Intersection |
| `type U = "a" \| "b"` | Literal union |
| `type R = { readonly id: string }` | Read-only member |
| `?` | Optional member or parameter |
| `T[]` / `Array<T>` | Array types |
| `[string, number]` | Tuple |
| `keyof T` | Union of keys |
| `typeof x` | Type of a value |
| `as const` | Literal, deeply-readonly inference |
| `satisfies T` | Validate against a type **without** widening |
| `Partial<T>` / `Required<T>` | Optional-ise / require all |
| `Readonly<T>` | Immutable-ise |
| `Pick<T, K>` / `Omit<T, K>` | Subset / exclude keys |
| `Record<K, V>` | Mapped key-value type |
| `ReturnType<F>` / `Parameters<F>` | Extract function types |
| `Awaited<T>` | Unwrap a promise type |
| Generics | `<T>(x: T) => T` |
| Constraints | `<T extends object>` |
| `unknown` | Safe top type; must be narrowed |
| `never` | Bottom type (unreachable) |
| `any` | Opt out of checking — **avoid** |
| Type guards | `x is Foo` |
| Discriminated unions | Shared literal tag drives narrowing |
| `enum` | Named constants (prefer `as const` objects) |
| Decorators | Metadata annotations (Stage 3) |
| `strict: true` | The setting that makes it all worthwhile |
| `noUncheckedIndexedAccess` | Makes indexed access return `T \| undefined` |

```ts
// Discriminated union — the single most valuable TS pattern
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: Error };

function unwrap<T>(r: Result<T>): T {
  if (r.ok) return r.value;        // narrowed to the success variant
  throw r.error;
}

// satisfies: validate without losing literal inference
const config = { port: 3000, host: "0.0.0.0" } satisfies Record<string, string | number>;
config.port;                       // still typed as number, not string | number

// Generics with a constraint
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map(i => i[key]);
}
```

**Practical settings:** enable `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, and `noImplicitOverride`. Type-check in CI with `tsc --noEmit` even if a bundler strips types, since bundlers do not type-check.

## Appendix E: Quick Reference Card

```js
// Equality & existence
x === null || x === undefined    →  x == null
x ??= default                    // only if nullish

// Safe access
user?.address?.city              // no throw
obj?.[dynamicKey]
fn?.()

// Clone
const shallow = { ...obj };
const deep = structuredClone(obj);

// Deduplicate
const uniq = [...new Set(arr)];

// Group
const byType = Object.groupBy(items, i => i.type);          // ES2024

// Sort stably by a key
const sorted = [...rows].toSorted((a, b) =>
  a.name.localeCompare(b.name, undefined, { numeric: true }));

// Parallel with a concurrency cap
const results = await Promise.all(ids.map(id => fetch(id)));

// Timeout any promise
await Promise.race([work(), new Promise((_, r) =>
  setTimeout(() => r(new Error("timeout")), 5000))]);

// Debounce
const debounce = (fn, ms = 200) => {
  let t;
  return (...a) => { clearTimeout(t); t = setTimeout(() => fn(...a), ms); };
};

// Throttle to one call per interval (leading edge — calls inside the window are dropped)
const throttle = (fn, ms = 200) => {
  let last = 0;
  return (...a) => {
    const now = Date.now();
    if (now - last >= ms) { last = now; fn(...a); }
  };
};

// Retry with exponential backoff
const retry = async (fn, n = 3) => {
  for (let i = 0; ; i++) {
    try { return await fn(); }
    catch (e) { if (i >= n) throw e; await new Promise(r => setTimeout(r, 2 ** i * 100)); }
  }
};

// Chunk
const chunk = (arr, n) =>
  Array.from({ length: Math.ceil(arr.length / n) }, (_, i) =>
    arr.slice(i * n, i * n + n));
```
