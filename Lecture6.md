# Sharing Objects

## Publishing vs. Escape

- publishing: making an object available outside the scope in which it was created
- escape: unintended or poorly considered publishing (major source of errors)

## Errors

- publishing an inner object (exposes outer object via `this$0` field)
- allowing `this`to escape from the constructor
- starting a thread in the constructor

```java
public class MutablePoint {
    private Point p1;
    private Point p2;
    public Point getP1() {
        return p1;              // caller of getP1() can now screw up the p1 object
    }
}
```

```java
public class Unsafe {
    public static Set<Unsafe> set = new HashSet<>();
    public Unsafe() {
        set.add(this);          // publishes this since set is globally accessible
    }
}
```

- use static factory methods to perform necessary work without unsafely publishing object

## Thread Confinement

- stack-confined object:
  - object is created in a thread
  - object is assigned to a local variable
  - object is never published outside the thread

## ThreadLocal

- `ThreadLocal` object is a thread-local container for other object(s)
- each thread accessing `ThreadLocal` is given its own variable to the contained object
- however, object(s) pointed to by those variables can still be shared between different threads

```java
ThreadLocal<ArrayList<Long>> workerIds = new ThreadLocal<ArrayList<Long>>(){
    @Override
    protected ArrayList<Long> initialValue() {
        return new ArrayList<Long> ();
    }
    // anonymous class allows us subclasses ThreadLocal implicitly
}
```

```java
class MyThread extends Thread {
    private static ThreadLocal<ArrayList<Integer>> container = new ThreadLocal<>() {
        @Override
        protected ArrayList<Integer> initialValue() {
          return new ArrayList<Integer>();
        }
    };
    private void populateList(ArrayList<Integer> list) {
        Random r = new Random();
        for (int i = 1; i < 10; i++) {
            list.add(r.nextInt(1000));
        }
    }
    // Each thread accesses the ArrayList<Integer> in the ThreadLocal container through its own localList variable
    public void run() {
        ArrayList<Integer> localList = container.get();
        populateList(localList);
    }
}
```

## Immutability

- immutability conditions:
  - all of the object's fields are `final`
  - object's state never changes after construction
  - object is properly constructed (`this` does not escape during construction)

```java
public class ImproperImmutableABimplements... {
    public final int a;
    public final int b;
    ImproperImmutableAB(int a, int b) {
        this.a = a;
        // if this constructor prints the value of b, it will see 0 instead of the b passed in
        new ThreadABPrinter(this).start();
        try { Thread.sleep(20); } catch ... {}
        this.b = b;
    }
}
```

- initialization safety: all writes to `final` fields in immutable objects are visible immediately

## Misc

```java
public class Holder {
    private int n;
    public Holder(int n) {
        this.n = n;  
    }
    public void assertSanity() {
        if (n != n) throw new AssertionError(“n != n !!");
    }
}
// suppose main creates a global Holder variable
// main then starts a thread t that calls h.assertSanity()
// main then instantiates h = new Holder(42)

// when h.assertSanity() happened n = 0 was read
// however when not equal comparison happens n = 42
// if instantiation is fast enough, the comparison 0 != 42 will occur
```
