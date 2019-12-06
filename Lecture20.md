# Nonblocking Synchronization

## Nonblocking

- algorithms that derive thread-safety from low-level atomic hardware primitives rather than locking
- immune to deadlock 
- starvation is possible

## Atomic Variables

- provide atomic read-modify-write operations without intrinsic locking
- cannot synchronize two atomic variables (just like `volatile`)
  - do not support atomic check-then-act sequences in general

## Compare And Swap (CAS)

- three operands (memory location `V`, expected value `A`, new value `B`)
- atomically updates `V` to `B`, but only if current value is `A`
  - if multiple threads try to update `V`, only one succeeds (others can try again)
 
```java
// not actually implemented this way!
public class SimulatedCAS {
    private int value;
    public synchronized int get() {
        return currValue;
    }
    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue= value;
        if (oldValue == expectedValue) value= newValue;    
        return oldValue;  
    }
    public synchronized boolean compareAndSet(int expectedValue, int newValue) {
        return (expectedValue == compareAndSwap(expectedValue, newValue));
    }
}
```

```java
// principle: implement compound atomic updates with a single update
public class CasNumberRange { 
    // IntPair is a pair of Integers 
    private final AtomicReference<IntPair> values = new AtomicReference<IntPair>(new IntPair(0, 0));
    public void setLower(int i) {    
        while(true) {
            // gets the current value atomically
            IntPair oldv = values.get(); 
            if (i > oldv.upper) throw new IllegalArgumentException();
            IntPair newv = new IntPair(i, oldv.upper);
            if (values.compareAndSet(oldv, newv)) return;
        }
    }
}
```

## Nonblocking Stack

```java
public class ConcurrentStack<E> {
    private static class Node<E> {
        public final E item;  
        public Node<E> next;  
        public Node(E item) { this.item= item; }
    }
    AtomicReference<Node<E>> top = new AtomicReference<Node<E>>();
    
    public void push(E item) {
        Node<E> newHead = new Node<E>(item);
        Node<E> oldHead;
        do {
            oldHead = top.get();
            newHead.next = oldHead;
        } while (!top.compareAndSet(oldHead, newHead));
    }
    
    public E pop() {
        Node<E> oldHead;  
        Node<E> newHead;
        do {
            oldHead = top.get();
            if (oldHead == null) 
                return null;
            newHead = oldHead.next;
        } while (!top.compareAndSet(oldHead, newHead));
        return oldHead.item;
    }
}
```

## Michael/Scott Approach

- threads should always be able to detect when the other is in progress
- thread B can wait for thread A before starting 
  - thread B may fail if thread A fails though
- M/S approach:
  - thread B arrives while operation is in progress for thread A
  - thread B finishes the update for thread A
  - thread A doesn't repeat work, if its work is done it skips it

## Nonblocking Queue

```java
public class ConcurrentQueue<E> {
    private static class Node<E> {
        final E item;
        final AtomicReference<Node<E>> next;
        public Node(E item, Node<E> next) { 
            this.item = item;
            this.next = new AtomicReference<Node<E>>(next);
        }
    }
    private final Node<E> dummy = new Node<E>(null, null);
    private final AtomicReference<Node<E>> head = new AtomicReference<Node<E>>(dummy);
    private final AtomicReference<Node<E>> tail = new AtomicReference<Node<E>>(dummy);
    
    public boolean put(Eitem) {
        Node<E> newNode = new Node<E>(item, null);
        while (true) {
            Node<E> currTail = tail.get();
            Node<E> tailNext = currTail.next.get();
            if (currTail == tail.get()) {                               // did tail change?
                if (tailNext != null) {                                 // queue in intermediate state, advance tail
                    tail.compareAndSet(currTail, tailNext);
                } else {                                                // in quiescent state, try inserting new node
                    if (curTail.next.compareAndSet(null, newNode)) {    // insertion succeeded, try advancing tail
                        tail.compareAndSet(curTail, newNode);           // will fail if tail already moved
                        return true;
                    }
                }
            }
        }
    }
```
