# Parallelization

## Loops

- method #1: iterations must be independent
- method #2: chunk iterations into groups

```java
// method #1

// sequential
for (Element e : collection)
    process(e);
    
// parallelized
for (Element e : collection) {
    exec.execute (new Runnable() {
        public void() run {
            process(e);
        }
    });
}
```

```java
# method #2

// sequential
sum = 0;
for (int i=0; i < a.length; i++)
    sum += a[i];
    
// parallelized
int sum[] = new int[NUMTASKS];                          // ensure initialization to 0
int size = a.length / NUMTASKS;
for (int i=0; i < NUMTASKS; i++) {
    exec.execute (new Runnable() {
        public void run() {
            for (j = i*size; j < (i+1)*size; j++)
                sum[i] += a[j];
            });
    }
}
```
