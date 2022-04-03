# Undefined reference to `__xxx_finite` - why and how?
This article explains the **undefined reference to `__xxx_finite`** problem, what is it, why it happens, and how to solve it.     

## The problem
Someday when I try to build a project on my new computer that runs `Debian 11`, a lot of `undefined reference` errors occurred, which leads to symbols missed. At a first glance I find that all the missed symbols have `_finite` suffix and looks like math functions, for example, `undefined reference to __pow_finite`. But why?    

## Reproduction
Let's try to reproduce it step-by-step.    

### Test enviornments
First of all, I setup two different enviornment as below to reproduce this problem:     

- `Debian 11`    
```bash
$ gcc --version
gcc (Debian 10.2.1-6) 10.2.1 20210110
...

$ ldd --version
ldd (Debian GLIBC 2.31-13+deb11u2) 2.31
...

$ ll /lib/x86_64-linux-gnu/libc-*.so
-rwxr-xr-x 1 root root 1839792 Oct  2 12:47 /lib/x86_64-linux-gnu/libc-2.31.so
```

- `Debian 10`    
```bash
$ gcc --version
gcc (Debian 8.3.0-6) 8.3.0
...

$ ldd --version
ldd (Debian GLIBC 2.28-10) 2.28
...

$ ll /lib/x86_64-linux-gnu/libc-*.so
-rwxr-xr-x 1 root root 1824496 May  1  2019 /lib/x86_64-linux-gnu/libc-2.28.so
```

Please noted that `ldd --version` indicates the version of `libc` and other included libraries, including `libm`, etc.    

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
$ gcc -c -ffast-math -o foo.o libfoo/foo.c
$ ar rcs libfoo.a foo.o

# build app
$ gcc -Ilibfoo -L. -o app.out app/main.c -lfoo -lm

# run app
$ chmod +x ./app.out && ./app.out
foo(2, 5) = 32.000000
```
Now we see the `app` calls `foo` successfully and output `foo(2.0, 5.0) = 32.000000`. It works well on both `Debian 10` and `Debian 11`.         
Be aware that we build `libfoo.a` with `-ffast-math` to leverage the [GCC FloatingPointMath Optimiation](https://gcc.gnu.org/wiki/FloatingPointMath).     


### Build `libfoo` on `Debian 10`
Now let's try to build the `libfoo` on `Debian 10`.    

```bash
# build libfoo.a on Debian 10 
$ gcc -c -ffast-math -o foo.o libfoo/foo.c
$ ar rcs libfoo.a foo.o

# build app on Debian 11, with `libfoo.a` copied from Debian 10
$ gcc -Ilibfoo -L. -o app.out app/main.c -lfoo -lm
/usr/bin/ld: libfoo-prebuilt-debian10/libfoo.a(foo.o): in function `foo':
foo.c:(.text+0x1d): undefined reference to __pow_finite
collect2: error: ld returned 1 exit status

```

It happens! But why?     

## Analysis

Let's look at the `libfoo.a` to check what inside it and what it need.    

```bash
# libfoo.a built on Debian 10
$ nm libfoo.a

foo.o:
                 U _GLOBAL_OFFSET_TABLE_
                 U __pow_finite
0000000000000000 T foo
```

The `nm` commands show that `libfoo.a` includes a `foo` function that matchs above source code. Also, it needs an external symbol `__pow_finite`, which is the error reported. But if the symbol is missed, why it works well if it's compiled on `Debian 11`? Let's checkout the `libfoo.a` that built `on Debain 11`.      

```bash
# libfoo.a built on Debian 11
$ nm libfoo.a

foo.o:
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 T foo
                 U pow
```

The `libfoo.a` built on `Debian 11` uses `pow` instead of `__pow_finite`, that's why it works well.     
Now let's compare the two versions of `libm` to see what's the different on this symbol.       

```bash
# libm-2.31 on Debian 11
$ nm -D /lib/x86_64-linux-gnu/libm-2.31.so | grep __pow_finite
000000000002e450 i __pow_finite@GLIBC_2.15

# libm-2.28 on Debian 10
$ nm -D /lib/x86_64-linux-gnu/libm-2.28.so | grep __pow_finite
000000000002d3e0 i __pow_finite

```

The `libm-2.28` on `Debian 10` has a default `__pow_finite`, so that `app` and link correctly. But `libm-2.31` on `Debian 11` only has an `__pow_finite@GLIBC_2.15` for backward compatibility, which can NOT be linked by default. [How the GNU C Library handles backward compatibility](https://developers.redhat.com/blog/2019/08/01/how-the-gnu-c-library-handles-backward-compatibility) explains the `glibc` backward compatibility handling.        

After some research, [Proposal: Remove or reduce math-finite.h](https://sourceware.org/legacy-ml/libc-alpha/2019-03/msg00395.html) tells us that **the *_finite names are just aliases of the normal name and so the asm attribute isn't accompishing anything**, and the [sourceware.org - glibc.git - commit: remove math-finite.h redirections for math functions](https://sourceware.org/git/gitweb.cgi?p=glibc.git;h=7bdb921d70bf9f93948e2e311fef9ef439314e41) removes them completely. It's actually an ABI change of `glibc` libraries, but doesn't properly handled.    

## How to solve    
Now we know why it happens, and there're several ways to solve it.      

### If you can change the source codes    
Simply add `-fno-finite-math-only` in compilation. For example,      

```bash
# build libfoo.a on Debian 10 
$ gcc -c -ffast-math -fno-finite-math-only -o foo.o libfoo/foo.c
$ ar rcs libfoo.a foo.o
$ nm libfoo.a

foo.o:
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 T foo
                 U pow
```

The `-fno-finite-math-only` flag tells compiler that don't use `finite-math` functions only, so that the result `libfoo.a` relis on `pow` directly, and it works.      

### If you only has an pre-built `.a/.so` that requires `__xxx_finite` functions     
You can not change the `.a/.so`'s source code and building process.      

#### Way1. Copy the `libmath-finite.c/h` into your project and compile them together     
For example,     

```bash
# build the app with `libmath-finite.c/h` together   
$ gcc  -Ilibfoo -L. -I../.. -o app.out app/main.c ../../libmath-finite.c -lfoo -lm
$ 
$ nm app.out  | grep __pow_finite
00000000000012a4 T __pow_finite
```

In this case, the `__pow_finite` symbol has been defined in the `app` directly.      

#### Way2. Build `libmath-finite.a` and link it in your project      
For example,     

```bash
# build `libmath-finite.a`    
$ gcc -c -o libmath-finite.o libmath-finite.c
$ ar rcs libmath-finite.a libmath-finite.o

# build app that linking `libmath-finite.a`
$ gcc  -Ilibfoo -L. -L../.. -o app.out app/main.c -lfoo -lmath-finite -lm
```

Be aware that `-lmath-finite` has to be placed after `-lfoo` and before `-lm`, otherwise the symbol can not be found correctly.      

#### Way3. Build `libmath-finite.so` and link it in your project      

For example,     

```bash
# build `libmath-finite.so`    
$ gcc -c -fPIC -o libmath-finite.o libmath-finite.c
$ gcc -shared -fPIC -o libmath-finite.so libmath-finite.o -lm

# build app that linking `libmath-finite.a`
$ gcc  -Ilibfoo -L. -L../.. -o app.out app/main.c -lfoo -lmath-finite -lm

# run app
$ export LD_LIBRARY_PATH=$(pwd)/../..
$ ./app.out
foo(2.0, 5.0) = 32.000000
```

Same with **Way2**, the `libmath-finite.so` should be linked correctly. On the other hand, the `libmath-finite.so` should be distributed with your program together, and need to be find by your program when run. In this example, `export LD_LIBRARY_PATH=$(pwd)/../..` is helpful to make `app.out` find the `libmath-finite.so`.     

## Conclusion    
- This problem is due to a ABI change of `glibc` between `2.30` and `2.31`;       
- You may fix it by various ways described above.      


## References
- [Math changes in glibc 2.31](https://sourceware.org/pipermail/libc-alpha/2020-May/113773.html)
- [Math changes in glibc 2.31 - Answer page](https://sourceware.org/pipermail/libc-alpha/2020-May/113840.html)
- [Proposal: Remove or reduce math-finite.h](https://sourceware.org/legacy-ml/libc-alpha/2019-03/msg00395.html)
- [Link errors with -ffast-math (-ffinite-math-only) and glibc 2.31](https://stackoverflow.com/questions/63261220/link-errors-with-ffast-math-ffinite-math-only-and-glibc-2-31)
- [sourceware.org - glibc.git](https://sourceware.org/git/?p=glibc.git)
- [sourceware.org - glibc.git - commit: remove math-finite.h redirections for math functions](https://sourceware.org/git/gitweb.cgi?p=glibc.git;h=7bdb921d70bf9f93948e2e311fef9ef439314e41)
- [C - Common mathematical functions](https://en.cppreference.com/w/c/numeric/math)
- [GCC - FloatingPointMath Optimiation](https://gcc.gnu.org/wiki/FloatingPointMath)
- [GCC - Options That Control Optimization](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
- [How the GNU C Library handles backward compatibility](https://developers.redhat.com/blog/2019/08/01/how-the-gnu-c-library-handles-backward-compatibility)
