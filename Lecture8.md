# Concurrent Collections

## Methods

- manually allow concurrency with locks, `wait()/notify()/notifyAll()`
- use inbuilt `Collections` factory methods for instance confinement
- use concurrent collections from `java.util.concurrent`

```java
// list will be thread-safe because of inbuilt synchronized wrapper code
List<Integer> list = Collections.synchronizedList(new ArrayList<Integer>());
```

```java
// instance confinement b/c accesses are thread-safe while List<T> isn't
class SynchronizedList<T> implements List<T> {

    // list passed in is the backing list
    final List<T> list;

    SynchronizedList<T>(List<T> list) {
        this.list = list;
    }

    public synchronized int size () {
        return list.size();
    }

    // static factory method
    public static <T> List<T> synchronizedList(List<T> list) {
        return new SynchronizedList<T> (list);
    }
}
```

```java
// compound actions require "client-side" locking on the whole SynchronizedList to be thread-safe
public static Object getLast (List<Object> l) {
    synchronized (l) {
        int lastIndex = l.size() - 1;
        return (l.get(lastIndex));
    }
}
```

## java.util.concurrent

- `ConcurrentHashMap`: uses lock striping to prevent need to lock on the whole data structure
- `CopyOnWriteArrayList`:
  - no locking needed to read a list
  - list modification creates local copy of list
  - modified list is republished once update is complete (no `ConcurrentModificationException`)
- `SynchronizedList` is better for frequent updates while `CopyOnWriteArrayList` better if mostly reading

## Producer consumer pattern

- producers are populating a queue with elements
- consumers are taking out the elements in queue
