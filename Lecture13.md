# Streams

## Core

- computed on demand (intermediate results are not stored)
- not evaluated until terminal operation (lazy evaluation)

## Operations

- `limit(n)`: returns a stream that is no longer than n elements (nonterminal)
- `skip(n)`: returns a stream with the first n elements skipped (nonterminal)
- `findFirst()`, `findAny()`: returns an `Optional<T>` object (nonterminal)
- `anyMatch(predicate)`, `allMatch(predicate)`, `noneMatch(predicate)`: return a boolean (nonterminal)

## Special Streams

- `mapToInt(predicate)`, `mapToDouble(predicate)`, `mapToLong(predicate)`: return a specialized `Stream<T>` (nonterminal)
- `range(end)`, `rangeClosed(start, end)`: return a stream of all elements in the range (nonterminal)
- `Stream.iterate(start, predicate)`, `Stream.generate()`: return infinite streams 
  - possible because of lazy evaluation
  
## Nonterminal Operations

- `filter(predicate)`
- `map(function)`
- `flatMap(function)`
- `sorted(comparator)`
- `peek(consumer)`
- `distinct()`
- `limit(long n)`
- `skip(long n)`

## Terminal Operations

- 
