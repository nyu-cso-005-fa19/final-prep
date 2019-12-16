# Final Exam Preparation

Below are some additional practice problems related to concurrency
that you can use for your exam preparation.


## Multithreading

Consider the following C program

```c
#include <pthread.h>
#include <stdio.h>

int y = 0;

void* foo(void* x0) {
  int* x = (int*) x0;
  printf("%d", y);
  y = *x;
  return NULL;
}

int main(void) {
  pthread_t tid1, tid2;
  int x1 = 1;
  int x2 = 2;
  pthread_create(&tid1, NULL, &foo, &x1);
  pthread_create(&tid2, NULL, &foo, &x2);
  pthread_join(tid1, NULL);
  pthread_join(tid2, NULL);
  return 0;
}
```

1. What are the potential outputs of this program?

2. What are the potential outputs if the lines

   ```c
   pthread_create(&tid1, NULL, &foo, &x2);
   pthread_join(tid1, NULL);
   ```
   
   are swapped?
   
3. What are the potential outputs if the lines

   ```c
   pthread_join(tid1, NULL);
   pthread_join(tid2, NULL);
   ```
   
   are omitted?

Assume that both calls to `pthread_create` return 0.

## Locking

Programmers at the Flaky Computer Cooperation designed the following
lock implementation to achieve mutual exclusion between an arbitrary
number of threads. 

```c
typedef struct {
  pid_t turn;
  int busy;
} flaky_lock_t;

void flaky_lock(flaky_lock_t *lock) {
  pid_t me = gettid();
  do {
    do {
      lock->turn = me;
    } while (lock->busy);
    lock->busy = 1;
  } while (lock->turn != me);
}

void flaky_unlock(flaky_lock_t *lock) {
  lock->busy = 0;
}
```

* Does this lock implementation satisfy mutual exclusion?
* Is this lock implementation deadlock-free?
* Is this lock implementation starvation-free?

If your answer is 'No' to any of the questions, describe an execution
that demonstrates why the property does not hold.

You may assume that all assignments and accesses to shared memory
locations execute atomically. Further assume that `lock->busy` is
initialized to `0` before any call to `flaky_lock` is executed.


## Linearizability

Consider a data structure implementing a read/write register for
atomically reading and writing integer values. The data structure
provides the following methods:

```c
struct reg_t;

int read(struct reg_t *reg);

void write(struct reg_t *reg, int new_v);
```

Here, `read` takes a pointer to a register `reg` and returns the current
integer value stored in `reg`. The method `write` takes a pointer
`reg` and replaces the current value stored in `reg` with the new
value `new_v`.

The sequential correctness specification of the register states that
every call to `read` must return the value written by the most recent
call to `write`. If there is no preceding call to `write`, then the
value returned by `read` is undefined.

For each of the following concurrent executions, decide whether it is
linearizable:

Execution 1:

```
Thread A: -------[ read(r)/1 ]---------------------->
Thread B: ----[ write(r,1)    ]---[ read(r)/2 ]----->
Thread C: ------ [ write(r,2) ]--------------------->
```

Execution 2:

```
Thread A: ---------[ read(r)/1 ]-------------------->
Thread B: ------[ write(r,1)      ]----[ read(r)/1]->
Thread C: -----------[ write(r,2) ]----------------->
```

Execution 3:

```
Thread A: ----[ read(r)/1 ]------------------------->
Thread B: ------[ write(r,1) ]---------[ read(r)/1]->
Thread C: ---------------[ write(r,2) ]------------->
```

Execution 4:

```
Thread A: ----[ read(r)/1 ]------------------------->
Thread B: ------[ write(r,1) ]-------[ read(r)/1]--->
Thread C: ---------------[ write(r,2) ]------------->
```

Execution 5:

```
Thread A: [ read(r)/1 ]----------------------------->
Thread B: -[ write(r,1) ]--------------[ read(r)/1]->
Thread C: ---------------[ write(r,2) ]------------->
```

If an execution is linearizable, show its linearization. If it is not
linearizable, explain why.
