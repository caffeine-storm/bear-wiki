This page describes the config file format for version 3.1 and onward.

# Location

Will search for these files:

* the explicitly given command line `--config=PATH_TO_FILE`,
* the `BEAR_CONFIG` environment pointed location,
* the current directory as`./bear.conf`,
* the user home directory as `~/.config/bear.conf`.

When you are unsure what config it reads, just add `--config-dump` at the end of the options and it will dump the content.

# Content

The configuration file format is [yaml](http://yaml.org/)

## Intercept

To create a compilation database involves to capture the executed commands. This will be the source of creating the output.

Intercepting command executions can be done many ways. Each has its own limitations and benefits.

* Dynamic library **preload** is done by the dynamic linker of the operating system. The process execution C functions (which are defined in the `libc` library) are hijacked and another implementation is used. Can be used in docker containers. But does not work with statically linked build systems, or daemons.

* Compiler **wrapper** is a fake compiler set, which does not do real compilation but only reports the execution. Can be used with statically linked build tools or when dynamic library preload does not work. It might be significantly faster than executing the build with the real tools. But it records only the compiler and linker calls.

* Using **ptrace** functions. Can be very slow. Might not work within containers. (Available in 3.2)

The interception output can be appended to earlier run. This way multiple build phases can be seen as a single execution.

```yaml
intercept:
  mode: preload
  append: false
```

## Output

The output of the compilation database is defined. But the same compilation can be represented many different ways.

* The file and directory paths can be absolute or relative to the project root.
* The output can contains only the C/C++ source files, but it can also contains the header files too.
* The commands can be rendered as a single shell`command` (string) or the list of `arguments` (array).
* The entry of the database may contain the output file name too. 

```yaml
output:
  relative: "/path/to/project/sources"
  headers: false
  command: array
  output: false
```

## Sources

To identify the sources can be done by its file name extensions or the location of the files.

* `sources.extensions` is a list of file name extension which will be considered as source file.
* `sources.paths` is a list of directories where any file having such prefix will be considered as source file.

```yaml
sources:
  extensions:
    - ".c"
    - ".cc"
  paths:
    - "/path/to/source/dir"
```

## Compiler

* `compiler.phases` is a list of the compilation phases (`preproc`, `compilation`, `link`) which shall be included in the result.
* `compiler.flags` is a map of the flags which shall be left out from the output. The key of the map is the flag name, the value is how many following arguments shall be removed. (eg.: `"-MD": 0` means that only the `-MD` argument will be removed. While `"-MF": 1` means that `-MF` and the following `/path/source.c.d` will be removed.)

```yaml
compiler:
  languages:
    c++:
      - g++
      - clang++
    c:
      - gcc
      - clang
    mpi:
      - mpiCC
      - mpicc
      - mpicxx
      - mpic++
    wrapper:
      - distcc
      - ccache
  phases:
    - compilation
  flags:
    "-MD": 0
    "-MMD": 0
```
