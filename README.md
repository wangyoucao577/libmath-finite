# libmath-finite     
**libmath-finite** intends to solve the `xx_finite` symbols missed problem due to `glibc` upgrade. It generally happens when distribute your pre-built binray(`.a/.so/executable`) from `glibc<2.31` build enviornment to `glibc>=2.31` systems, such as `Debian 10 -> Debian 11`, or `Ubuntu18.04 -> Ubuntu20.04`. [Math changes in glibc 2.31](https://sourceware.org/pipermail/libc-alpha/2020-May/113773.html) also discusses the problem.          
It's a very simple library, which only wraps functions of the standard math library `libm`. **Many thanks to **Étienne** who proposes the idea in [Link errors with -ffast-math (-ffinite-math-only) and glibc 2.31](https://stackoverflow.com/questions/63261220/link-errors-with-ffast-math-ffinite-math-only-and-glibc-2-31)**. Feel free to build `.a/.so` for integration, or just copy the sources into your project.      


## The problem
Someday when I try to build a project on my new computer that runs `Debian11`, a lot of `undefined reference` errors occurred, which leads to symbols missed. At a glance I find that all the missed symbols have `_finite` suffix and looks like math functions, for example, `undefined reference to __pow_finite`. But why?    

## Reproduction
Let's try to reproduce it step-by-step.    

### Test enviornments
First of all, to reproduce this issue I need two different enviornments, which are key factors for reproducing this issue. They're setup as below via docker:    

- `Debian 11`    
```bash
$ gcc --version
gcc (Debian 10.2.1-6) 10.2.1 20210110
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$ ldd --version
ldd (Debian GLIBC 2.31-13+deb11u2) 2.31
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.

$ ll /lib/x86_64-linux-gnu/libc-*.so
-rwxr-xr-x 1 root root 1839792 Oct  2 12:47 /lib/x86_64-linux-gnu/libc-2.31.so
```

- `Debian 10`    
```bash
$ gcc --version
gcc (Debian 8.3.0-6) 8.3.0
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$ ldd --version
ldd (Debian GLIBC 2.28-10) 2.28
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.

$ ll /lib/x86_64-linux-gnu/libc-*.so
-rwxr-xr-x 1 root root 1824496 May  1  2019 /lib/x86_64-linux-gnu/libc-2.28.so
```

### Normal routine
Let's say if we have a library `libfoo` and an `app` uses it. The codes may like this:     
```c
// libfoo/foo.h
double foo(double x, double y);

// libfoo/foo.c
#include "foo.h"
#include <math.h>
double foo(double x, double y) {
    return pow(x, y);
}

// app/main.c
#include "foo.h"
#include <stdio.h>
int main () {
    printf("foo(2.0, 5.0) = %f\n", foo(2.0, 5.0)); 
    return 0;
}
```      

Then build and run it:     
```bash
# we put main.c in app/ and foo.* in libfoo
$ tree
.
|-- app
|   `-- main.c
`-- libfoo
    |-- foo.c
    `-- foo.h

# build libfoo.a
$ gcc -c -o foo.o libfoo/foo.c
$ ar rcs libfoo.a foo.o

# build app
$ gcc -Ilibfoo -L. -o app.out app/main.c -lfoo -lm

# run app
$ chmod +x ./app.out && ./app.out
foo(2, 5) = 32.000000
```
Now we see the `app` calls `foo` successfully and output `foo(2.0, 5.0) = 32.000000`.       




TODO: 

```bash
# build static library
gcc -c -o libmath-finite.o libmath-finite.c
ar rc libmath-finite.a libmath-finite.o

# link the library in your project
```

It will NOT happen if you build all your source codes at once.

## References
- [Math changes in glibc 2.31](https://sourceware.org/pipermail/libc-alpha/2020-May/113773.html)
- [Math changes in glibc 2.31 - Answer page](https://sourceware.org/pipermail/libc-alpha/2020-May/113840.html)
- [Proposal: Remove or reduce math-finite.h](https://sourceware.org/legacy-ml/libc-alpha/2019-03/msg00395.html)
- [Link errors with -ffast-math (-ffinite-math-only) and glibc 2.31](https://stackoverflow.com/questions/63261220/link-errors-with-ffast-math-ffinite-math-only-and-glibc-2-31)
- [sourceware.org - glibc.git](https://sourceware.org/git/?p=glibc.git)
- [sourceware.org - glibc.git - commit: remove math-finite.h redirections for math functions](https://sourceware.org/git/gitweb.cgi?p=glibc.git;h=7bdb921d70bf9f93948e2e311fef9ef439314e41)
- [C - Common mathematical functions](https://en.cppreference.com/w/c/numeric/math)
- [FloatingPointMath](https://gcc.gnu.org/wiki/FloatingPointMath)
