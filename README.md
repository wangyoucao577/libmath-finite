# libmath-finite
TODO: 

```bash
# build static library
gcc -c -o libmath-finite.o libmath-finite.c
ar rc libmath-finite.a libmath-finite.o

# link the library in your project
```
## References
- [Math changes in glibc 2.31](https://sourceware.org/pipermail/libc-alpha/2020-May/113773.html)
- [Math changes in glibc 2.31 - Answer page](https://sourceware.org/pipermail/libc-alpha/2020-May/113840.html)
- [Proposal: Remove or reduce math-finite.h](https://sourceware.org/legacy-ml/libc-alpha/2019-03/msg00395.html)
- [Link errors with -ffast-math (-ffinite-math-only) and glibc 2.31](https://stackoverflow.com/questions/63261220/link-errors-with-ffast-math-ffinite-math-only-and-glibc-2-31)
- [sourceware.org - glibc.git](https://sourceware.org/git/?p=glibc.git)
- [sourceware.org - glibc.git - commit: remove math-finite.h redirections for math functions](https://sourceware.org/git/gitweb.cgi?p=glibc.git;h=7bdb921d70bf9f93948e2e311fef9ef439314e41)
- [C - Common mathematical functions](https://en.cppreference.com/w/c/numeric/math)

