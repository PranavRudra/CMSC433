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

- loop iterations are split up among threads with barrier is at end to synchronize

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

