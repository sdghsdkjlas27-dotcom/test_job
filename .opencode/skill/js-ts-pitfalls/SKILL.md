---
name: js-ts-pitfalls
description: Use when reading or reviewing JavaScript/TypeScript code. Checklist of real JS/TS bugs - async mistakes, coercion, null chains, array mutation, closure traps.
---

# JS/TS pitfalls — scan each function against every line

## Async (the #1 JS bug class)
- Missing `await`: `const d = fetchData(); d.field` → undefined (promise has no field). Async function without await inside — fire-and-forget errors vanish.
- Sequential awaits in loops that should be parallel (perf bug) — or `forEach` with async callback that NEVER awaits (forEach ignores promises → race/ordering bug).
- try/catch around `await`-less call does not catch async rejection.
- Promise.all fails-fast: one reject loses others' results — should be allSettled when partial success is OK.
- Async callbacks after timeout/teardown use closed resources.

## Null/undefined chains
- `a?.b.c` — the `?.` guards only one hop; `a?.b` can still be undefined and `.c` crashes.
- Optional chaining + default: `x ?? y` vs `x || y` — `||` treats 0/""/false as missing (legit values replaced).
- Array destructuring `const [first] = arr` → undefined on empty, then `first.foo` crashes.
- `parseInt(x)` without radix; `parseInt(undefined)` → NaN silently propagates.

## Coercion & equality
- `==` with null/undefined/0/""/false cross-type surprises — must be `===` everywhere (grep for `==` that is not `===`).
- `+` with one string → concatenation: `"total: " + 1 + 2` → "total: 12".
- `NaN` comparisons always false: `x === NaN` never true — Number.isNaN needed.
- `typeof null === "object"`; `Array.isArray` required not `typeof x === "array"`.

## Arrays & objects
- Mutation of shared state: sort/reverse/splice on props/const arrays (sort mutates in place! `[...arr].sort()` required).
- `splice(i, 0, x)` vs `slice` confusion; splice returns removed items not the array.
- Loop with closures over `var` (shared binding) — let/const fix; async in loops captures stale values.
- Object spread is SHALLOW: `{...a, nested}` — nested object still shared reference; mutating it corrupts the original.
- JSON.parse without try/catch on external input; JSON.stringify drops undefined fields (API contract change).

## Types (TS)
- Non-null assertion `x!` and `as X` casts — each one is an unverified claim; check the runtime can actually deliver.
- `strict: false` / `any` params: real types mismatch at runtime (string vs number from API).
- Enum reverse mapping surprises; `keyof` on optional fields lies.

## Misc
- `Date` months are 0-based: `new Date(2024, 12, 1)` is next January.
- `setTimeout(fn, 0)` ordering vs promise microtasks.
- Regex without global flag reused with `.exec`/`test` (lastIndex state); WITH global flag `.test` alternates results.
- `for...in` on arrays iterates string keys incl. prototype junk — `for...of` needed.
