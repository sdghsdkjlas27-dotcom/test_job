---
name: java-pitfalls
description: Use when reading or reviewing Java code. Checklist of real Java bugs - NPE paths, equals/hashCode, generics, resource leaks, concurrency.
---

# Java pitfalls — scan each method against every line

## Null paths (the #1 Java bug)
- Every method returning an object: is there a null path (failed Map.get, Optional.get on empty, array [index] miss, iterator next on empty)?
- Every chained call `a.b().c()`: can `b()` return null? Autoboxing: `Integer x = map.get(k); int v = x;` → NPE on miss.
- Annotations lie: `@NonNull` on a method whose body CAN return null.

## equals / hashCode / comparison
- equals overridden without hashCode → broken HashMap/HashSet behavior (duplicate keys).
- equals with wrong signature `equals(Foo other)` instead of `equals(Object)` → never called by collections.
- `==` on boxed types / Strings that can come from different sources → identity comparison bug.
- compareTo inconsistent with equals in sorted collections.

## Resources & IO
- Stream/Connection/Reader opened without try-with-resources → leak on exception path.
- close() in wrong order; closing then using; flush missing before close on buffered writer.
- InputStream.read() returns -1 vs 0 confusion; read(byte[]) returns count, not fills.

## Generics & collections
- Raw types casting: `(List<String>)` unchecked — ClassCastException at a distance from the real cause.
- Arrays.copyOf with wrong length (off-by-one loses/duplicates elements).
- Modifying collection while iterating (ConcurrentModificationException) — even single-threaded in loops.
- subList views of a backing list mutated afterwards → undefined behavior.

## Numbers
- Integer overflow: int math on sizes/timestamps (use long); Math.abs(Integer.MIN_VALUE) is still negative.
- Division truncation toward zero for negatives (-7 / 2 == -3).
- BigDecimal.equals compares scale too (2.0 != 2.00) — use compareTo.
- Double math for money.

## Strings
- String.split regex metacharacters: split(".") returns empty array.
- String.format locale: %d with locale grouping, %s on null prints "null".
- Charset missing in new String(bytes) / getBytes() → platform-dependent.

## Concurrency
- Lazy init without synchronization: check-then-act race (`if (x == null) x = new ...`).
- SimpleDateFormat / Calendar shared across threads.
- Non-volatile mutable flag read by another thread.
- HashMap read while another thread writes (infinite loop pre-Java-8 / CEs after).

## Exceptions
- catch (Exception e) swallowing without logging → hides real failure.
- finally that returns/throws → overrides the original exception.
- Losing cause: new MyException(msg) without wrapping original e.
