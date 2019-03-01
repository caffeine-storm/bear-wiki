This page describes the config file format for version 3.1 and onward.

# Location

Will search for these files:

* the explicitly given command line `--config=PATH_TO_FILE`,
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

```yaml
intercept:
  mode: preload
```

## Output

The output of the compilation database is defined. But the same compilation can be represented many different ways.

* The file and directory paths can be absolute or relative to the project root.
* The commands can be rendered as a single shell `command` (string) or the list of `arguments` (array).
* The entry of the database may contain the compilation output file name.
* The entry of the database may be with the original compiler wrapper or without it.

```yaml
output:
  relative_to: "/path/to/project/sources"
  command_as_array: true
  drop_output_field: false
  drop_wrapper: true
```

## Strategy

```yaml
strategy:
  append_to_existing: false
  include_headers: false
  include_linking: false
  compilers: ..
  sources: ..
  flags: ..
```

## Compiler

```yaml
compilers:
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
```

## Sources

To identify the sources can be done by its file name extensions or the location of the files.

```yaml
sources:
  extensions_to_exclude:
    - ".o"
  extensions_to_include:
    - ".c"
    - ".cc"
  paths_to_exclude:
    - "/path/to/build/dir"
  paths_to_include:
    - "/path/to/source/dir"
```

## Flags

```yaml
  flags:
    to_exclude:
      "-MD": 0
      "-MMD": 0
```
