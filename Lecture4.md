# Synchronization

## Goals

- atomicity: locking to obtain mutual exclusion
- visibility: changes made in one thread must be visible to other threads
- ordering: ensure that we aren't surprised by order in which statements are executed

## Reordering

- Java permits reordering of statements by compiler (sequential order preserved however)

## Visibility

```java
public class NoVisibility {

    private static boolean ready;
    private static int number;

    private static class ReaderThread extends Thread{

        public void run() {

            while (!ready){                     // may never run because ready in main memory is still false
                Thread.yield();
            }
            System.out.println(number);         // output could be 42, nothing, 0, or block forever
        }
    }

    public static void main(String[] args) {

        new ReaderThread().start();
        number = 42;                            // could update thread-local cache instead of main memory
        ready = true;                           // could update thread-local cache instead of main memory
    }                                           // since ReaderThread may not see updates from main thread, visibility isn't guaranteed
}
```

- all writes from thread holding lock M guaranteed to be visible to next thread that grabs M

## Happens before

- if A happens before B, memory effects of A are visible to thread performing B before B's done
- rules
  - Transitivity rule: if A happens before B and B happens before C then A happens before C
  - Program order rule: each statement in thread happens before subsequent statements in that thread
  - Monitor lock rule: an unlock on a monitor happens before every subsequent lock on same monitor
  - Volatile variable rule: a write to a volatile variable happens before every read from that variable
  - Thread start rule: a call to `Thread.start()` happens before every action in the started thread
  - Thread termination rule: any action in a thread happens before another thread determines thread has terminated
  - Interruption rule: thread calling an interrupt on another thread happens before interrupted thread detects interrupt

## Volatility

- JVM ensures that all threads read the latest value for volatile field
- writing to volatile field pushes change directly to main memory
- problems
  - cannot keep two volatile fields in sync
  - incrementing a volatile field is NOT atomic
  - writes to volatile field that depend on previous value of field don't work
  - volatile reference to an object is not the same as having a volatile object

```java
private int a = 1;
private int b = 2;
private int c = 3;
private volatile boolean hasValue = true;

a = 5;                  // write can be reordered
b = 6;                  // write can be reordered
c = 7;                  // write can be reordered
hasValue = true;        // this must occur last since hasValue is volatile
```
