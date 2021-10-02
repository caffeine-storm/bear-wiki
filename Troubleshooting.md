For many of problems, the best way to solve it is to build some understanding how Bear works. This section will help to get familiar with Bear's internals and also advice what to check in certain error cases.

TBD

# The build with and without Bear behaves differently

TBD

## Error messages which appears with Bear

```
ERROR: ld.so: object '/usr/local/$LIB/bear/libexec.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
```

First, I would like to point out that `$LIB` in the error message does not refer to an environment variable called `LIB`. This is a defined symbol for the dynamic linker on your system, which expands in a `LD_PRELOAD` usage. (Read `man ld.so` for more.)

Depending on your Linux distribution and the architecture of your machine, the `$LIB` expands to `lib`, `lib64`, `lib/i386-linux-gnu`, etc... Check out what is the directory name in `/usr` that contains your `libc.so`. Fedora running on a `x86_64` machine, this is `/usr/lib64/libc.so`, therefore the `$LIB` expands to `lib64`. In this case the `libexec.so` should be installed as `/usr/local/lib64/bear/libexec.so`.

CMake somehow does not pay attention for this important detail. And it installs the library as `/usr/local/lib/bear/libexec.so`. (That's why the [INSTALL.md](https://github.com/rizsotto/Bear/blob/master/INSTALL.md) gives instruction how to change that.)

Workarounds:

- use an OS package if available. (Packagers are already solved this issue.)
- Figure out what the `$LIB` expands to on your system (as I've detailed above), and check the `libexec.so` file location.


# The output is empty

The most common cause for empty outputs is that the build command did not
execute any commands. The reason for that could be, because incremental builds
not running the compilers if everything is up-to-date. Remember, Bear does not
understand the build file (eg.: makefile), but intercepts the executed
commands.

The other common cause for empty output is that the build has a "configure"
step, which captures the compiler to build the project. In case of Bear is
using the _wrapper_ mode (read `intercept` man page), it needs to run the
configure step with Bear too (and discard that output), before run the build
with Bear.

# The output is missing entries.

TBD

# The output has duplicate entries.

TBD

# The output has entries, which are no longer part of the project.

TBD
