For many of problems, the best way to solve it is to build some understanding how Bear works. This section will help to get familiar with Bear's internals and also advice what to check in certain error cases.

TODO: explain what Bear does

# The build with and without Bear behaves differently

There are two types of differences: the harmless and the harmful. The harmless category means that the result of your build process result the same output, but there were messages during the build which were not there before. The harmful category include cases, when the result of the build was influenced by Bear.

## Error messages which appears with Bear

```
ERROR: ld.so: object '/usr/local/$LIB/bear/libexec.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
```

First, I would like to point out that `$LIB` in the error message does not refer to an environment variable called `LIB`. This is a defined symbol for the dynamic linker on your system, which expands in a `LD_PRELOAD` usage. (Read `man ld.so` for more.)

Depending on your Linux distribution and the architecture of your machine, the `$LIB` expands to `lib`, `lib64`, `lib/i386-linux-gnu`, etc... Check out what is the directory name in `/usr` that contains your `libc.so`. Fedora running on a `x86_64` machine, this is `/usr/lib64/libc.so`, therefore the `$LIB` expands to `lib64`. In this case the `libexec.so` should be installed as `/usr/local/lib64/bear/libexec.so`.

CMake somehow does not pay attention for this important detail. And it installs the library as `/usr/local/lib/bear/libexec.so`. (That's why the [INSTALL.md](https://github.com/rizsotto/Bear/blob/master/INSTALL.md) gives instruction how to change that.)

**Workarounds**:

- Use an OS package if available. (Packagers are already solved this issue.)
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

In case if you are using cross compilers (or not the default `cc`), Bear might miss to recognize that as a compilation step.

**Workarounds**:

- Clean your build (eg.: run `make clean`) and run your build with Bear again.
- Run the "configure" step with Bear too. Discard its output, and proceed with the build with Bear.
- In case if you are using non default compilers, you might want to write a configuration file to hint Bear which compilers to recognize.

# The output is missing entries.

You've been running a build with Bear, and found that the compilation database is not complete, missing entries which should be there. The reason for this is very similar to [output is empty](https://github.com/rizsotto/Bear/wiki/Troubleshooting#the-output-is-empty) problem above.

The most common cause for this is, incremental builds not running the compilers if everything is up-to-date. Remember, Bear does not
understand the build file (eg.: makefile), but intercepts the executed commands.

The other cause for this is, that Bear was not intercepted or recognized all tools in your build. This case is discussed in "the output is empty" section above.

**Workarounds**:

- Same workarounds as for [The output is empty](https://github.com/rizsotto/Bear/wiki/Troubleshooting#the-output-is-empty).
- Use `--append` flag on Bear, so previous run results are not overwritten, but extended.

# The output has duplicate entries.

What counts as duplicate entry? In the JSON compilation database, there are no single primary key. None of the attributes are required to be unique. All attributes has to be identical to call it a duplicate. (If Bear emits such output, that's a bug on Bear.)

In some cases the duplicate entries are the result of:

- The project builds the same module multiple times. This is not ideal, but could be a good reason to do so.
  - The build was creating a debug and non-debug version.
  - The build recompile modules for test (with different flags).
  - The build runs different compilers against the same modules to validate portability.
- The project tracks module dependencies with the help of compiler.
  - It uses `-M` flags to emit `make` dependency files.

**Workarounds**:

- Use the configuration file of Bear to filter out entries based on their location. (`paths_to_include` and `paths_to_exclude` fields)
- Use the configuration file of Bear to filter out entries based on compiler. (`compilers_to_exclude` field)
- Use the configuration file of Bear to filter out flags, therefore make two compiler calls identical, which will result a single entry for that. (`flags_to_remove` field)

# The output has entries, which are not part of the project.

The two known scenarios for this problems are:

- You are using the `--append` flag. Which carries over existing entries from previous run. (And you've changed the project since, so removed files will appear in the output.)
- You are including the "configure" step in the build.

**Workarounds**:

- Clean your build (eg.: run `make clean`) and run your build with Bear again without the `--append` flag.
- In case if you need to run the configure step with Bear. (Using the compiler wrappers it is desired to capture the compilers locations.) Run that step separately from your build, and remove the output after the "configure" step.
