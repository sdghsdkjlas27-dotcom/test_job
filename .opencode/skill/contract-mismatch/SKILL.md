---
name: contract-mismatch
description: Use during Phase 2 cross-file checks. How to find bugs at call boundaries between modules - argument types, nil/null propagation, return shapes, error contracts.
---

# Cross-file contract mismatch hunting

A contract mismatch is when file A calls file B with assumptions B does not guarantee.
These are the highest-value bugs. Check EVERY call boundary you see.

## The 7 contracts to verify at each boundary

1. **Type contract**: caller passes String, callee calls `.map` / `.length` (Ruby String has no `.map`) → crash.
   - Ruby: Integer passed where Array expected (`arr.first` on Integer → NoMethodError)
   - Java: Object passed, cast without instanceof
   - JS: number passed, callee does `.trim()`
2. **Nil/null contract**: callee can return nil/null (failed lookup, `find` miss, `[]` on missing key) and caller chains a method on it (`user.name` on nil → NoMethodError / NPE).
   - Trace: does the callee have a path where the lookup fails? Hash `[]` returns nil by default. `find` returns nil on miss. DB/API lookups fail.
3. **Shape contract**: callee returns Hash `{a: 1}` but caller does `result[0]` (array indexing on hash → nil). Or returns a wrapped object (`resp.data.items`) but caller reads `resp.items`.
4. **Empty-collection contract**: callee returns `[]` on no results, caller does `.first.something` → nil crash. Or caller does `result - 1` on empty-count.
5. **Error contract**: callee raises on bad input; caller does not rescue → crash path. OR callee returns error-code/sentinel (`nil`, `false`, `""`) and caller checks `raise` — or vice versa. Mixed conventions at one boundary = bug.
6. **Mutation contract**: does callee mutate the argument it receives? If two callers share an array/hash and one path sorts/`uniq!`s it in place, the other caller gets corrupted data. Bang methods (`sort!`, `uniq!`, `map!`, `merge!`) return nil when no change — chaining on their result crashes.
7. **Unit/range contract**: seconds vs milliseconds, bytes vs KB, 0-based vs 1-based, inclusive vs exclusive range (`..` vs `...`), cents vs dollars. Grep for numbers multiplied/divided at boundaries.

## How to trace efficiently
1. Grep the symbol: `grep "def suspicious_method"` to find its definition.
2. Read the FULL definition — every return path, not just the happy path.
3. Grep call sites: `grep "suspicious_method("` — read 5 lines around each.
4. Ask: for EVERY return path of callee, does EVERY caller survive the value?
5. Pay double attention to optional/keyword args with defaults — defaults often have the wrong type (`def f(opt: nil)` then `opt.size`).

## Red flags worth grepping directly
- `.first.` / `.last.` / `.dig(...).` — chained on possibly-empty/nil
- `rescue` followed by empty body or `nil` — swallowed errors hide real bugs downstream
- `to_i` / `to_s` / `to_sym` used to "fix" type mismatches — look for the original mismatch
- Constants/config read in one file and redefined in another (grep the constant name)
