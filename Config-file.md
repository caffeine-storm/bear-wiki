This page describes the config file format for version 3.1 and onward.

# Location

Will search for these files:

* the explicitly given command line `--config=PATH_TO_FILE`,
* the `BEAR_CONFIG` environment pointed location,
* the current directory as`./.bear.yaml`,
* the user home directory as `~/.bear.yaml`.

When you are unsure what config it reads, just add `--config-dump` at the end of the options and it will dump the content.

# Content

The configuration file format is [yaml](http://yaml.org/)

```yaml
compiler:
  languages:
    c++:
      - c++
      - g++
      - clang++
    c:
      - cc
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
    # available options: preproc, compilation, link
    - compilation
  flags:
    ignore:
      "-MD": 0
      "-MMD": 0

source:
  extension:
    - ".c"
    - ".C"
    - ".cc"
    - ".cpp"

output:
  # set the file/directory paths relative to
  # the current directory (or the given location).
  realative: true
  # include header files in the output.
  headers: false
  # present the command arguments as "array" or "string".
  command: array
  # emit the output file
  output: false
```