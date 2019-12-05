# OpenMP

## Functions

- parallelize work across threads on shared-memory systems

## Directives

- `parallel`: parallelize this block
- `barrier`: synchronize this block
- `for`: split this loop for me (fine-grained)
- `sections`: divide these sections among threads (coarse-grained)
- `shared/private`: variables should be marked as shared or private

## parallel for

- loop iterations are split up among threads with implicit barrier at end to synchronize

```C
ifirst = 10;
#pragma omp parallel for
for(i = 1; i <= imax; i++) {
    // problem since i2 is shared
    // all variables except loop 
    // index are shared by default with parallel for
    i2 = 2*i;
    j[i] = ifirst + i2;
}

ifirst= 10;
#pragma omp parallel for private(i2)
for(i = 1; i <= imax; i++) {
    // no problem since i2 private now
    i2 = 2*i;
    j[i] = ifirst + i2;
}
```

```C
ifirst = 10;
#pragma omp parallel for   \
    default(none)          \
    shared(ifirst,imax,j)  \
    private(i2)
for(i = 0; i < imax; i++){
    i2 = 2*i;
    j[i] = ifirst + i2;
}
// default(none) clears defaults and allows you to specify custom rules
```

```C
iper = 0;
#pragma omp parallel for private(iper)
// problem because iper being private won't
// allow its initial value to be carried into
// the loop
for(i = 0; i < imax; i++){
    iper = iper + 1;
    j[i] = iper;
}

iper = 0;
#pragma omp parallel for \
    firstprivate(iper)
// no problem since firstprivate 
// will copy initial value of 
// iper from master thread
// into each worker while "privatizing" it
for(i = 0; i < imax; i++){
    iper = iper + 1;
    j[i] = iper;
}
```

```C
#pragma omp parallel for \
    lastprivate(i)
for(i = 0; i < maxi - 1; i++){
    a[i] = b[i];
}
a(i) = b(1);
// saves value corresponding to 
// last loop index (in the 
// serial sense) in i
```

## reduction

```C
sum1 = 0.0;
// problem since there may be concurrent writes to sum1
for(i = 0; i < imaxi; i++) {
    sum1 = sum1 + a[i];
}

sum1 = 0;
#pragma omp parallel for \
    reduction(+:sum1)
// no problem since each thread will now perform its 
// own reduction (aggregating partial sums) and then
// perform an overall reduction into sum1 after
// all threads complete loop
for(i = 0; i < imaxi; i++){
    sum1 = sum1 + a[i];
}
```

## enhancements

```C
#pragma omp parallel
#pragma omp for
for(i = 0; i < maxi; i++){
    a[i] = b[i];
}
#pragma omp for
for(i = 0; i < maxi; i++){
    c[i] = a[2];
}
#pragma omp end parallel
// faster than using two parallel for directives (since parallel has overhead)
// #pragma omp parallel for is equivalent to:
// #pragma omp parallel
// #pragma omp for
```

## barrier

- synchronizes threads (execution won't proceed until all threads hit the barrier)

## master

- master directive forces execution only on master thread (other threads will skip over that code)

```C
#pragma omp parallel private(myid, istart, iend)
myrange(myid, istart, iend);
for(i = istart; i <= iend; i++){
    a[i] = a[i] – b[i];
}
// all threads must hit this line before continuing exeuction
#pragma omp barrier
// this code will only execute on master thread
#pragma omp master
fwrite(fid, sizeof(float),iend - istart + 1,a);
#pragma omp end master
// this will be done in parallel
do_work(istart, iend);
#pragma ompend parallel 
```

## single

- force code to only be executed by a single thread

```C
#pragma omp parallel private(myid, istart, iend)
myrange(myid, istart, iend);
for(i = istart; i <= iend; i++){
    a[i] = a[i] – b[i];
}
// all threads must hit this line before continuing exeuction
#pragma omp barrier
// this code will only execute on one thread
#pragma omp single
fwrite(fid, sizeof(float),iend - istart + 1,a);
#pragma omp end single
// unlike end master, ^ has an implicit barrier associated with it
do_work(istart, iend);
// ^ will be done in parallel
#pragma ompend parallel 
```

## critical

- every thread must execute this code
- execution can be in any order (i.e. no thread ordering)
- code cannot be executed simultaneously by multiple threads

```C
#pragma omp parallel private(myid, istart, iend)
myrange(myid, istart, iend);
for(i = istart; i <= iend; i++) {
    a[i] = a[i] – b[i];
}
#pragma omp critical 
mycrit(myid, a);
#pragma omp end critical
// no implicit barrier associated with ^
do_work(istart, iend);
#pragma omp end parallel
```

## ordered

```C
#pragma omp parallel for
for(i = 0; i < nproc; i++) {
    do_lots_of_work(result[i]);
    #pragma omp ordered
    fprintf(fid,”%d%f\n,”i,result[i]”);
    #pragma omp end ordered
    // ^ will force serial order within a parallel block
}
```

## sections

- force designated sections within a parallel region to be executed by different threads (coarse-grained)

```C
#pragma omp parallel
#pragma omp sections
// each section block will be executed by a different thread
#pragma omp section
init_field(a);
#pragma omp section
check_grid(x);
#pragma omp end sections
#pragma omp end parallel
```

## atomic

```C
for(i = 0; i < 10; i++) {
    #pragma omp atomic
    x[j[i]] = x[j[i]] + 1.0;
}
```

## flush

- different threads can have different values for shared variable 
- flush ensures calling thread has a consistent view of memory

```C
int isync[num_threads];
#pragma omp parallel default(private) shared(isync)
iam = omp_get_thread_num();
isync[iam] = 0;
#pragma omp barrier
work();
isync[iam] = 1;
// let other threads know I finished work
#pragma omp flush(isync)
while(isync[neigh] == 0)
    #pragma omp flush(isync)
#pragma omp end parallel
```