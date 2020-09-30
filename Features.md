Trying to harvest the existing and desired features of Bear.

## Command attribute (JSON CDB)

The JSON compilation database specification gives two options to specify the compilation command. One is an array of strings (`arguments`), the other is a single string (`command`). While the `arguemnts` fits more natural to the job, this option did come later in the specification. So, tools might still expect the `command` only.

The problem with the single string, that it needs to be shell escaped. This is not impossible, but might give chance for different interpretation of it.

While early versions were emitting the `command`, since version 2.x it does only the `arguments`.

* Both format needs to be read. (since 2.3.0)
* Both format can be written. (since 3.0.0)

## Output attribute (JSON CDB)

The JSON compilation database specification mentions an optional `output` field, which names the compiler output.

* Output filed is present. (since 2.4.2)

## Append to existing (JSPN CDB)

The output can be generated only from the intercepted commands from a single run (of the build command). Or can be appended to an existing list of compilation commands.

One of the difficulties here is, how to invalidate entries if they are no longer valid. One logic can always check if the entry source file is exists or not. If not, remove it from the output. (This is the behaviour since the feature is implemented. This might not be the best thing to do, but nobody complained since.)

Another corner case of entry validation, when the build commands are changing it will generate duplicated entries. There is not known algorithm to detect if this is an intended duplicate, or it's caused by the build system change. This limitation is documented, but got ticket about it.

* Append to existing output (since 2.0.0)

## Append to update (JSON CDB)

Extending the append functionality, the update should not only insert new entries into the output. But insert them right after the command was run. (This feature was requested by ... in a ticket ... The use case is to speed up the language server indexer.)

* Update output on compilation (planed ...)

## Recognized compilers (JSON CDB)

Since the output contains compiler calls, it does matter which program will be detected as compiler. Simple cases like `clang` or `gcc` were implemented in early versions. The recognized compilers list was extended later with: not common compilers, compiler wrappers, cross compilers, etc..

* Support major compilers (since 0.3)
* Support cross compilers (since 1.4.2)
* Support fortran compilers (since 2.4.2)
* Allow to insert compilers (since 2.3.0)
* Support compiler wrappers
  * Support Open MPI wrappers (since 2.3.7)
  * Support MPICH wrappers (since 2.3.7)
  * Support `dictcc` wrappers (since 2.3.7)
  * Support `ccache` wrappers (since 2.3.7)
  * Support GNU libtool wrappers (planed ...)
  * Support CUDA wrappers (since 2.4.3)

## Filter flags (JSON CDB)

Not all compiler filter is relevant. The best example of the `-MD` preprocessor flag "family". Which might be used in a way that cause duplicate entries in the output. (Eg.: a file is compiled with and without it. The first one is used by the build system to track dependencies. The second is the real compilation.)

* Filter flags to avoid duplicate entries (since 2.1.0)

## Filter entries by source directory (JSON CDB)

Not all compilation entries are relvant. Users might want to exclude certain source files from the output. They can provide multiple directory names to exclude (or explicitly include) in the output.

* Support to exclude/include entries (since 2.4.2)

## Compiler names (JSON CDB)

Some tools are sensitive how the compiler is named in the JSON compilation database.

* Use the current compiler as is (since 2.3.12)
* Use the current compiler with full path (since 3.0.0)
* Substitute the recognized compiler with a generic one (planed ...)

## Paths (JSON CDB)

Recognize the part of the compiler call which refer to something on the filesystem and transform their values.

* Use the current values as is (planed ...)
* Try to use relative values (partially since 2.3.0, planed ...)
* Try to use absolute values (planed ...)

## Include headers (JSON CDB)

[CompDb](https://github.com/Sarcasm/compdb#generate-a-compilation-database-with-header-files) does this.

* Emit include files (planed ...)

## Include linking (JSON CDB)

Some compiler call might look linking, but it might involve compilations too.

* Include linker calls which does compilation (partially since ..., planed ...)
* Include linker calls (planed ???)

## Don't use temporary folder

The interception phase collects all command which it was able to intercept into a temporary folder. This might be problematic for some use cases. Alternatively it can use IPC to send this information to the supervisor process. (This is how before version 0.5 was.)

* Avoid to use not specified resources (planed 3.0.0)

## Support multilib builds

On intel 64 bits machines OSes are supporting to run 32 bits binaries. When a machine hosts 32 and 64 bits libraries are called multilib. Build processes can use these mixed software libraries in many different ways. (Eg.: only the compiler is a 32 bit binary. or part of the build chain is a 32 bit binary, but the compiler itself is a 64 bit binary. etc...)

* Support multilib builds (partial since 2.3.2, planed ...)

## Support multiple character encoding

Although it's not a common pattern, but some build is using non ascii characters as arguments for the compiler. This might cause problems to serialize into the output or just pass around within the process.

* Support multiple character encoding (partial since 2.3.0, planed ...)

## Support MS Windows

Have seen PR with MinGW (to use the same library preload trick), but this can be extended for other "normal" users too.

* Support MS Windows (planed ...)

## Support MacOS

Newer version of MacOS is locked down with security features. Might require to re-think the intercept mode to satisfy this.

* Support MacOS (partially since 1.0, with compiler wrappers 3.0.0)

## Support Fortran compilers (JSON CDB)

Issue #241 (since 2.4.2)

## Support ontology output

Emit the intercepted command with extra informations.

* Exit status included (planed ...)
* Time information included (planed ...)
* Parent PID included (planed ...)
* Enrich the commands with classifications (planed ...)