# Basic Usage

After installation the usage is like this:

    bear -- <your-build-command>

The output file called `compile_commands.json` is saved in the current directory.

For more options you can check the man page or pass `--help` parameter.

## Example

Suppose you have a `Makefile` and compile your project with the following command:

    make -j4

Then you'd just need to invoke bear as follows:

    bear -- make -j4

## Limitations

Bear uses operating system features to intercept process executions during the build process. Which means it is unaware about the build system. It records only those executions which it were happen during the build process. (It does not read the build description (`Makefile`, `CMakeLists.txt`, etc..), but it intercepts the commands were executed.)

It implies if your build breaks and the build process stops. The output will contains only those execution which was happened. (Future ones is not visible to it.) And if you restart the build, and it resumes the build (and do not recompile the already compiled) those element will be missing from the output.

* One solution for it is to force your build system to recompile everything.
* The other one is to ask Bear to append the current run results to an existing ones.

# Cross Compilers

Cross compilers should work with Bear. If Bear works with a native compiler then it should work with a cross compiler too. The only catch is that Bear does not know the compiler name to recognize it, so you need to configure the compiler names. (Read about the configuration file in `man citnames`.)

# Multilib Issues

Multilib is one of the solutions allowing users to run applications built for various application binary interfaces (ABIs) of the same architecture. The most common use of multilib is to run 32-bit applications on 64-bit kernel.

For OS-es which are using the _preload_ mode to intercept the executed commands, a small tune is needed. Need to compile `libexec.so` library for 32-bit and for 64-bit too. Then install these libraries to the OS preferred multilib directories. The `libexec.so` path value has to be a single path, that matches both. (The match can be achieved by the `$LIB` token expansion from the dynamic loader. See `man ld.so` for more.) The `INSTALL.md` file has a section which explains the how the build command should look like to enable the multilib build. Pay attention for the section, which describes the value of `CMAKE_INSTALL_LIBDIR`.

First, build and install the project for a 32 bit machine:

    cd ~/build32
    cmake -DCMAKE_C_COMPILER_ARG1="-m32" -DENABLE_MULTILIB=ON -DCMAKE_INSTALL_LIBDIR=lib/i386-linux-gnu "$BEAR_SOURCE_DIR"
    make all -j8
    make install

Second, build and install the project for 64 bit machine:

    cd ~/build64;
    cmake -DCMAKE_C_COMPILER_ARG1="-m64" -DENABLE_MULTILIB=ON -DCMAKE_INSTALL_LIBDIR=lib/x86_64-linux-gnu "$BEAR_SOURCE_DIR"
    make all -j8
    make install

This way the 64 bit install will overwrite the 32 binaries, but the necessary `libexec.so` will have both the 32 and 64 bit versions, because of the different installation paths.

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

## Windows

Not yet supported. (Contact me if you want to develop this.)

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

Bear uses the dynamic linker to work. This implies that, if the build tool is a
static binary, the dynamic linker will not be involved, so Bear's intercept
logic is not called. The result is that tool executions are not logged and an
empty output file will be created.

To work around this, use `--force-wrapper` to use compiler wrappers so that
static build tool calls _can_ be detected. Note that this requires the build
system to respect the `CC` and `CXX` environment variables when selecting the
compiler.
