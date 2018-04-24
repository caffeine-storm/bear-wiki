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

Security extension/modes on latest OSX releases prevent the dynamic linker to preload libraries. This case Bear behaves normally, but the result compilation database will be empty.

To check is SIP enabled run: `csrutil status | grep 'System Integrity Protection'`

* Workaround could be to disable the security feature while running Bear. This might involve reboot of your computer, so might be heavy workaround.
* Another option if the build tool is not installed under certain directories.

## Windows

## AIX

## Solaris

# Build tools

## Make

## SCons

## CMake

## QMake

## meson
