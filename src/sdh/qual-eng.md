% SPDX-License-Identifier: CC-BY-SA-4.0

% Copyright (C) 2026 embedded brains GmbH & Co. KG

# Qualification engineering

## Overview

Usually your task is to pre-qualify a group of already existing functions –
such as all functions related to semaphores or all functions related to events.
At the very core, you have to write an interface specification for each
publicly visible function or macro, write the requirements that each such
function or macro must meet, write validation tests that verify each function
or macro meets all its requirements and make sure code coverage goals are
reached. The typical workflow looks like this (the steps are described in more
detail later):

1. Create a new feature branch for your work.

2. Put all functions and macros visible in the API in Doxygen groups.

3. Add all functions and code needed to the pre-qualified API subset.

4. Create a specification directory tree for this function group. In the next
   steps, you will create YAML files in these directories. These YAML files
   will contain the function specification, the requirements and the validation
   tests.

5. Write or generate interface specifications. The result is a YAML file for
   each publicly visible API function or macro describing the function, its
   parameters and the result. These YAML files can often be generated from the
   existing Doxygen information found in the C header files. See
   {ref}`InterfaceItems`.

6. For each API function or macro:

   6.1 Write down all requirements the function or macro must meet. The result
   is a YAML file for each publicly visible API function or macro containing
   all its requirements.

   6.2 Write validation tests. These tests come in two flavours:

   - **Simple Validation Tests** These consist of unstructured test functions
     and are written in an extra YAML file. These are appropriate for macros
     and functions that do a simple computation

   - **Action Requirements** These consist of pre- and post-conditions and a
     table relating the conditions to each other. The whole and often extensive
     test logic is automatically generated for you. See
     {ref}`ActionRequirements`.

   6.3 Generate C- and doc-files from the YAML files written above. You must
   add the generated tests to an existing test suite or create a new test suite
   for them once.

   6.4 Compile and run the tests

   6.5 Debug and fix test failures

   6.6 View coverage data

7. Create a pull request once your work is completed, tests and coverage goals
   are reached.

Finally, performance and memory benchmarks may be needed.

(QualEngOverviewExamples)=

```{admonition} Examples
Examples of already pre-qualified function groups are often helpful.

[**Base Definitions**](https://docs.rtems.org/doxygen/main/group__RTEMSAPIBaseDefs.html)
define commonly used macros. This is a good example for writing
simple requirements and simple validation tests.

* `cpukit/include/rtems/score/basedefs.h` – header file

* `spec/rtems/basedefs` – specification directory tree contains all the
  YAML files created for pre-qualification

* `spec/rtems/basedefs/if/array-size.yml` – interface specification of
  the `RTEMS_ARRAY_SIZE` macro, see {ref}`InterfaceItems` for how to write
  such a file

* `spec/rtems/basedefs/req/array-size-0.yml` – requirement of the
  `RTEMS_ARRAY_SIZE` macro, note that some macros require several
  requirements

* `spec/rtems/basedefs/val/basedefs.yml` – defines many simple validation
  tests. The one for the `RTEMS_ARRAY_SIZE` macro starts at line 231.
  Note that `$${../if/array-size:/name}` is a substitution, it substitutes
  the value of the key `name` from the file `../if/array-size.yml` which
  happens to be `RTEMS_ARRAY_SIZE`. `$${.:/step}` is substituted by a number
  counting the `T_step_*` check macros in the C file generated from this
  YAML file.

* `testsuites/validation/tc-basedefs.c`  – the corresponding generated
  validation test file

* `spec/build/testsuites/validation/validation-no-clock-0.yml` – the YAML
  file which defines the test suite to which the validation test belongs

* `testsuites/validation/ts-validation-no-clock-0.c` – main C file of the
  test suite, see also `testsuites/validation/ts-default.h`

* `spec/testsuites/validation-no-clock-0.yml` – definition of the test
  suite file `ts-validation-no-clock-0.c` itself

* `spec/build/testsuites/validation/grp.yml` – this group defines which
  test suites exist.

[**Rate Monotonic Manager**](https://docs.rtems.org/docs/main/c-user/rate-monotonic/index.html#rate-monotonic-manager)
from the Classic API executes a job periodically. This is a good example
for action requirements.

* `cpukit/include/rtems/rtems/ratemon.h` – header file

* `cpukit/rtems/src/ratemoncreate.c`, `cpukit/rtems/src/ratemoncancel.c`,
  `cpukit/rtems/src/ratemongetstatus.c` – some of its implementation files
  (note that the validation tests must cover the code in further files
  to reach the code coverage goals)

* `spec/rtems/ratemon` – specification directory tree contains all the
  YAML files created for pre-qualification

* `spec/rtems/ratemon/req/create.yml` – one of its action requirement
  YAML files; {ref}`ActionRequirements` builds the closely analogous
  `spec/rtems/timer/req/create.yml` from scratch as a worked example

* `testsuites/validation/tc-ratemon-create.c` – the corresponding
  generated validation test file

* `spec/build/testsuites/validation/validation-no-clock-0.yml` – the YAML
  file which defines the test suite to which the validation test belongs.
  This is the same as the one of the Base Definitions example above.
```

## Create a feature branch

A feature branch to accumulate all your changes is typically created like this:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ git checkout -b $${my-new-branch-name} eb/next
```

```{raw} latex
\end{footnotesize}
```

## Add Doxygen groups

A [Doxygen group](https://www.doxygen.nl/manual/grouping.html) collects a set
of related declarations under one identifier, independent of the files in which
they live. RTEMS uses two such groups per manager, or per part of a manager: an
*API group* for its publicly visible functions and macros, and an
*implementation group* for the internal data structures and helper functions
behind them. Groups nest through `@ingroup`, so an implementation group points
at its parent implementation group and an API group points at its parent API
group, mirroring the software architecture down to the Classic API, POSIX API,
or Score component level.

Doxygen groups matter for qualification because the group a source element
belongs to, together with a link into the generated Doxygen
@{/glossary/html:/term} output (the @{/glossary/sdd:/term}), is recorded in a
Doxygen *tagfile*. That tagfile is what lets the traceability matrices in the
@{/glossary/icd:/term} and the @{/glossary/srs:/term} be built and checked
automatically, complete with working links into the @{/glossary/sdd:/term}. A
function or macro that is not in a group cannot be traced this way, so it
cannot be pre-qualified.

Before adding anything new, check whether suitable groups already exist for the
functions and macros you are pre-qualifying; most Classic and POSIX API
managers already have both. The Rate Monotonic Manager is a complete example to
study:

- `cpukit/include/rtems/rtems/ratemon.h` defines the API group once with
  `@defgroup RTEMSAPIClassicRatemon`, then tags each public function and type
  individually with `@ingroup RTEMSAPIClassicRatemon`.

- `cpukit/include/rtems/rtems/ratemonimpl.h` defines the implementation group
  with `@defgroup RTEMSImplClassicRateMonotonic`, and wraps every declaration
  that belongs to it between `@@{` and `@}` so the declarations do not each
  need their own `@ingroup`:

  ```{raw} latex
  \begin{footnotesize}
  ```

  ```{code-block} c
  ---
  linenos:
  ---
  /**
   * @defgroup RTEMSImplClassicRateMonotonic Rate Monotonic Manager
   *
   * @ingroup RTEMSImplClassic
   *
   * @brief This group contains the Rate Monotonic Manager implementation.
   *
   * @@{
   */

  /* ... declarations belonging to the group ... */

  /**@}*/
  ```

  ```{raw} latex
  \end{footnotesize}
  ```

- Every `.c` file that implements part of the manager, for example
  `cpukit/rtems/src/ratemoncreate.c`, cannot be nested inside that `@@{ @}`
  block, so it tags itself with `@ingroup RTEMSImplClassicRateMonotonic` in its
  own `@file` doc comment instead.

If no group exists yet for your functions, add one the same way: pick an
identifier that follows the pattern of its neighbours (`RTEMSAPI...` for an API
group, `RTEMSImpl...` for an implementation group), give it a `@brief`, and
`@ingroup` it under the closest existing parent group. Use `@ref <GroupID>` to
refer to a group from surrounding prose, as `ratemonimpl.h` does in its own
file brief.

Choose the identifier carefully: a later step in this workflow creates a
requirement item that names the implementation group as its `identifier`, for
example `spec/rtems/ratemon/req/group.yml` for `RTEMSImplClassicRateMonotonic`.
That item must reuse the exact identifier chosen here.

## Add a function to the pre-qualified subset

Whether a `.c` or `.h` file is compiled into the pre-qualified subset or into
full, unqualified RTEMS is decided entirely by which specification item lists
it, not by anything in the file itself. The `RTEMS_QUAL` build option selects
between the two, set to `True` in the BSP variant's section (for example,
`[sparc/leon3]`) of the `config.ini` file passed to `./waf configure`: when it
is enabled, only files that are listed unconditionally are built; when it is
disabled, those files are built together with everything else. A file listed in
an item whose `enabled-by` attribute negates `RTEMS_QUAL` is therefore only
built for full, unqualified RTEMS and is not, or not yet, part of the
pre-qualified subset; a file listed without that condition is always built and
is part of it.

For `cpukit`, this split is visible in a pair of specification items:

- `spec/build/cpukit/librtemscpu.yml` defines the content of the library
  `librtemscpu.a`, and lists, unconditionally, the source and header files that
  belong to the pre-qualified subset. It also links, as `build-dependency`, to
  a number of smaller items for optional features that contribute further
  object files to the same library; some of those are unconditional too, and
  some are split the same way as `cpukit` itself.

- `spec/build/cpukit/objextra.yml` lists everything not, or not yet, part of
  the pre-qualified subset, guarded by an `enabled-by` attribute that negates
  `RTEMS_QUAL`. A feature can keep its own, smaller such item instead of using
  `objextra.yml` directly; for example, `spec/build/cpukit/objsmpextra.yml`
  holds the SMP support files that are not yet pre-qualified, guarded the same
  way, and is a `build-dependency` of `spec/build/cpukit/objsmp.yml`, which
  holds the SMP support files that already are.

Since a publicly visible RTEMS function usually has its own `.c` file, adding
it to the pre-qualified subset is usually just moving its one `source:` entry
from `objextra.yml`, or the matching feature-specific item, to
`librtemscpu.yml`, keeping the list in alphabetical order like its neighbours.
If the function's declaration is not installed unconditionally yet either, move
the matching `install:` entry the same way. For example,
`cpukit/rtems/src/taskinitusers.c` moved from `objextra.yml`'s `source:` list
to `librtemscpu.yml`'s in a single-line change.

Two complications can turn this into more than a one-line move:

1. **A file mixes pre-qualified and not-yet-pre-qualified code.** A whole file
   can only be built or excluded as a unit, so such a file must be split, or
   handled some other way, before any of it can be moved. If in doubt, ask us
   (embedded brains).

2. **The function calls into a file that is not yet in the subset.** This
   typically happens when an API function in `cpukit/rtems/src` calls a helper
   function implemented in `cpukit/score`, the Score (Super Core), that has not
   been pre-qualified yet. In that case, move all of the files involved
   together, and all of them must eventually be pre-qualified as well. For
   example, the Rate Monotonic Manager statistics functions
   `cpukit/rtems/src/ratemongetstatistics.c`,
   `cpukit/rtems/src/ratemonreportstatistics.c`,
   `cpukit/rtems/src/ratemonresetall.c`, and
   `cpukit/rtems/src/ratemonresetstatistics.c` moved into `librtemscpu.yml`
   together with `cpukit/score/src/timespecdividebyinteger.c`, since
   `ratemonreportstatistics.c` calls `_Timespec_Divide_by_integer()`.

To find out where a `.c` or `.h` file currently lives, search for its name
across the whole build specification:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ grep -rl taskinitusers.c spec/build
```

```{raw} latex
\end{footnotesize}
```

## Create a specification directory tree

For the function group you are pre-qualifying, create a specification directory
tree of `.yml` files under `spec/rtems`, named after the group, for example
`spec/rtems/ratemon` for the Rate Monotonic Manager used as the running example
throughout this chapter. At this point, only create the next directory level
down, as empty directories; the sections that follow fill them with content.

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ mkdir -p spec/rtems/$${my-group}/if spec/rtems/$${my-group}/req \
    spec/rtems/$${my-group}/val
```

```{raw} latex
\end{footnotesize}
```

Only `if/`, `req/`, and `val/` are mandatory: every group needs at least an
interface, a requirement, and a validation test. `constraint/` and `glossary/`
are optional; add them only when you need them.

````{admonition} Directory tree of a fully pre-qualified group
`spec/rtems/ratemon` already went through every step of this workflow.
Shortened to a few representative files, it looks like this:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
spec/rtems/ratemon/
├── constraint/
│   └── max.yml
├── glossary/
│   ├── job.yml
│   ├── ownertask.yml
│   └── ... (14 more terms)
├── if/
│   ├── header.yml
│   ├── create.yml
│   ├── period.yml
│   └── ... (18 more interface items)
├── req/
│   ├── group.yml
│   ├── create.yml
│   ├── ident.yml
│   ├── timeout.yml
│   └── ... (11 more requirements)
└── val/
    ├── ident.yml
    ├── mem-period.yml
    ├── mem-period-del.yml
    └── ratemon.yml
```

```{raw} latex
\end{footnotesize}
```
````

- **`if/`** holds interface items: one YAML file per publicly visible function,
  macro, type, or enumerator, plus one for the header file itself. See
  {ref}`InterfaceItems`.

- **`req/`** holds requirement items, see {ref}`QualEngWriteRequirements`. Most
  are *action requirements*: a requirement broken into pre-conditions and
  post-conditions with a transition-map relating them, from which the C test
  file and most of its test logic are generated automatically; see
  {ref}`ActionRequirements`. A requirement can instead be a short, simple
  requirement text whose test is written by hand in `val/`, or a non-functional
  requirement, like the Doxygen implementation group requirement added earlier
  in this chapter or a memory-usage benchmark.

- **`val/`** holds the hand-written tests for simple requirements and the
  memory-usage benchmarks for non-functional requirements in `req/`. Action
  requirements do not need a file here: their test code already lives inside
  the `req/` item itself and is generated from it. See
  {ref}`QualEngWriteSimpleValidationTests`.

- **`constraint/`** holds short, reusable statements of a usage constraint, for
  example a configurable maximum, that an interface item in `if/` can reference
  through a `constraint` link.

- **`glossary/`** holds short definitions of terms specific to this group,
  referenced from `req/` and other texts the same way as the project-wide
  glossary, for example `$${../glossary/job:/term}`.

## Create interface specifications

See {ref}`InterfaceItems`, which walks through writing an interface
specification item from scratch.

(QualEngWriteRequirements)=

## Write requirements

See the
[*RTEMS Software Engineering Manual* chapter *Software Requirements Engineering*](https://docs.rtems.org/docs/main/eng/req/index.html#software-requirements-engineering).

We use the
[Easy Approach to Requirements Syntax (EARS)](https://docs.rtems.org/docs/main/eng/req/req-for-req.html#syntax)
to express requirements. This is fine for simple requirements. Yet, because
action requirements are broken into pre-conditions and post-conditions, the
requirement texts are also broken apart. The *when*, *while*, *if*, *where*
parts appear in the pre-conditions while the *\<system name> shall \<system
response>* parts appear in the post-conditions. See {ref}`ActionRequirements`
for the concrete YAML mechanics behind this split.

(QualEngWhereTextAppears)=

```{admonition} Where does the text written in those YAML files appear?

The texts from the interface specifications appear in the generated
RTEMS header files, in the *RTEMS Classic API Guide* or the *RTEMS POSIX
API Guide*, as well as in the *RTEMS Doxygen*. In a package, they also appear
in the *Interface Control Document (ICD)*. {ref}`InterfaceItems` covers how to
write these YAML files.

The texts from the requirements and the description of the tests are part
of *Test Reports (TR), Software Requirements Specification (SRS), Software
Unit and Integration Test Plan (SUITP), Software Validation Specification
(SVS)*. They also appear as comments in the generated C files.
{ref}`ActionRequirements` covers how to write the requirement YAML files for
the action requirement case.
```

(QualEngWriteSimpleValidationTests)=

## Write simple validation tests

Follow the examples. Each test has an *action* part which contains the C code
to exercise the function or macro and a *check* part which should check for the
expected result(s). Each of these sections has a brief description. Moreover,
each such *check* part has a *links* part which links the check to one or
several requirements. In the end, all requirements must have at least one test.

Note that at the bottom of such a YAML file, are keys like

- `test-brief` to describe the whole test case,
- `test-includes` to define header files to be included,
- `test-support` to contain code which is put at the top of the generated C
  file, and
- `test-target` the name of the C file to be generated.

Consider also that newly generated C test case files must be added to a test
suite in a YAML file (see the examples above).

The test cases are based on the RTEMS test framework as described in the
[*RTEMS Software Engineering Manual* chapter *Software Test Framework*](https://docs.rtems.org/docs/main/eng/test-framework.html#software-test-framework).
This permits the use of test fixtures. It is helpful to have a look at the
*Test Checks* and the *Log Message* in that chapter.

The RTEMS coding rules apply, see
[*RTEMS Software Engineering Manual* chapter *Coding Standards*](https://docs.rtems.org/docs/main/eng/coding.html#coding-standards).

```{admonition} Unit vs. validation tests
Most projects have both *unit tests* and *validation tests*, and use them for
different purposes.

Unit tests are white box, or glass box, tests: the programmer who wrote the
code under test can see its source and writes the test to match. They check
that the code does what the programmer intended, not what a requirement
says, so requirements play no role in them. Modules are usually tested in
isolation with the help of mocks, and unit tests are also the usual means to
reach code coverage goals.

Validation tests are black box tests: written by someone who was not
involved in producing the code under test and does not look at its source,
strictly against the requirements, with one or several tests for each
requirement. This checks that the code actually implements the
requirements, and that the requirements were not misinterpreted by the
original programmer.

RTEMS pre-qualification has almost only validation tests instead. They
still strictly check the requirements, but they are white box tests like
unit tests, and, since RTEMS has no unit tests, they are also what has to
reach the code coverage goals, see {ref}`CreateCoverageReport`. With rare
exceptions, they exercise the public API only, for the following reasons:

- The RTEMS project has always tested at the API level, even for tests
  that target an internal implementation function.
- RTEMS traditionally has no mocks for tests and is not prepared to
  support them.
- For an operating system, testing every unit in isolation runs into
  difficulties: some units change the state of the processor at the
  register level, which makes mocking difficult.
- RTEMS has no deep function call hierarchies, and internal states are
  usually independent of each other, so most code can be reached through
  the API alone.
- Without unit tests, the validation tests are what has to meet the code
  coverage goals.

Some validation tests directly manipulate RTEMS internal data structures to
set up the scenario a requirement describes. This does not turn them into
unit tests: the check itself still goes through the public API, against the
requirement, not against the implementation.
```

## Write action requirements

See {ref}`ActionRequirements`, which walks through writing an action
requirement from scratch, including the pre-condition/post-condition/
transition-map mechanics only summarized above.

(QualEngGenerateCCode)=

## Generate C code from the YAML files

See the description of the `Makefile.work` in chapter
{ref}`QualEngEnvironment`. Building the workspace application with `make`
invokes the `specwareexport` tool internally, see {ref}`ToolSpecwareexport`.

## Add test cases to test suites

A generated validation test case is not built or run until its `.c` file is
added to the `source:` list of a test-program build item under
`spec/build/testsuites/validation`; see the
{ref}`examples in the overview <QualEngOverviewExamples>`. For example, after
working through {ref}`ActionRequirements`,
`testsuites/validation/tc-timer-create.c` exists but is not yet part of any
suite. Adding it to the `source:` list of
`spec/build/testsuites/validation/validation-no-clock-0.yml`, next to the other
`tc-timer-*.c` entries, is what actually compiles and runs it:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
- testsuites/validation/tc-timer-create.c
```

```{raw} latex
\end{footnotesize}
```

```{admonition} Selecting the right Validation Test Suite
Not all test cases of a function group need to be part of the same test
suite. It depends on the requirements a particular test has on the
executable. For example, a test may require the usual clock ticks, others
may require no clock to "manually" trigger the clock tick as part of the test
execution, others may need exactly one CPU, others at least three.
```

## Create a test suite

Create a new test suite only when none of the existing ones give the test
executable the resources, or restrictions, it needs, see the admonition above.
A test suite is defined by a pair of specification items that share the same
name, for example `validation-one-cpu-0`:

- `spec/testsuites/$${my-suite}.yml` (`type: test-suite`) generates the suite's
  main C file, named by its `test-target` attribute,
  `testsuites/validation/ts-$${my-suite}.c`. Its `test-code` attribute is a
  short `main`-like body: a handful of `#define CONFIGURE_...` options followed
  by `#include "ts-default.h"`. For example,
  `spec/testsuites/validation-no-clock-0.yml` defines
  `CONFIGURE_APPLICATION_DOES_NOT_NEED_CLOCK_DRIVER`, while
  `spec/testsuites/validation-one-cpu-0.yml` instead limits
  `CONFIGURE_MAXIMUM_PROCESSORS` to `1`; the two items differ only in these
  `#define` lines.

- `spec/build/testsuites/validation/$${my-suite}.yml`
  (`build-type: test-program`) is the matching build item. Its `source:` list
  starts with only the generated `ts-$${my-suite}.c`; add test case files to it
  the same way as in the previous section. Its `target` attribute names the
  executable, `testsuites/validation/ts-$${my-suite}.exe`.

The easiest way to create both is to copy an existing pair, then adjust the
copies:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ cp spec/testsuites/validation-no-clock-0.yml \
    spec/testsuites/$${my-suite}.yml
$ cp spec/build/testsuites/validation/validation-no-clock-0.yml \
    spec/build/testsuites/validation/$${my-suite}.yml
```

```{raw} latex
\end{footnotesize}
```

Adjust the `test-brief`, `test-description`, and `#define` lines in
`spec/testsuites/$${my-suite}.yml`, and clear the `source:` list in
`spec/build/testsuites/validation/$${my-suite}.yml` down to only the generated
`ts-$${my-suite}.c`.

A new suite is not built until it is linked in as a `build-dependency` of
`spec/build/testsuites/validation/grp.yml`:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
- role: build-dependency
  uid: $${my-suite}
```

```{raw} latex
\end{footnotesize}
```

## Compile and run the tests

See section {ref}`CreateWorkspace`.

## Run a test suite

See section {ref}`RunWorkspaceApplication`.

## Debug tests

See section {ref}`DebugWorkspaceApplication`.

## Generate and view code coverage

See section {ref}`CreateCoverageReport`.

## Create a pull request

Make sure everything compiles and links, the tests are running without
reporting failures and the code coverage goals are reached. Moreover, make sure
you have committed all your changes.

1. **Push your branch**

   Push your feature branch to your personal fork of the embedded brains RTEMS
   repository:

   ```{raw} latex
   \begin{footnotesize}
   ```

   ```{code-block} none
   ---
   linenos:
   ---
   $ git push -u origin $${my-new-branch-name}
   ```

   ```{raw} latex
   \end{footnotesize}
   ```

   This will create a copy of your feature branch at your personal repository
   fork and set your feature branch to track that branch.

2. **Create a pull request**

   Log in to your GitHub account and go to your fork of the repository. Select
   your feature branch. GitHub will offer to create a *pull request* for it. As
   the target branch in the embedded brains RTEMS repository, select `eb/next`.
   Give your pull request a name and add a short description. It is advisable
   to set the pull request to *draft* state initially. Then click on *Create
   pull request*.

3. **Check and fix errors**

   You should see your new pull request now. If not, navigate to the
   [embedded brains RTEMS repository on GitHub](https://github.com/embedded-brains/rtems),
   choose the *Pull requests* tab, and select your pull request.

   Wait until all CI jobs are completed. If there are any errors, please fix
   them.

   Wait until *GitHub Copilot* finishes its review (you will get an email). If
   the review does not start automatically, *GitHub Copilot* appears in the
   list of possible reviewers on the right side. Click the small circle icon
   next to it.

   Check all its review comments, fix the relevant ones and close the review
   comments you have examined.

4. **Push your fixes**

   Assuming you have made fixes in your local working tree and have committed
   them, push your fixes again to your local fork of the repository. This will
   automatically update the pull request:

   ```{raw} latex
   \begin{footnotesize}
   ```

   ```{code-block} none
   ---
   linenos:
   ---
   $ git push
   ```

   ```{raw} latex
   \end{footnotesize}
   ```

   If needed, use the `--force` option.

   Afterwards, repeat step 3 until all issues are resolved.

5. **Have your pull request merged**

   Once all issues found by the CI jobs and *GitHub Copilot* are resolved,
   remove the *draft* status. The embedded brains staff will review your pull
   request and merge it.
