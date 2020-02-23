Read from bottom up if you want to get full picture how it works. Read normally if you want to know what's coming.

# 3.x Build semantics (feasibility check)

Summary:

* Written in: rust
* Build system: cargo

Bear produces ontology as output. While compilation databases are good for tools which are based on Clang tooling library, found that this is not the only usage scenario. People often use this tool to get insights into their product build process. (Especially when the project is big or complex.)

These use cases could be:

* To see all executed commands in a project, in order to:
  * Find duplicated tasks.
  * Find where time is spent.
  * Find bugs in the process.
* To help to migrate from one build tool to another.
* To build cross project indexes:
  * of symbols defined in projects,
  * to see symbols usages in cross projects.

# 3.0 Bear in C++ (in progress)

Summary:

* Written in C++14 dialect (or C++17) and Python
* Build system: CMake

Major change the `libear` library does not send the report itself, and does not do anything with the environment variables. Instead, it execute a wrapper process `pear` which does all these. The motivation behind it is to reduce the complexity of `libear`. (Do not allocate memory during the exec calls, because it might be not safe. Do not try to encode the received parameters, remove potential failures. Does not use any symbol from any libraries except the dynamic loader library.)

The process `pear` is a statically linked executable which supervise the child process. (Static linking is relevant, that ensures that it won't call itself.) It gives more room to implement the following features:

* Can report on process exit status.
* Statically linked compilers can be recorded.
* Encoding problems can be solved.
* Multi threading issues can be solved.
* Tools which manipulating the `LD_PRELOAD` environment can work.
* Using compiler wrappers is easy with this, which solves issues on OSX and Windows.

# 2.x Polyglot Bear

Summary:

* Written in C89 and Python 2.7 and 3.x.
* Build system: CMake

The major change here is to rewrite the `bear` process in Python. The built in JSON module and the better file system handling modules makes the code slim. It also improves portability.

It also drops the socket connection and uses temporary files instead. Which helps to deal with big builds. But the main drive behind it is: programming bugs for long running builds are not fatal. (The recorded executions are on the file system not in the memory.)

# 1.x Bear in C

Summary:

* Written in C98.
* Build system: CMake.
* Dependency: `libconfig`.

The core concept is to use the operating system dynamic loader [preload](https://en.wikipedia.org/wiki/Dynamic_linker#Systems_using_ELF) feature to hijack C function calls. The relevant process execution methods are implemented in a shared library called `libear`. The loading of this library is to modify the default behaviour of the dynamic loader, by setting the `LD_PRELOAD` environment variable to the `libear` library.

The content of the `libear` library is the implementation of the [exec system calls](https://en.wikipedia.org/wiki/Exec_(system_call)). In this version the implementation is:

* Open a connection to a known destination.
  * Reports the `exec` call details (PID, parent PID, the command, the current working directory).
* Take a reference to the system `exec` call,
  * modify the environment variables (to ensure that child process will also load this library),
  * and call the system `exec` function with the original parameters.

The connected process which collects the execution reports is called `bear`. This is the build wrapper which user has to call explicitly. The logic of this process is:

* Prepare environment variables (to ensure that child process will load the `libear` library).
* Execute the given command,
  * open a (UNIX domain socket) listener to accept connections,
  * accept connections and record the payload into memory,
  * signal about the child termination breaks the accept loop.
* Post process the recorded execution messages.
  * Filters out non compiler invocations,
  * classifies compiler call parameters,
  * takes the source file from the argument list of a compiler call,
  * encode the output into a JSON file (as the compilation database specifies).
