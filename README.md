# libmath-finite     
**libmath-finite** intends to solve the `__xxx_finite` symbols missed problem due to `glibc` upgrade. It generally happens when distribute your pre-built binray(`.a/.so/executable`) from `glibc<2.31` build enviornment to `glibc>=2.31` systems, such as `Debian 10 -> Debian 11`, or `Ubuntu18.04 -> Ubuntu20.04`. [Math changes in glibc 2.31](https://sourceware.org/pipermail/libc-alpha/2020-May/113773.html) also discusses the problem.          
It's a very simple library, which only wraps functions of the standard math library `libm`. **Many thanks to **Étienne** who proposes the idea in [Link errors with -ffast-math (-ffinite-math-only) and glibc 2.31](https://stackoverflow.com/questions/63261220/link-errors-with-ffast-math-ffinite-math-only-and-glibc-2-31)**.    

## Why need it
Please read [Undefined reference to `__xxx_finite` - why and how?](./doc/undefined_reference_to__xxx_finite__why_and_how.md).       

## How to use
There're various ways to use it. The simplest way is just copy the sources into your project and compile them together. Alternativesly build static/shared library to and link them. Read [Undefined reference to `__xxx_finite` - why and how?](./doc/undefined_reference_to__xxx_finite__why_and_how.md) for more details.        

- Build static library

```bash
$ gcc -c -o libmath-finite.o libmath-finite.c
$ ar rcs libmath-finite.a libmath-finite.o
```

- Build shared library

```bash
$ gcc -c -fPIC -o libmath-finite.o libmath-finite.c
$ gcc -shared -fPIC -o libmath-finite.so libmath-finite.o -lm
```

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
- [Undefined reference to `__xxx_finite` - why and how?](./doc/undefined_reference_to__xxx_finite__why_and_how.md)    
- [Undefined reference to `__xxx_finite` - why and how? - medium.com](https://medium.com/@wangyoucao577/undefined-reference-to-xxx-finite-why-and-how-84f52766f1b4)
- [Undefined reference to `__xxx_finite` - why and how? - github.io](https://wangyoucao577.github.io/en/2022/04/03/undefined_reference_to__xxx_finite__why_and_how.html)


