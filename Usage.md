# Basic Usage

Bear uses operating system features to intercept process executions during the build process. Which means it is unaware about the build system. It records only those executions which it were seen.

It implies if your build breaks and the build process stops. The output will contains only those execution which was happened. (Future ones is not visible to it.) And if you restart the build, and it resumes the build (and do not recompile the already compiled) those element will be missing from the output.

* One solution for it is to force your build system to recompile everything.
* The other one is to ask Bear to append the current run results to an existing ones.

# Cross Compilers

# Multilib Issues

# Compiler Wrappers

## distcc

## Ccache

## MPI

## CUDA

# OS support

## Linux

## FreeBSD

## OSX

### SIP

Security extension/modes on latest OSX releases prevent the dynamic linker to preload libraries. This case Bear behaves normally, but the result compilation database will be empty.

To check is SIP enabled run: `csrutil status | grep 'System Integrity Protection'`

* Workaround could be to disable the security feature while running Bear. This might involve reboot of your computer, so might be heavy workaround.
* Another option if the build tool is not from the official XCode, but installed from some other sources. (eg.: instead of using the system `make` command, try to install `gmake` from `brew`.)

### Static binary

Bear uses dynamic linker to work. Which implies if the build tool is staic binary, so the dynamic linker is not involved, so Bear intercept logic is not called, so the executions are not logged and it results an empty output.

* Workaround could be to use a non static build tool. (The examples I've got was: using the system `sh` to call the compiler makes empty output. While install `bash` from `brew` fix the issue.)

## Windows

## AIX

## Solaris

# Build tools

## Make

## SCons

## CMake

## QMake

## meson
