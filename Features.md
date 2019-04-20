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

## Filter flags (JSON CDB)

Not all compiler filter is relevant. The best example of the `-MD` preprocessor flag "family". Which might be used in a way that cause duplicate entries in the output. (Eg.: a file is compiled with and without it. The first one is used by the build system to track dependencies. The second is the real compilation.)

* Filter flags to avoid duplicate entries (since ...)

## Compiler names (JSON CDB)

Some tools are sensitive how the compiler is named in the JSON compilation database.

* Use the current compiler as is (since ...)
* Use the current compiler with full path (planed ...)
* Substitute the recognized compiler with a generic one (planed ...)

## Paths (JSON CDB)

Recognize the part of the compiler call which refer to something on the filesystem and transform their values.

* Use the current values as is (planed ...)
* Try to use relative values (partially since ..., planed ...)
* Try to use absolute values (planed ...)

## Include headers (JSON CDB)

[CompDb](https://github.com/Sarcasm/compdb#generate-a-compilation-database-with-header-files) does this.

* Emit include files (planed ...)

## Include linking (JSON CDB)

Some compiler call might look linking, but it might involve compilations too.

* Include linker calls which does compilation (partially since ..., planed ...)
* Include linker calls (planed ???)

## Don't use temporary folder

The interception phase collects all command which it was able to intercept into a temporary folder. This might be problematic for some use cases. Alternatively it can use IPC to send this information to the supervisor process. (This is how version 1.x was doing.)

* Avoid to use not specified resources (planed ...)

## Support MS Windows

Have seen PR with MinGW (to use the same library preload trick), but this can be extended for other "normal" users too.

* Support MS Windows (planed ...)

## Support MaxOS

Newer version of MacOS is locked down with security features. Might require to re-think the intercept mode to satisfy this.

* Support MacOS (partially since 1.0, planed ...)

## Support Fortran compilers (JSON CDB)

Issue #241 (planed ...)