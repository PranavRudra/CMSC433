# Testing

## Types

- functional: does the system deliver the required features
- performance: does system execute in a timely manner
- stress: how does system respond to unexpected operating conditions

## Software Tests

- unit: checks an individual code unit (e.g. classes)
- integration: checks collections of units (e.g. components)
- validation: checks entire software system (e.g. end-to-end tests)
- regression: ensures that new changes don't break existing code (e.g. continuous deployment)

## Coverage

- function coverage: has each function been called
- statement coverage: has each line been executed
- branch coverage: has each if-else block been executed
- conditon coverage: has each boolean sub-expression been executed

## Interleavings

```java
private int numRuns = 10;
public void test() throws InterruptedException {
    for (int i = 0; i < numRuns; i++) {
        IncThread.resetShared();
        Thread t1 = new Thread (new IncThread("t1"));
        Thread t2 = new Thread (new IncThread("t2"));
        passes = (IncThread.getShared() == 2) ? (passes + 1) : passes;
    }
    // if code has no race conditions, passes should equal numRuns (for a very large numRuns)
}
```
