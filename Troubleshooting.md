For many of problems, the best way to solve it is to build some understanding how Bear works. This section will help to get familiar with Bear's internals and also advice what to check in certain error cases.

TBD

# The build with and without Bear behaves differently

TBD

# The output is empty

The most common cause for empty outputs is that the build command did not
execute any commands. The reason for that could be, because incremental builds
not running the compilers if everything is up-to-date. Remember, Bear does not
understand the build file (eg.: makefile), but intercepts the executed
commands.

The other common cause for empty output is that the build has a "configure"
step, which captures the compiler to build the project. In case of Bear is
using the _wrapper_ mode (read `intercept` man page), it needs to run the
configure step with Bear too (and discard that output), before run the build
with Bear.

# The output is missing entries.

TBD

# The output has duplicate entries.

TBD

# The output has entries, which are no longer part of the project.

TBD
