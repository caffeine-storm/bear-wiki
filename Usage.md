# Basic Usage

After installation the usage is like this:

    bear <your-build-command>

The output file called `compile_commands.json` is saved in the current directory.

For more options you can check the man page or pass `--help` parameter.

## Limitations

Bear uses operating system features to intercept process executions during the build process. Which means it is unaware about the build system. It records only those executions which it were happen during the build process. (It does not read the build description (`Makefile`, `CMakeLists.txt`, etc..), but it intercepts the commands were executed.)

It implies if your build breaks and the build process stops. The output will contains only those execution which was happened. (Future ones is not visible to it.) And if you restart the build, and it resumes the build (and do not recompile the already compiled) those element will be missing from the output.

* One solution for it is to force your build system to recompile everything.
* The other one is to ask Bear to append the current run results to an existing ones.

# Cross Compilers

Cross compilers should work with Bear. If Bear works with a native compiler then it should work with a cross compiler too. The only catch is that Bear does not know the compiler name to recognize it, so you need to explicitly pass the compiler names. (See `--use-cc`, `--use-c++` or `--use-only` flags in the help or in the manual page for more.)

# Multilib Issues

Multilib is one of the solutions allowing users to run applications built for various application binary interfaces (ABIs) of the same architecture. The most common use of multilib is to run 32-bit applications on 64-bit kernel.

For OSX this is not an issue. The build commands from previous section will work, Bear will intercept compiler calls for 32-bit and 64-bit applications.

For Linux, a small tune is needed at build time. Need to compile `libear.so`/`libexec.so` library for 32-bit and for 64-bit too. Then install these libraries to the OS preferred multilib directories. And replace the `libear.so`/`libexec.so` path default value with a single path, that matches both. (The match can be achieved by
the `$LIB` token expansion from the dynamic loader. See `man ld.so` for more.)

Debian derivatives are using `lib/i386-linux-gnu` and `lib/x86_64-linux-gnu`, while many other distributions are simple `lib` and `lib64` directories. Here comes an example build script to install a multilib capable Bear. It will install Bear under `/opt/bear` on a non Debian system.

    (cd ~/build32; cmake "$BEAR_SOURCE_DIR" -DCMAKE_C_COMPILER_ARG1="-m32"; VERBOSE=1 make all;)
    (cd ~/build64; cmake "$BEAR_SOURCE_DIR" -DCMAKE_C_COMPILER_ARG1="-m64" -DDEFAULT_PRELOAD_FILE='/opt/bear/$LIB/libear.so'; VERBOSE=1 make all;)
    sudo install -m 0644 ~/build32/libear/libear.so /opt/bear/lib/libear.so
    sudo install -m 0644 ~/build64/libear/libear.so /opt/bear/lib64/libear.so
    sudo install -m 0555 ~/build64/bear/bear" /opt/bear/bin/bear

To check your installation, install `lit` and run the test suite.

    PATH=/opt/bear/bin:$PATH lit -v test
    PATH=/opt/bear/bin:$PATH lit -v test -DMULTILIB=true

# Compiler Wrappers

Compiler wrappers are programs which are behaving like a compiler and are executing a real C/C++ compiler. The real compiler might be called with different command line arguments. (The challenge here is to output either the wrapper call or the real compiler call, but not both.)

The supported/recognized wrappers are:

* [distcc](http://distcc.org/)
* [ccache](https://ccache.dev/)
* MPI (The output contains the real compiler call.)
  * [OpenMPI](https://www.open-mpi.org/)
  * [MPICH](https://www.mpich.org/)
* [CUDA](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html) (The output contains the wrapper call)

# OS support

## Linux

Supported. (Check for distribution [packages](https://repology.org/project/bear/versions).)

## FreeBSD

Supported. (Check for distribution [package](https://www.freshports.org/devel/bear/).)

## OSX

Supported. (Check for distribution [package](https://formulae.brew.sh/formula/bear).)

### SIP

Security extension/modes on latest OSX releases prevent the dynamic linker to preload libraries. This case Bear behaves normally, but the result compilation database will be empty.

To check is SIP enabled run: `csrutil status | grep 'System Integrity Protection'`

* Workaround could be to disable the security feature while running Bear. This might involve reboot of your computer, so might be heavy workaround.
* Another option if the build tool is not from the official XCode, but installed from some other sources. (eg.: instead of using the system `make` command, try to install `gmake` from `brew`.)

## Windows

Will be supported in 3.0+ version.

## AIX

Not yet supported. (Contact me if you want to develop this.)

## Solaris

Not yet supported. (Contact me if you want to develop this.)

# Build tools

Bear is build tool independent. But there are build systems which are problematic
or does not require to use Bear to produce the compilation database.

## Bazel

The two main constraints to intercept compiler execution from bazel builds are:
bazel runs a daemon which runs the compilations, and it creates an isolated
environment to run the compiler. These problems are not just hard to circumvent,
but the workaround would not be stable to support it by this tool.

The good news is: there are extensions for bazel to generate the compilation
database.

## CMake

CMake creates compilation database out of the box. You don't need to use Bear
for that. Just pass `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` flag when you call
`cmake`.

## Statically linked build-tool/compiler

Bear uses dynamic linker to work. Which implies if the build tool is static binary, so the dynamic linker is not involved, so Bear intercept logic is not called, so the executions are not logged and it results an empty output.

* Workaround could be to use a non static build tool. (The examples I've got was: using the system `sh` to call the compiler makes empty output. While install `bash` from package manager fix the issue.)
