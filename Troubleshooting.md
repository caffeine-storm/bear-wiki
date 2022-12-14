For many of problems, the best way to solve it is to build some understanding how Bear works. This section will help to get familiar with Bear's internals and also advice what to check in certain error cases.

# How it works?

To build the compilation database, Bear split this work into two major steps:

1. It execute the build and intercepts the executed commands (most likely the compiler calls).
1. It read the command execution log, and build the final output (deduce the semantic of the commands).

In the current implementation these two steps are implemented in two separate executable, and a third one is calling this two programs one after the other. This 3rd program is called `bear`, and that's what users interact with mostly. The 1st command intercepting program is called `intercept`. The 2nd reasoning program is called `citnames` (it's the word "semantic" just backwards).

Running `bear` with verbose option, you can observe the `intercept` and `citnames` program executions. A set of `-v` flag can trigger the verbose logging:

    $ bear -vvvv -- <build command>

## How intercept works?

Currently there are 2 modes how command interception is possible with this tool:

1. The operating system dynamic linker library preload mechanism.
1. Interposing compiler wrappers.

Both solutions have some limitations, and a context when it performs better than the other.

### Intercept with dynamic linker preload

Many operating systems are supporting dynamic linking, and many of these [dynamic linker](https://en.wikipedia.org/wiki/Dynamic_linker) supports library preloading. It means that a shared library is loaded first into the memory when a program is executed. The `intercept` program is using it to record the program execution calls. (The preloaded library is implementing the [`exec*` POSIX functions](https://en.wikibooks.org/wiki/C_Programming/POSIX_Reference/unistd.h/exec) and part of the Bear project.)

The `intercept` program is using this mechanism to "hijack" the program executions from the build tool. When your build tool wants to execute a program, it will call one of the `exec*` function. But the preloaded library `exec*` function gets called. And this library will not execute the program the build tool was calling, but will execute another one. This another program is called `wrapper`. When the library executes the `wrapper` command, it passes all context to it.

What the `wrapper` command is doing? Let's discuss it after the compiler wrappers, because that mode is also using it.

### Intercept with interposing compiler wrappers

In this mode, the `intercept` tool is relies on the build tool usage of environment variables. By conventions, build tools are using the following environment variables to control which compiler tools to use for compiling C or C++ sources:

- `CC` for C compiler
- `CXX` for C++ compiler
- `AS` for assembler
- `FC` for fortran compiler

You can read more about this variables [here](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html).

The idea here, that `intercept` will override these environment variables before calling the build tool. So, the build tool will call the tools that `intercept` was interpose (and not the real compilers). The tool that `intercept` is interposing as compiler is the same `wrapper`. But this time, it can't pass the whole context of the execution (because the execution will be done by the build tool). So, when Bear is installed on a machine, it not only installed the `wrapper` program, but a set of soft links to the `wrapper` program. (In this wrapper directory, you will find all the tools that `intercept` can interpose.)

Now let's see how the `wrapper` program works...

### What the `wrapper` command is doing?

When the `wrapper` command is executed during the build process, it is expected that it will execute a "real" program (like the C compiler, or the linker). So, the `wrapper` needs to produce the same output as the "real" program would do. How does it do?

- It calls the "real" program,
- or it fakes the "real" program outputs.

In the current implementation, it always calls the real program. How does it know what's the real program is?

When the `wrapper` is called, it has an environment variable that contains an IPC address to talk back to `intercept` program. (In the process tree, the `intercept` process is a parent process of the build process, and the build process is a parent process of the `wrapper` process.) So, what the `wrapper` to `intercept` IPC does?

- It asks for the "real" program,
- it reports the program execution,
- and it report the program execution status (when the process terminated).

## Deduce the semantic of the commands

At this stage of the run the `intercept` program is terminated (the build is finished). We have a list of commands (as input) to create the compilation database. The compilation database, has compiler calls as entries. It gives these main tasks for `citnames`:

- recognize compiler calls: it starts with the program name (if it is a known program name like `cc`, `gcc` or `clang`).
- recognize the compile pass: we only want to see compiler calls which are compiling (not interested when it linking only).
- filter the argument list: we only want the compilation flags (and not interested linker flags).

When the candidate elements for the compilation database are calculated, it saves them into the JSON compilation database. Since we are supporting "append mode", it takes care to not append elements which are already in.

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

```
wrapper: failed with: gRPC call failed: Connection reset by peer
``` 

```
wrapper: failed with: gRPC call failed: failed to connect to all addresses
```

These error messages are coming from the gRPC client, which can be influenced by environment variables. The most probable cause when the HTTP proxy environment variables are presents. (`http_proxy`, `https_proxy`, `all_proxy` and their capitalized versions.)

**Workarounds**:

- Unset the HTTP proxy variables.
- When unset is not an option, try to set `no_proxy=localhost,127.0.0.1` as environment variable.

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

Running Bear outside of a Docker command will not work. Because `docker exec ...` is not executing anything, it just sends the command to the `docker` daemon, and that will execute the command. Bear has no access an already running process child processes, can't intercept the executed processes.

**Workarounds**:

- Clean your build (eg.: run `make clean`) and run your build with Bear again.
- Run the "configure" step with Bear too. Discard its output, and proceed with the build with Bear.
- In case if you are using non default compilers, you might want to write a configuration file to hint Bear which compilers to recognize.
- If you are running your build inside a Docker container, run Bear inside the container too.

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
