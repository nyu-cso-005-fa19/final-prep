## Multithreading

1. `00`, `01`, `02`

2. only `01`

3. no output, `0`, `00`, `01`, and `02`.


## Locking

1. The implementation satisfies mutual exclusion. For a proof by
   contradiction, suppose it did not. That is, it is possible for two
   threads A and B to enter their critical sections at the same time.
   Suppose without loss of generality that thread A entered the
   critical section first, i.e., exited the outer loop in
   `flaky_lock`. At the point when `A` exits the loop we must have
   `lock->busy == 1`. Moreover, for `A` to exit the loop we further
   must have `lock->turn == A` and this must have been true
   continuously since `A` set `lock->turn` to `A` in the inner
   loop. Thus, `B` cannot yet have set `lock->turn` to `B` in the
   meantime and his hence not yet past the inner loop, or it is past
   the inner loop but the outer loop condition `lock->turn != B` now
   evaluates to `1` for `B` and `B` has to retry. In any case, `B`
   will not be able to proceed to the critical section, yielding a
   contradiction.
   
2. The algorithm is not deadlock free. Consider an execution where two
   threads A and B call `flaky_lock`. The following sequence of
   events leads to a deadlock:
   
   a. Thread A writes A into `lock->turn`
   b. Thread A reads `0` from `lock->busy` and exits the inner loop
   c. Thread B writes B into `lock->turn`
   d. Thread A write `1` into `lock->busy` (B can't leave inner loop)
   e. Thread A reads `B` from `lock->turn` and hence remains in the
   outer loop
   f. Thread A reenters the inner loop and is stuck as well.
   
   To make it deadlock free the outer loop has to set the `busy` flag
   back to `0` whenever it retries. Here is a correct implementation:
   
   ```c
   void flaky_lock(flaky_lock_t *lock) {
     pid_t me = gettid();
     while (1) {
       do {
         lock->turn = me;
       } while (lock->busy);
       lock->busy = 1;
       if (lock->turn == me) break;
       lock->busy = 0;
     }
   }
   ```
   
   
3. Since the implementation is not deadlock free it is also not
   starvation-free.
   
## Linearizability

Execution 1:

```
Thread A: -------[ read(r)/1 ]---------------------->
Thread B: ----[ write(r,1)    ]---[ read(r)/2 ]----->
Thread C: ------ [ write(r,2) ]--------------------->
```

Linearizable: `write(r, 1); read(r)/1; write(r, 2); read(r)/2;`

Execution 2:

```
Thread A: ---------[ read(r)/1 ]-------------------->
Thread B: ------[ write(r,1)      ]----[ read(r)/1]->
Thread C: -----------[ write(r,2) ]----------------->
```

Linearizable: `write(r, 2); write(r, 1); read(r)/1; read(r)/1;`


Execution 3:

```
Thread A: ----[ read(r)/1 ]------------------------->
Thread B: ------[ write(r,1) ]---------[ read(r)/1]->
Thread C: ---------------[ write(r,2) ]------------->
```

Linearizable: `read(r, 1); write(r, 2); write(r, 1); read(r)/1;`


Execution 4:

```
Thread A: ----[ read(r)/1 ]------------------------->
Thread B: ------[ write(r,1) ]-------[ read(r)/1]--->
Thread C: ---------------[ write(r,2) ]------------->
```

Linearizable: `write(r, 2); write(r, 1); read(r)/1; read(r)/1;`


Execution 5:

```
Thread A: [ read(r)/1 ]----------------------------->
Thread B: -[ write(r,1) ]--------------[ read(r)/1]->
Thread C: ---------------[ write(r,2) ]------------->
```

Not linearizable because `read(r)/1` must take effect after
`write(r,2)` which in turn must take effect after `write(r,1)`.
