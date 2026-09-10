% SPDX-License-Identifier: CC-BY-SA-4.0

% Copyright (C) 2026 embedded brains GmbH & Co. KG

(ActionRequirements)=

# Action requirements

This chapter assumes you have read {ref}`SpecificationItems` and
{ref}`InterfaceItems`. It continues the `rtems_timer_create()` example from
{ref}`InterfaceItems` to build the matching *functional specification*: a
precise, testable statement of what the directive does for every possible
combination of inputs and system state.

## What is an action requirement?

An interface item (the previous chapter) says *what a function is*: its name,
parameters, and the shape of its return value. It does not yet say *what the
function does*. That is the job of an **action requirement**. Where the text it
produces ends up (Test Reports, the Software Requirements Specification, the
Software Unit and Integration Test Plan, the Software Validation Specification,
and generated C-file comments) is covered in
{ref}`where the generated text ends up <QualEngWhereTextAppears>`.

An action requirement models a directive as a small, semi-formal finite state
machine:

- **Pre-conditions**: every input, and every piece of system state, that can
  affect the outcome. Each pre-condition has a name and a list of possible
  *states*.
- **A trigger action**: the C statement that calls the function under test.
- **Post-conditions**: every observable effect of the call. Each post-condition
  also has a name and a list of possible states.
- **A transition-map**: the rules that say, for each combination of
  pre-condition states, which state each post-condition ends up in.

This is not test code with extra paperwork attached. The pre-condition and
post-condition state *names* are also used to generate human-readable
requirement text (each state carries a `text` field written as a requirement:
"While ... " for pre-conditions, "The ... shall be ..." for post-conditions).
The transition-map is simultaneously the formal definition of the behavior
*and* the input that drives generated test code covering every reachable
combination. Get the pre-conditions and post-conditions right, and both the
prose specification and the test coverage follow from the same source.

An action requirement item lives in a `req/` directory that is a sibling of the
interface item's `if/` directory. For `rtems_timer_create()`
(`spec/rtems/timer/if/create.yml`), the action requirement goes into
`spec/rtems/timer/req/create.yml`, UID `/rtems/timer/req/create`.

```{figure} ../images/action-requirement-workflow.*
---
alt: Overview of an action requirement's properties and activities
width: 70%
---
Action requirement overview
```

## Before you start

You need the same local checkout as in {ref}`InterfaceItems`, plus the RTEMS
implementation sources (`cpukit/`) and the validation test suite sources
(`testsuites/validation/`) so that you can read the actual C implementation you
are specifying.

## Step 1: scaffold the requirement

Every `req/*.yml` file has the same overall shape, so start from a minimal
skeleton rather than inventing the structure from scratch: one placeholder
pre-condition per parameter, and a placeholder `Status` post-condition already
listing the interface item's discrete return values. For `rtems_timer_create()`
(two parameters, `name` and `id`; four return values, `Ok` / `InvName` /
`InvAddr` / `TooMany`), the skeleton at `spec/rtems/timer/req/create.yml` looks
like this:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
SPDX-License-Identifier: CC-BY-SA-4.0 OR BSD-2-Clause
copyrights:
- Copyright (C) <year> <your name or organization>
enabled-by: true
functional-type: action
links:
- role: interface-function
  uid: ../if/create
pre-conditions:
- name: Name
  states:
  - name: Valid
    test-code: |
      /* TODO */
    text: |
      /* TODO: While the name parameter is ... */
  test-epilogue: null
  test-prologue: null
- name: Id
  states:
  - name: Valid
    test-code: |
      /* TODO */
    text: |
      /* TODO: While the id parameter is ... */
  test-epilogue: null
  test-prologue: null
post-conditions:
- name: Status
  states:
  - name: Ok
    test-code: |
      /* TODO */
    text: |
      /* TODO */
  - name: InvName
    test-code: |
      /* TODO */
    text: |
      /* TODO */
  - name: InvAddr
    test-code: |
      /* TODO */
    text: |
      /* TODO */
  - name: TooMany
    test-code: |
      /* TODO */
    text: |
      /* TODO */
  test-epilogue: null
  test-prologue: null
rationale: null
references: []
requirement-type: functional
skip-reasons: {}
test-action: |
  /* TODO: call $${../if/create:/name}() */
test-brief: null
test-cleanup: null
test-context: []
test-context-support: null
test-description: null
test-header: null
test-includes:
- rtems.h
test-local-includes:
- tx-support.h
test-prepare: null
test-setup:
  brief: null
  code: null
  description: null
test-stop: null
test-support: null
test-target: testsuites/validation/tc-timer-create.c
test-teardown: null
text: $${.:text-template}
transition-map: []
type: requirement
```

```{raw} latex
\end{footnotesize}
```

Every attribute is present from the start, even where the value is only a
placeholder or `null` -- the schema requires them. `test-target` is the C test
source file that will hold the generated test code; it does not need to exist
yet. The one placeholder state per pre-condition and the one state per known
return value are not yet the real states -- those come from Steps 3 and 4
below. Everything from here on is about turning this skeleton into a correct
specification.

(ActionRequirementsStep2)=

## Step 2: read the implementation

A directive's pre-conditions and post-conditions come from two sources. Get all
of them right. Full code coverage depends on it: a missing pre-condition or
post-condition state leaves part of the implementation untested. The
documentation gives a first, partial set of pre-conditions and post-conditions.
The implementation then completes that set.

Look at the header file, the generated documentation, or the interface item
before you open the implementation. The header file
`cpukit/include/rtems/rtems/timer.h` declares `rtems_timer_create()` as
follows, together with its Doxygen comment. See the
[RTEMS license](https://www.rtems.org/license/) for the terms that apply to it,
and to every other RTEMS source quoted in this section.

```{raw} latex
\begin{footnotesize}
```

```{code-block} c
---
linenos:
---
/**
 * @ingroup RTEMSAPIClassicTimer
 *
 * @brief Creates a timer.
 *
 * @param name is the object name of the timer.
 *
 * @param[out] id is the pointer to an ::rtems_id object.  When the directive
 *   call is successful, the identifier of the created timer will be stored in
 *   this object.
 *
 * This directive creates a timer which resides on the local node.  The timer
 * has the user-defined object name specified in ``name``.  The assigned object
 * identifier is returned in ``id``.  This identifier is used to access the
 * timer with other timer related directives.
 *
 * @retval ::RTEMS_SUCCESSFUL The requested operation was successful.
 *
 * @retval ::RTEMS_INVALID_NAME The ``name`` parameter was invalid.
 *
 * @retval ::RTEMS_INVALID_ADDRESS The ``id`` parameter was NULL.
 *
 * @retval ::RTEMS_TOO_MANY There was no inactive object available to create a
 *   timer.  The number of timers available to the application is configured
 *   through the @ref CONFIGURE_MAXIMUM_TIMERS application configuration
 *   option.
 *
 * @par Notes
 * @parblock
 * The processor used to maintain the timer is the processor of the calling
 * task at some point during the timer creation.
 *
 * For control and maintenance of the timer, RTEMS allocates a TMCB from the
 * local TMCB free pool and initializes it.
 * @endparblock
 *
 * @par Constraints
 * @parblock
 * The following constraints apply to this directive:
 *
 * - The directive may be called from within device driver initialization
 *   context.
 *
 * - The directive may be called from within task context.
 *
 * - The directive may obtain and release the object allocator mutex.  This may
 *   cause the calling task to be preempted.
 *
 * - The number of timers available to the application is configured through
 *   the @ref CONFIGURE_MAXIMUM_TIMERS application configuration option.
 *
 * - Where the object class corresponding to the directive is configured to use
 *   unlimited objects, the directive may allocate memory from the RTEMS
 *   Workspace.
 * @endparblock
 */
rtems_status_code rtems_timer_create( rtems_name name, rtems_id *id );
```

```{raw} latex
\end{footnotesize}
```

Do not guess at behavior from the interface item's prose alone. Read the actual
C implementation next, quoted below verbatim from
`cpukit/rtems/src/timercreate.c`:

```{raw} latex
\begin{footnotesize}
```

```{code-block} c
---
linenos:
---
rtems_status_code rtems_timer_create( rtems_name name, rtems_id *id )
{
  Timer_Control *the_timer;

  if ( !rtems_is_name_valid( name ) ) {
    return RTEMS_INVALID_NAME;
  }

  if ( !id ) {
    return RTEMS_INVALID_ADDRESS;
  }

  the_timer = _Timer_Allocate();

  if ( !the_timer ) {
    _Objects_Allocator_unlock();
    return RTEMS_TOO_MANY;
  }

  the_timer->the_class = TIMER_DORMANT;
  _Watchdog_Preinitialize( &the_timer->Ticker, _Per_CPU_Get_snapshot() );

  *id = _Objects_Open_u32( &_Timer_Information, &the_timer->Object, name );
  _Objects_Allocator_unlock();
  return RTEMS_SUCCESSFUL;
}
```

```{raw} latex
\end{footnotesize}
```

Four branches, checked in this order: an invalid `name`, a `NULL` `id`, no free
timer object available, and otherwise success. This is exactly the kind of
white-box inspection that determines your pre-conditions and post-conditions --
for a function this small, the implementation *is* the specification.

## Step 3: derive the pre-conditions

Ask, for every parameter and every relevant piece of system state: what
distinct value domains change the outcome? Each parameter is a natural starting
point; add one pre-condition for each piece of environment state the
implementation actually branches on.

- **`Name`**: the implementation calls `rtems_is_name_valid()`. Two states:
  `Valid`, `Invalid`.
- **`Id`**: the implementation checks `id` for `NULL`. Two states: `Valid`,
  `Null`.
- **`Free`**: the implementation calls `_Timer_Allocate()`, which can fail if
  no inactive timer object remains. This is not one of the two parameters -- it
  is environment state the test has to arrange (by exhausting the object pool)
  -- but it is still a pre-condition, because it changes the outcome. Two
  states: `Yes`, `No`.

The pre-conditions below, and the post-conditions, trigger action, and
transition-map through the rest of this chapter, are quoted verbatim from
`spec/rtems/timer/req/create.yml`; see the
[RTEMS license](https://www.rtems.org/license/) for the terms that apply to
them.

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
pre-conditions:
- name: Name
  states:
  - name: Valid
    test-code: |
      ctx->name = NAME;
    text: |
      While the $${../if/create:/params[0]/name} parameter is valid.
  - name: Invalid
    test-code: |
      ctx->name = 0;
    text: |
      While the $${../if/create:/params[0]/name} parameter is invalid.
  test-epilogue: null
  test-prologue: null
- name: Id
  states:
  - name: Valid
    test-code: |
      ctx->id = &ctx->id_value;
    text: |
      While the $${../if/create:/params[1]/name} parameter references an object
      of type $${../../type/if/id:/name}.
  - name: 'Null'
    test-code: |
      ctx->id = NULL;
    text: |
      While the $${../if/create:/params[1]/name} parameter is
      $${/c/if/null:/name}.
  test-epilogue: null
  test-prologue: null
- name: Free
  states:
  - name: 'Yes'
    test-code: |
      /* Ensured by the test suite configuration */
    text: |
      While the system has at least one inactive timer object available.
  - name: 'No'
    test-code: |
      ctx->seized_objects = T_seize_objects( Create, NULL );
    text: |
      While the system has no inactive timer object available.
  test-epilogue: null
  test-prologue: null
```

```{raw} latex
\end{footnotesize}
```

Each state's `text` is phrased as a requirement using `While ...`, and its
`test-code` is the C snippet that arranges the system into that state before
the trigger action runs. Notice `'Null'` and `'Yes'`/`'No'` are quoted: they
would otherwise be interpreted as the YAML literals `null`/`true`/`false`.

Two YAML-authoring rules worth internalizing here, from the wider `req/*.yml`
convention:

- Pre-condition and post-condition names are `CamelCase`, never start with a
  digit, and are unique within the item.
- The set of states of one pre-condition must be pairwise disjoint -- a given
  system state must map to exactly one state of each pre-condition, never zero
  and never more than one.

````{admonition} Helper functions for the C code
The pre-condition `Free`, state `No`, needs every free timer object seized.
The call `T_seize_objects( Create, NULL )` above does this. It is declared in
`cpukit/include/rtems/test.h`.

The matching `test-cleanup` attribute must give every seized object back:

```{raw} latex
\begin{footnotesize}
```

```{code-block} c
---
linenos:
---
T_surrender_objects( &ctx->seized_objects, rtems_timer_delete );
```

```{raw} latex
\end{footnotesize}
```

`test-cleanup`, `test-setup`, `test-teardown`, `test-prepare`, and
`test-support` are described in the *Software Test Framework* chapter of the
@`/ref/rtems/eng:/cite-long`.

Do not repeat the same helper function in several `req/*.yml` files. Define
it once in a local C file instead. `spec/rtems/timer/req/create.yml` already
does this: its `test-local-includes` key references the shared
`tx-support.h`/`tx-support.c` pair.
````

## Step 4: derive the post-conditions, and keep them separate

The naive approach is to write one post-condition, `Status`, with four states
(`Ok`, `InvName`, `InvAddr`, `TooMany`) and be done. But look again at what the
implementation actually does on the success path: it also *writes to* `*id`,
and it makes the newly created timer *identifiable by name* through
`rtems_timer_ident()`. These are two more independently observable effects, and
the object being findable-by-name/not is not automatically the same fact as
`IdObj` being set/not.

**The rule**: any set of effects that can vary independently of each other
across the pre-condition space must each be their own post-condition, with its
own states and its own row in the transition-map -- even if one `test-code`
block could conveniently check several of them at once. Do not write a
post-condition whose `text` reads like "the return status shall be X and the
identifier shall be set", bundling two effects into one sentence. That is a
strong warning sign: grep your own drafts for "and ... shall be" in a state's
`text` -- it usually means two effects got merged.

So `rtems_timer_create()` gets three post-conditions:

- **`Status`**: the return status. States: `Ok`, `InvName`, `InvAddr`,
  `TooMany`.
- **`Name`**: whether the object is now findable by name via
  `rtems_timer_ident()`. States: `Valid`, `Invalid`.
- **`IdObj`**: whether the object referenced by `id` was written. States:
  `Set`, `Nop` -- plus `N/A` when the `id` parameter itself is `NULL`, since
  there is then no object to talk about.

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
post-conditions:
- name: Status
  states:
  - name: Ok
    test-code: |
      T_rsc_success( ctx->status );
    text: |
      The return status of $${../if/create:/name} shall be
      $${../../status/if/successful:/name}.
  - name: InvName
    test-code: |
      T_rsc( ctx->status, RTEMS_INVALID_NAME );
    text: |
      The return status of $${../if/create:/name} shall be
      $${../../status/if/invalid-name:/name}.
  - name: InvAddr
    test-code: |
      T_rsc( ctx->status, RTEMS_INVALID_ADDRESS );
    text: |
      The return status of $${../if/create:/name} shall be
      $${../../status/if/invalid-address:/name}.
  - name: TooMany
    test-code: |
      T_rsc( ctx->status, RTEMS_TOO_MANY );
    text: |
      The return status of $${../if/create:/name} shall be
      $${../../status/if/too-many:/name}.
  test-epilogue: null
  test-prologue: null
- name: Name
  states:
  - name: Valid
    test-code: |
      id = 0;
      sc = rtems_timer_ident( NAME, &id );
      T_rsc_success( sc );
      T_eq_u32( id, ctx->id_value );
    text: |
      The unique object name shall identify the timer created by the
      $${../if/create:/name} call.
  - name: Invalid
    test-code: |
      sc = rtems_timer_ident( NAME, &id );
      T_rsc( sc, RTEMS_INVALID_NAME );
    text: |
      The unique object name shall not identify a timer.
  test-epilogue: null
  test-prologue: |
    rtems_status_code sc;
    rtems_id          id;
- name: IdObj
  states:
  - name: Set
    test-code: |
      T_eq_ptr( ctx->id, &ctx->id_value );
      T_ne_u32( ctx->id_value, INVALID_ID );
    text: |
      The value of the object referenced by the $${../if/create:/params[1]/name}
      parameter shall be set to the object identifier of the created timer
      after the return of the $${../if/create:/name} call.
  - name: Nop
    test-code: |
      T_eq_u32( ctx->id_value, INVALID_ID );
    text: |
      The object referenced by the $${../if/create:/params[1]/name} parameter
      shall not be modified by the $${../if/create:/name} call.
  test-epilogue: null
  test-prologue: null
```

```{raw} latex
\end{footnotesize}
```

Every post-condition state's `text` uses `The ... shall be ...`, the
requirement-language mirror of the pre-conditions' `While ...`.

### Post-condition ordering is not arbitrary

List post-conditions in this order:

1. The return status (`Status`, `Retval`, whatever your item calls it).
2. Error indications (`Errno`, if the directive sets it).
3. Output-related states, in the order of the parameters they correspond to.
4. Everything else.

There is one override: if, in the transition-map, one post-condition's logic
needs to look at another post-condition's *already-resolved* state (see
`then-specified-by` below), the referenced post-condition **must** be listed
first, even if that conflicts with the category order. A forward reference to a
not-yet-resolved post-condition cannot be evaluated. That is exactly why
`Status` is listed first here: both `Name` and `IdObj` branch on whether
`Status` resolved to `Ok`.

## Step 5: the trigger action

The trigger action is the one line of C that actually calls the function under
test:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
test-action: |
  ctx->status = rtems_timer_create( ctx->name, ctx->id );
```

```{raw} latex
\end{footnotesize}
```

Everything the pre-conditions set up (`ctx->name`, `ctx->id`) feeds into this
one call, and everything the post-conditions check (`ctx->status`, the object
referenced by `ctx->id`, ...) is observed after it returns. The surrounding
`test-context`, `test-setup`, `test-cleanup`, and similar attributes hold the C
state needed to make this work and follow the project's C test framework
conventions -- treat them as plumbing you will pick up by reading existing
`req/*.yml` items and getting review feedback, rather than something this
chapter derives from first principles.

## Step 6: write the transition-map

With three pre-conditions of two states each, there are eight reachable
combinations. The most literal way to specify the behavior is one row per
combination -- the same shape `specwareview` can render back to you:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
| Entry | Descriptor | Name    | Id    | Free | Status  | Name    | IdObj |
| ----- | ---------- | ------- | ----- | ---- | ------- | ------- | ----- |
| 0     | 0          | Valid   | Valid | Yes  | Ok      | Valid   | Set   |
| 1     | 0          | Valid   | Valid | No   | TooMany | Invalid | Nop   |
| 2     | 0          | Valid   | Null  | Yes  | InvAddr | Invalid | N/A   |
| 3     | 0          | Valid   | Null  | No   | InvAddr | Invalid | N/A   |
| 4     | 0          | Invalid | Valid | Yes  | InvName | Invalid | Nop   |
| 5     | 0          | Invalid | Valid | No   | InvName | Invalid | Nop   |
| 6     | 0          | Invalid | Null  | Yes  | InvName | Invalid | N/A   |
| 7     | 0          | Invalid | Null  | No   | InvName | Invalid | N/A   |
```

```{raw} latex
\end{footnotesize}
```

```{admonition} What N/A means in the table
The post-condition `IdObj` is `N/A` whenever the pre-condition `Id` is
`Null`. No object exists in that case, so no requirement applies to `IdObj`.
Since no requirement applies, no test checks `IdObj` either. The generated
test therefore skips the check code for `IdObj` on every row where `N/A`
applies.
```

You could write this out as eight explicit transition-map descriptors, one per
row. Do not. The project's convention is a single, minimal, **redundancy-free**
descriptor using `if`/`then`/`else` logic instead, with `pre-conditions: all`
for every pre-condition (or an `applicable:` clause where a pre-condition
genuinely does not apply to every combination -- not needed here, since all
three pre-conditions apply to all eight combinations):

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
transition-map:
- enabled-by: true
  post-conditions:
    Status:
    - if:
        pre-conditions:
          Name: Invalid
      then: InvName
    - if:
        pre-conditions:
          Id: 'Null'
      then: InvAddr
    - if:
        pre-conditions:
          Free: 'No'
      then: TooMany
    - else: Ok
    Name:
    - if:
        post-conditions:
          Status: Ok
      then: Valid
    - else: Invalid
    IdObj:
    - if:
        pre-conditions:
          Id: 'Null'
      then: N/A
    - if:
        post-conditions:
          Status: Ok
      then: Set
    - else: Nop
  pre-conditions:
    Name: all
    Id: all
    Free: all
```

```{raw} latex
\end{footnotesize}
```

`Name` does not re-derive "was the name valid, was the id non-null, was there a
free object" -- it simply branches on whether `Status` already resolved to
`Ok`. Chaining off an already-resolved post-condition like this is the primary
technique for keeping a transition-map non-redundant, and it is exactly why
`Status` had to be listed first in Step 4.

`IdObj` cannot chain off `Status` alone: when `Id` is `Null` there is no object
for "was it written" to be about at all, regardless of what `Status` resolves
to, so that case has to be routed to `N/A` by checking `Id` directly, ahead of
the `Status`-chaining fallback. Only once `Id` is known not to be `Null` does
`IdObj` fall back to the same "did `Status` resolve to `Ok`" chain as `Name`.

If you are unsure how to derive this compact form directly, write the naive
one-row-per-combination version first, save it, then mechanically apply the
{ref}`optimization pass <ActionRequirementsOptimizationPass>` from the
reference section below, step by step, to turn it into the if/then/else form.
Two shapes cannot be folded into the single if/then/else descriptor this way,
and must stay their own separate descriptor instead: one using a skip reason
(whose `post-conditions` is a bare string, not a dictionary, so it cannot carry
`if`/`then` logic), and one gated by a build-conditional `enabled-by` other
than `true` (since `if`/`then` can only test pre-condition and post-condition
*values*, never build options).

(ActionRequirementsStep7)=

## Step 7: render, format, and validate

Render the transition-map back into a table and check it against your intent.
For an overview of the `specwareview` and `specverify` tools used in this step,
see {ref}`ToolsSpecification`.

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ uv run specwareview --filter=action-compact-table --format=myst /rtems/timer/req/create
```

```{raw} latex
\end{footnotesize}
```

```{eval-rst}
.. table::
    :class: longtable
    :widths: 22,18,13,24,13,10

    +----------------+-------------+---------+-----------------+---------+-------+
    | Pre-Conditions                         | Post-Conditions                   |
    +----------------+-------------+---------+-----------------+---------+-------+
    | Name           | Id          | Free    | Status          | Name    | IdObj |
    +================+=============+=========+=================+=========+=======+
    | Valid          | Valid       | Yes     | Ok              | Valid   | Set   |
    +----------------+-------------+---------+-----------------+---------+-------+
    | Valid          | Valid       | No      | TooMany         | Invalid | Nop   |
    +----------------+-------------+---------+-----------------+---------+-------+
    | Valid          | Null        | Yes, No | InvAddr         | Invalid | N/A   |
    +----------------+-------------+---------+-----------------+---------+-------+
    | Invalid        | Valid       | Yes, No | InvName         | Invalid | Nop   |
    +----------------+-------------+---------+-----------------+---------+-------+
    | Invalid        | Null        | Yes, No | InvName         | Invalid | N/A   |
    +----------------+-------------+---------+-----------------+---------+-------+
```

This is the fully-resolved semantics of the compact `if`/`then`/`else`
descriptor, grouped back by outcome -- compare it row by row against the
eight-row table from Step 6 (or against your own understanding of the
implementation) before moving on. Notice that the `InvName` outcome needs two
rows, not one: `Id: Valid` and `Id: Null` both produce `InvName`/`Invalid` for
`Status`/`Name`, but they diverge on `IdObj` (`Nop` vs `N/A`), so they cannot
share a row. `--format=myst` is what produces this exact `{eval-rst}` block,
ready to paste straight into a MyST document; use `--format=commonmark` instead
when you want a plain Markdown table for a terminal, a commit message, or a
code review comment.

Now format and validate, exactly as for an interface item:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ uv run specverify --format-items --do-not-indent-lists \
    --clang-format-style=default:file:_clang-format \
    spec/rtems/timer/req/create.yml
$ uv run specverify spec
$ uv run specwareview
```

```{raw} latex
\end{footnotesize}
```

### The diff check: formatting is not semantics

`specverify` only proves the item is *structurally* valid -- well-formed YAML,
resolvable links, no missing attributes. It does **not** prove that a rewrite
of the transition-map (by hand, or by the compaction script) still means the
same thing. For that, capture the rendered table before and after your edit and
diff them:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ uv run specwareview --filter action-compact-table /rtems/timer/req/create > /tmp/before.txt
# ... edit the transition-map ...
$ uv run specverify --format-items --do-not-indent-lists \
    --clang-format-style=default:file:_clang-format \
    spec/rtems/timer/req/create.yml
$ uv run specwareview --filter action-compact-table /rtems/timer/req/create > /tmp/after.txt
$ diff /tmp/before.txt /tmp/after.txt
```

```{raw} latex
\end{footnotesize}
```

An empty diff is your evidence that the rewrite preserved the item's actual
meaning. This has caught real correctness mistakes that a passing `specverify`
did not -- treat it as mandatory whenever you reshape an existing
transition-map, not just when you write one from scratch.

## Reference: the transition-map grammar

This section is a complete reference for the `pre-conditions` and
`post-conditions` blocks inside a `transition-map` descriptor. The
`rtems_timer_create()` example above only needed a fraction of this grammar;
you will need the rest as soon as a function has dependent parameters,
infeasible combinations, or effects that only apply on some paths.

### Restricting which pre-condition combinations a descriptor covers

Each entry under a descriptor's `pre-conditions` key is either the literal
`all` (every state of that pre-condition is feasible here), or a dictionary
with one key, `applicable` or `not-applicable`, whose value is a *pre-condition
state set expression* built from:

- `pre-conditions`: a dictionary of pre-condition name to state name (or a list
  of state names) -- the leaf test.
- `and`: a list of expressions, all of which must hold.
- A bare list anywhere an expression is expected means implicit `or`. This is
  the idiomatic way to combine alternative pre-condition tuples in this
  codebase -- prefer it over spelling out an explicit `or:` key.

For example, a pre-condition `Index` that is only meaningful for certain
combinations of two other pre-conditions might read:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
Index:
  applicable:
  - pre-conditions:
      API: Valid
      Class: ValidMaybeObjects
      Node: Valid
  - pre-conditions:
      Index:
      - Zero
      - Invalid
```

```{raw} latex
\end{footnotesize}
```

`skip` uses the same grammar, under a `condition` key, paired with a `reason`
naming an entry in the item's `skip-reasons` dictionary -- see
{ref}`ActionRequirementsSkipVsNA` below for when to reach for it.

### Deciding each post-condition's state: `if` / `then` / `else` / `specified-by`

Each post-condition is a list of expressions evaluated top to bottom; the first
one whose `if` is true (or which has no `if` at all) wins:

- `if` + `then`: if the condition holds, the state is the literal named by
  `then`.
- `if` + `then-specified-by`: same, but the state is *copied* from the current
  state of the named pre-condition or post-condition, instead of a fixed
  literal.
- `else`: the unconditional fallback, used as the last entry after one or more
  `if` entries.
- `specified-by` on its own (no `if`/`else` at all): an unconditional copy of
  another pre-condition's or post-condition's state, used whenever the
  correspondence holds across the transition's *entire* feasible domain.

The `if` expression's leaf test can check either a `pre-conditions` state (as
above) or a `post-conditions` state -- but only a post-condition **listed
earlier** in the item's top-level `post-conditions:` list, since it must
already be resolved. `and:` combines a `pre-conditions` leaf with a
`post-conditions` leaf in a single expression, for example:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
Owner:
- if:
    and:
    - pre-conditions:
        Count: Zero
    - post-conditions:
        Status: Ok
  then: Caller
- if:
    post-conditions:
      Status: Ok
  then: 'No'
- else: N/A
```

```{raw} latex
\end{footnotesize}
```

`or:` and `not:` also exist in the schema, but no item in this project actually
uses them -- the bare-list-is-`or` shorthand already covers every real case.
Reach for them only if a bare list genuinely cannot express what you need.

### A synthetic worked example combining all four techniques

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
- enabled-by: true
  pre-conditions:
    PreCondA: all
    PreCondB: all
    PreCondC:
      applicable:
        pre-conditions:
          PreCondA: StateU
  post-conditions:
    PostCondA:
    - if:
        pre-conditions:
          PreCondA: StateU
      then: StateU
    - if:
      - pre-conditions:
          PreCondA: StateV
          PreCondB: StateV
      - pre-conditions:
          PreCondA: StateV
          PreCondB: StateW
      then: StateV
    - else: StateW
    PostCondB:
    - specified-by: PreCondB
    PostCondC:
    - if:
        post-conditions:
          PostCondA: StateU
      then-specified-by: PreCondC
    - else: N/A
    PostCondD:
    - if:
        and:
        - pre-conditions:
            PreCondB: StateX
        - post-conditions:
            PostCondA: StateV
      then: Special
    - else: N/A
```

```{raw} latex
\end{footnotesize}
```

Reading it: `PostCondA` uses a plain `if`/`then`, then merges two different
pre-condition tuples that both yield `StateV` into one `if` (a bare list), then
falls back to `else`. `PostCondB` is an unconditional copy of `PreCondB`.
`PostCondC` chains off `PostCondA`'s already-resolved state and copies its
value from `PreCondC` when applicable, `N/A` otherwise. `PostCondD` combines a
pre-condition leaf and a post-condition leaf with `and:`. Note the ordering:
`PostCondA` is listed before `PostCondC` and `PostCondD`, because both of the
latter chain off it -- required by the dependency rule, not a stylistic choice.

(ActionRequirementsSkipVsNA)=

### Skip-reasons, `N/A`, and `Nop` are three different things

These are easy to blur together; they mean three distinct things:

- **Skip reason**: the pre-condition combination is **logically impossible** --
  it cannot occur in reality. For example, a size parameter cannot be positive
  if the pointer it describes is `NULL`. Define a descriptive entry in
  `skip-reasons`, then add a separate transition-map descriptor whose
  `pre-conditions` targets exactly the infeasible combination and whose
  `post-conditions` is the skip reason's name as a bare string (not a
  dictionary). Never use `N/A` for this case.
- **`N/A` (not applicable)**: the combination is feasible, but this *particular
  post-condition* has nothing to say about it -- typically because the object
  the post-condition describes does not exist on this path. `IdObj` above is
  the example: when `Id` is `Null`, nothing was written anywhere, so "was the
  object referenced by `id` modified" is not merely `Nop`, it is structurally
  undefined: `N/A`.
- **`Nop`**: the post-condition is still meaningful and was actively evaluated
  -- it just observed "no operation happened" as its answer (for example, an
  output buffer was not modified because a size parameter was `0`). This is a
  real state with a real state definition, not `N/A`.

Getting `N/A` and `Nop` swapped is the single most common transition-map
mistake: ask yourself "does this post-condition's concept even make sense on
this path" (→ `N/A`) versus "does it make sense, and the answer happens to be
'nothing changed'" (→ `Nop`).

### Never merge independently-varying effects

This was already applied in Step 4 above, but it deserves restating as a
standalone rule, because it is easy to violate by accident once a directive has
more than two effects: any set of effects that can vary independently of each
other across the pre-condition space (the return status, `errno`, an output
parameter's value, an object's internal state, a resource count, which of
several callbacks fired, ...) must each be modeled as their own post-condition.
Two effects are independent the moment there exists at least one feasible
pre-condition combination where one of them takes a state the other does not
mirror. A merged post-condition still compiles, still passes its test, and
still silently throws away requirement granularity a careful author would have
written -- it just cannot be reordered, chained off, or independently reasoned
about afterwards.

(ActionRequirementsOptimizationPass)=

### Optimization pass

Once your transition-map covers every combination correctly, run this
checklist, in this order (each step can expose a new opportunity for the next
one):

1. **Flatten trivial `if`/`else`.** If an `if` branch and the trailing `else`
   resolve to the same state, drop the `if` entirely and write the state as a
   bare literal.
2. **Prefer `specified-by`.** If a post-condition's state always equals a
   pre-condition's (or another post-condition's) state across the whole
   feasible domain, replace the entire chain with a single
   `[{specified-by: X}]`.
3. **Chain off already-resolved post-conditions.** Before re-deriving a
   distinction from `pre-conditions`, check whether an earlier post-condition
   already resolved it -- branch on `post-conditions:` instead of repeating the
   logic (this is exactly what `Name` does by branching on `Status` in the
   running example; `IdObj` does the same for its `Set`/`Nop` split, once its
   own `Id: Null → N/A` case has been ruled out first).
4. **Merge same-pre-condition branches.** If two `if` branches differ only in
   one pre-condition's states and produce the same outcome, combine those
   states into a list under that one key.
5. **Merge cross-tuple branches.** If different pre-condition combinations
   produce the same outcome, combine them with a bare list under `if` (implicit
   `or`), as `PostCondA`'s second branch does in the synthetic example above.

## Checklist

Before you consider an action requirement finished:

- [ ] Read the actual C implementation; did not derive behavior from the
  interface item's prose alone.
- [ ] Every pre-condition's states are pairwise disjoint, `CamelCase`, and
  quoted where YAML would otherwise misinterpret them (`'Null'`, `'Yes'`,
  `'No'`).
- [ ] No post-condition bundles two independently-varying effects into one
  state's `text`.
- [ ] Post-conditions are ordered: status/return value, then error indication,
  then outputs in parameter order, then everything else -- except where a
  dependency forces an earlier position.
- [ ] The transition-map is a single, minimal descriptor per `enabled-by`
  variant, using `if`/`then`/`else`/`specified-by` rather than one row per
  combination.
- [ ] `N/A` is used only where a post-condition's concept does not apply on a
  feasible path, `Nop` for a meaningful "nothing happened" outcome, and a skip
  reason for logically impossible pre-condition combinations.
- [ ] `specwareview --filter action-compact-table` was captured before and
  after any transition-map rewrite, and the diff is empty.
- [ ] The format command of {ref}`Step 7 <ActionRequirementsStep7>`,
  `specverify spec`, and `specwareview` all ran clean.

With both the interface item and its action requirement in place, linked
together and validated, you have produced the complete functional specification
for the directive: the ICD content from {ref}`InterfaceItems`, and the SRS
content from this chapter.

(ActionRequirementsReuse)=

## Reuse a test from another item

This is an advanced topic. This section shows how to call an existing test from
a new action requirement.

RTEMS implements many mechanisms once, in its SuperCore. A directive is often a
thin wrapper around a SuperCore object, for example a SuperCore mutex, a
SuperCore semaphore, or a SuperCore thread queue. An action requirement for
such an object usually already exists.

A new test for a directive that is based on such a SuperCore object can simply
reuse the existing action requirement's test. There is no need to define the
requirement and implement the test again, especially as these SuperCore tests
are often complex.

### An example across three layers

The following example spans the SuperCore, the Classic API, and the newlib API.

```{figure} ../images/spec-item-test-reuse.*
---
alt: Five items across three layers, linked by test-run and other roles
width: 70%
---
Reuse across the SuperCore, the Classic API, and the newlib API
```

The table below lists every file in the example.

```{eval-rst}
.. table::
    :class: longtable
    :widths: 29,16,41,14

    +----------------------------------------+-----------------------+---------------------------------------------------------+---------------------+
    | File                                   | Type                  | Role in the example                                     | Generated test file |
    +========================================+=======================+=========================================================+=====================+
    | score/tq/req/surrender.yml             | requirement, action   | SuperCore thread queue behaviour, reused below          | tr-tq-surrender.c   |
    +----------------------------------------+-----------------------+---------------------------------------------------------+---------------------+
    | score/sem/req/surrender.yml            | requirement, action   | SuperCore semaphore behaviour, reuses the thread queue  | tr-sem-surrender.c  |
    |                                        |                       | test                                                    |                     |
    +----------------------------------------+-----------------------+---------------------------------------------------------+---------------------+
    | rtems/sem/req/release.yml              | requirement, action   | Classic API directive, reuses the semaphore test        | tc-sem-release.c    |
    +----------------------------------------+-----------------------+---------------------------------------------------------+---------------------+
    | newlib/req/sys-lock-semaphore-post.yml | requirement, function | Newlib API directive, no test of its own                | none                |
    +----------------------------------------+-----------------------+---------------------------------------------------------+---------------------+
    | newlib/val/sys-lock.yml                | test-case             | Validates the newlib directive, reuses the semaphore    | tc-sys-lock.c       |
    |                                        |                       | test                                                    |                     |
    +----------------------------------------+-----------------------+---------------------------------------------------------+---------------------+
```

`score/tq/req/surrender` specifies the dequeue and unblock behaviour of a
SuperCore thread queue. `score/sem/req/surrender` specifies the count and
status behaviour of a SuperCore semaphore. It reuses the thread queue test for
the dequeue behaviour.

Two directives reuse the test of `score/sem/req/surrender` in turn.
`rtems/sem/req/release` covers the Classic API directive
`rtems_semaphore_release()`. `newlib/req/sys-lock-semaphore-post` covers the
newlib directive `_Semaphore_Post()`. Its validation test is in
`newlib/val/sys-lock`.

### Invoke another item's test

The item `newlib/req/sys-lock-semaphore-post` contains only the requirement
text. The following excerpt is from the real item. See the
[RTEMS license](https://www.rtems.org/license/) for the terms that apply to it,
and to every other RTEMS source quoted in this section.

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
links:
- role: interface-function
  uid: ../if/sys-lock-semaphore-post
- role: function-implementation
  uid: /score/sem/req/surrender
- role: requirement-refinement
  uid: sys-lock
# ...
text: |
  The $${../if/sys-lock-semaphore-post:/name} directive shall surrender the
  counting modulo semaphore as specified by
  $${/score/sem/req/surrender:/spec}.
```

```{raw} latex
\end{footnotesize}
```

The `function-implementation` link points at the SuperCore item whose behaviour
the newlib directive shares. The `:spec` path in the `text` attribute returns
the full rendered requirement text of that item, so the requirement text above
avoids repetition.

Every requirement must have a test that validates it. For the requirement-only
item `newlib/req/sys-lock-semaphore-post`, that test is a check in
`newlib/val/sys-lock`, a specification item of type `test-case`. One of its
checks validates the directive:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
- brief: |
    Validate the $${../if/sys-lock-semaphore-post:/name} directive.
  code: |
    ctx->tq_sem_ctx.base.surrender = SemaphorePost;
    ctx->tq_sem_ctx.base.wait = TQ_WAIT_FOREVER;
    ctx->tq_sem_ctx.variant = TQ_SEM_COUNTING_MODULO;
    SemaphoreSetCount( &ctx->tq_sem_ctx, 1 );
    $${/score/sem/req/surrender:/test-run}( &ctx->tq_sem_ctx );
  links:
  - role: validation
    uid: ../req/sys-lock-semaphore-post
```

```{raw} latex
\end{footnotesize}
```

The `role: validation` link marks this check as the test for
`newlib/req/sys-lock-semaphore-post`. The check's own `code` sets up a
`TQSemContext` for a counting modulo semaphore, and calls
`$${/score/sem/req/surrender:/test-run}( &ctx->tq_sem_ctx )`. The call runs the
whole SuperCore test. That test calls `_Semaphore_Post()` through a function
pointer stored in `TQSemContext`.

The substitution `$${<uid>:/test-run}( <args> )` calls an action requirement's
generated test function. This function exists only for an item whose
`test-header` attribute is not null. Such an item does not generate a
standalone test. It generates a test meant for other tests to call, together
with a C header file named by the `target` attribute. The `run-params`
attribute lists the parameters of the generated function. The relevant part of
`score/sem/req/surrender`:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
test-header:
  run-params:
  - description: |
      is the thread queue context.
    dir: inout
    name: tq_ctx
    specifier: TQSemContext *$${.:name}
  target: testsuites/validation/tr-sem-surrender.h
```

```{raw} latex
\end{footnotesize}
```

The generated function takes one parameter, `tq_ctx`, of type `TQSemContext *`.
This is why the check above prepares a `TQSemContext` before it calls
`$${<uid>:/test-run}`.

### Invoke another item's test from an action requirement

`rtems/sem/req/release` is a full action requirement, with its own
pre-conditions, post-conditions, and transition-map for
`rtems_semaphore_release()`. It reuses `score/sem/req/surrender` inside one of
its own post-condition states:

```{raw} latex
\begin{footnotesize}
```

```{code-block} yaml
---
linenos:
---
- name: BinarySurrender
  test-code: |
    ctx->tq_ctx.enqueue_variant = TQ_ENQUEUE_BLOCKS;
    ctx->tq_ctx.get_owner = NULL;
    ctx->tq_sem_ctx.variant = TQ_SEM_BINARY;
    ctx->tq_sem_ctx.get_count = TQSemGetCountClassic;
    ctx->tq_sem_ctx.set_count = TQSemSetCountClassic;
    $${/score/sem/req/surrender:/test-run}( &ctx->tq_sem_ctx );
  text: |
    The calling task shall surrender the binary semaphore as specified by
    $${/score/sem/req/surrender:/spec}.
```

```{raw} latex
\end{footnotesize}
```

`rtems_semaphore_release()` uses more than one SuperCore mechanism: a SuperCore
mutex and a SuperCore semaphore. `rtems/sem/req/release` therefore carries two
`function-implementation` links, one to `score/sem/req/surrender` and one to
`score/mtx/req/surrender`. Each post-condition state reuses whichever SuperCore
item matches its pre-conditions.

The figure and the table leave `score/mtx/req/surrender` out. This example
follows only the semaphore path.

The context `ctx->tq_sem_ctx` is of type `TQSemContext`, the same type
`score/sem/req/surrender` expects. Before the `BinarySurrender` state calls
`$${/score/sem/req/surrender:/test-run}( &ctx->tq_sem_ctx )`, it assigns
`TQSemGetCountClassic` and `TQSemSetCountClassic` to two function pointers
inside that context:

```{raw} latex
\begin{footnotesize}
```

```{code-block} c
---
linenos:
---
typedef struct TQSemContext {
  TQContext base;
  TQSemVariant variant;
  uint32_t ( *get_count )( struct TQSemContext * );
  void ( *set_count )( struct TQSemContext *, uint32_t );
} TQSemContext;
```

```{raw} latex
\end{footnotesize}
```

Through this function pointer pair the test can run against a Classic API
semaphore, a POSIX semaphore, or a newlib semaphore. Each caller assigns its
own pair of adapter functions before it calls `...:/test-run`. The adapter
functions, together with `TQSemContext` and the other context types, live in a
shared header, `testsuites/validation/tx-thread-queue.h`, and its matching
source file.

### Naming conventions

A file name that starts with `tr-` holds a reusable test. Its item has a
`test-header` attribute, and another item calls it through `...:/test-run`.

A file name that starts with `tc-` holds a top-level test case. A `tc-` item
can call a lower-level `tr-` test, as `rtems/sem/req/release` does for
`score/sem/req/surrender`.

### Reuse a test in your own item

1. Find the item to reuse. Search for `run-params`, as shown below, or follow a
   `function-implementation` link from a similar item.
2. Read the item's `test-header` attribute. It gives the parameter list of the
   generated `...:/test-run` function, and the header that declares it.
3. Decide whether your new item needs its own transition-map, or whether a
   simple reuse is enough, as `newlib/req/sys-lock-semaphore-post` does.
4. For a simple reuse:
   1. Add a `function-implementation` link to the requirement item. Write a
      `text` that reads `... as specified by $${<uid>:/spec}`.
   2. Extend or create a validation test item below `val/`. Add the reused
      item's `test-header` filename to its `test-local-includes`. Add a check
      with a `role: validation` link to it. Prepare the parameters in the
      check's `code`, and call `$${<uid>:/test-run}` from the same code.
5. For your own transition-map:
   1. Prepare the parameters the reused item expects, inside the relevant
      post-condition's `test-code`.
   2. Call `$${<uid>:/test-run}` with those parameters. Add the reused item's
      `test-header` target header to your `test-local-includes`.
6. Format and validate the item, as in {ref}`Step 7 <ActionRequirementsStep7>`.

### List the reusable tests

Search for every item with a reusable test by its `run-params` attribute:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ grep -rl "run-params:" --include=*.yml spec
```

```{raw} latex
\end{footnotesize}
```
