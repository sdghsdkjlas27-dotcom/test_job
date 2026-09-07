---
name: go-pitfalls
description: Use when reading or reviewing Go code. Checklist of real Go bugs - nil interfaces, loop variable capture, slice aliasing, error shadowing, goroutine leaks.
---

# Go pitfalls — scan each function against every line

## nil interface vs typed-nil (the #1 Go trap)
- `var p *T = nil; var i error = p; i != nil` is TRUE — typed nil inside interface is not nil.
- Functions returning `(*T, error)` where err path returns `(nil, someErr)` but caller checks only `if result != nil` → treats error as success.
- Interface nil check after assignment from a nil concrete pointer never works — check before assigning.

## Loop variable capture
- Pre-1.22 semantics: `for _, x := range items { go func() { use(x) }() }` — all goroutines see last x. Same in closures appended in loops.
- Taking `&x` / `&v` of loop var → address of a reused slot.

## Slices
- Slice of a slice shares the backing array: append to subslice can corrupt the original; modify via subslice modifies parent.
- `append` result not assigned back: `append(s, x)` without `s =` — change lost (sometimes works when cap allows — HEISENBUG).
- Slice aliasing after `s[:0]` reuse tricks — data races with retained references.
- `len`/`cap` confusion; `s[low:high]` with high > len panics (ok only with 3-index slicing up to cap).

## Errors
- `err` shadowed inside if-statement: `if err := f(); err != nil` hides outer err reuse — check declared-vs-assigned.
- Wrapping loses the sentinel: `%v` instead of `%w` breaks errors.Is/As downstream.
- Ignored error: `_ = f()` or bare call — every `_ =` on error path is a candidate bug.
- Checking `err != nil` AFTER using the value (order inverted).

## Goroutines & channels
- Goroutine leak: goroutine blocks forever on channel send/recv that nobody serves — count sends vs receives vs close.
- Close of closed channel / send on closed channel → panic; who closes, when, exactly once?
- WaitGroup.Add inside the goroutine instead of before launching — race with Wait.
- Unbuffered channel assumed buffered — deadlock when single goroutine does both send and receive.

## Maps & values
- Map iteration order random — code depending on order breaks intermittently.
- Map zero value is nil: write to nil map panics; read is fine (zero value).
- Struct copy with mutex inside — lock the copy, not the original. Range over slice of structs copies each element; method with pointer receiver mutates the copy's fields invisibly.

## Misc
- `defer` in loop accumulates until function end (file handles exhausted).
- Shadowed stdlib names (a local `recover`/`copy`).
- Time.Duration is int64 nanoseconds — multiplying by "seconds" number without unit.
- Integer division + int overflow on 32-bit; uint subtraction underflow wraps to huge.
