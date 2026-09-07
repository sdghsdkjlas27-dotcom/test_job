---
name: ruby-pitfalls
description: Use when reading or reviewing Ruby code. Checklist of real Ruby bugs to scan every function against - nil chains, bang methods, mutable defaults, encoding, swallowed exceptions.
---

# Ruby pitfalls — scan each function against every line

## Nil & NoMethodError
- `x.a.b.c` chains: can ANY link return nil? Hash lookup `h[k]` → nil on miss. `arr.find {}` → nil on miss. `str.match` → nil on no match. `@ivar` never set → nil.
- `&.` guards one link only: `a&.b.c` still crashes if `a.b` returns nil.
- `config[:key]` on Hash with Symbol keys but caller passes String key (or vice versa) → silently nil.

## Bang methods return nil
- `sort!`, `uniq!`, `map!`, `compact!`, `flatten!`, `merge!` return **nil when nothing changed**.
- Pattern: `arr = arr.uniq!` or `x = h.merge!(other)` → x becomes nil sometimes. Also `return if (arr.uniq!)` logic is inverted.
- Chained: `list.sort!.first` → NoMethodError on nil when already sorted.

## Mutability & shared state
- Default args evaluated per call BUT mutated args leak: `def f(x, out = [])` is safe, but class-level `@@var` or `CONST << x` accumulates across calls.
- `+=` on strings in loops (O(n²) + encoding issues); frozen string literals make `<<` raise FrozenError.
- Methods that mutate their receiver used on shared/temporary objects.

## Numbers & coercion
- Integer division: `5 / 2 == 2` — money/percentage math losing cents.
- `to_i` on nil crashes; `Integer(x)` raises vs `x.to_i` silently 0 — wrong choice hides bad input.
- Float equality: `0.1 + 0.2 == 0.3` is false. Comparisons with `==` on floats after math.
- `rand(n)` vs `rand(a..b)` off-by-one; `SecureRandom` needed for security.

## Strings & encoding
- Encoding mix: `UTF-8` string + `ASCII-8BIT` (binary/HTTP body) → ArgumentError on concat/regexp.
- `str[-1]` returns char not code; multibyte slicing `str[0,3]` cuts mid-character.
- `gsub` with special chars in replacement (`\1` interpreted); `Regexp.new(user_input)` → regexp injection/crash.

## Exceptions
- `rescue => e` catching StandardError where the raise is outside the begin block (rescue modifier applies to whole expression — subtle).
- Empty rescue / rescue that returns nil: converts crashes into silent nil-poisoning downstream.
- `raise` inside `ensure` swallows the original exception.
- `retry` without counter → infinite loop on persistent error.

## Blocks, procs, iteration
- `return` inside a block returns from the ENCLOSING METHOD (not the block) — surprising early exits.
- Closure captures the VARIABLE, not value: loop variable reused across blocks created in a loop.
- `.each` vs `.map` misuse: `.each` returns the original array — chained `.each {}.first` works but `.map {}.map` double-transforms.
- `delete_if` / `reject!` while iterating same collection → skipped elements.
- Hash default block `Hash.new { [] }` without `<<` reassignment: `h[k] << v` silently discards.

## Class & module
- Method redefined twice in same file (second wins silently) — grep `def same_name`.
- `attr_accessor` on class-level config that multiple threads share.
- `method_missing` without `respond_to_missing?` → broken `respond_to?`.
- Monkeypatch of stdlib/gem class affecting the whole process (grep `class String`, `module Kernel`, `class Hash`).
