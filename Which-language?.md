# Which language to use to implement Bear

## Tasks per components

1. `libear.so`
   1. to compile as shared object
   1. `libc` function calls
   1. vararg functions to implement
   1. no memory allocation
   1. environment reading
   1. conditional compilation
1. `wrapper`
   1. `libc` function calls
1. `pear`
   1. to compile as static executable
   1. tempfile/tempdir
   1. network communication
   1. environment read/write
   1. rdf read/write
   1. char encoding
1. `bear`
   1. tempfile/tempdir
   1. network communication
   1. environment read/write
   1. JSON read/write

## Tasks rating in languages

| Feature             | C/C++ | Rust | Python |
| ------------------- | -----:| ----:| ------:|
| comp. dyn. lib      |    ++ |   ++ |      - |
| comp. static.exe    |    ++ |   ++ |      - |
| comp. conditional   |    ++ |    ? |      - |
| call libc           |    ++ |    + |      - |
| impl. vararg        |    ++ |    - |      - |
| no mem. allocation  |    ++ |    + |      - |
| temp. read/write    |     + |    + |     ++ |
| env. read/write     |     + |    + |     ++ |
| JSON read/write     |     - |   ++ |     ++ |
| RDF read/write      |     - |    + |      + |
| network read/write  |     + |    ? |      - |

## Things to consider

1. Build system support for polyglot project?
   1. CMake: C/C++ compilation, conditional compilation, basic text processing, test runner, packaging
   1. Cargo: rust compilation, C/C++ compilation via build scripts, test runner, packaging
   1. meson: ???
