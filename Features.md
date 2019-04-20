Trying to harvest the existing and desired features of Bear.

## Command attribute (JSON CDB)

The JSON compilation database specification gives two options to specify the compilation command. One is an array of strings (`arguments`), the other is a single string (`command`). While the `arguemnts` fits more natural to the job, this option did come later in the specification. So, tools might still expect the `command` only.

The problem with the single string, that it needs to be shell escaped. This is not impossible, but might give chance for different interpretation of it.

While early versions were emitting the `command`, since version 2.x it does only the `arguments`.

* Both format needs to be read. (since ...)
* Both format can be written. (plan 3.x)

## Output attribute (JSON CDB)

The JSON compilation database specification mentions an optional `output` field, which names the compiler output.

* Output filed is present. (plan 3.x)

## Append to existing (JSPN CDB)

The output can be generated only from the intercepted commands from a single run (of the build command). Or can be appended to an existing list of compilation commands.

One of the difficulties here is, how to invalidate entries if they are no longer valid. One logic can always check if the entry source file is exists or not. If not, remove it from the output. (This is the behaviour since the feature is implemented. This might not be the best thing to do, but nobody complained since.)

Another corner case of entry validation, when the build commands are changing it will generate duplicated entries. There is not known algorithm to detect if this is an intended duplicate, or it's caused by the build system change. This limitation is documented, but got ticket about it.

* Append to existing output (since ...)

## Append to update (JSON CDB)

Extending the append functionality, the update should not only insert new entries into the output. But insert them right after the command was run. (This feature was requested by ... in a ticket ... The use case is to speed up the language server indexer.)

* Update output on compilation (planed ...)

## Recognized compilers (JSON CDB)

Since the output contains compiler calls, it does matter which program will be detected as compiler. Simple cases like `clang` or `gcc` were implemented in early versions. The recognized compilers list was extended later with: not common compilers, compiler wrappers, cross compilers, etc..

* Support major compilers (since ...)
* Support cross compilers (since ...)
* Allow to insert compilers (since ...)
* Support compiler wrappers
  * Support Open MPI wrappers (since ...)
  * Support MPICH wrappers (since ...)
  * Support `dictcc` wrappers (since ...)
  * Support `ccatch` wrappers (since ...)
  * Support GNU libtool wrappers (planed ...)
  * Support CUDA wrappers (planed ...)
