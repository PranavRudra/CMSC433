# Java Memory Model

## Out-Of-Thin-Air Safety

- a read of a variable yields a value written by some thread (no garbage values like in C/C++)

## Order

- program order: events occur in the order specified in the source code
  - guaranteed to be well-formed
    - begin is first event
    - end is the last event
- as-if-serial semantics: events can be reordered but results must be consistent with program order

## JMM Goals

- sequential consistency for multi-threaded programs with no data races
  - consider threads t1, t2, t3 with program orders S1, S2, S3
  - multithreaded execution is sequentially consistent if there is an interleaving S
    - S must preserve the order of events in S1, S2, S3 in an as-if-serial way

## Roach Motel Constraint

- reads/writes can be reordered within a synchronized (lock/unlock) block 
- reads/writes within a synchronized block cannot be moved outside of it