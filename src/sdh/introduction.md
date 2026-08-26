% SPDX-License-Identifier: CC-BY-SA-4.0

% Copyright (C) 2026 embedded brains GmbH & Co. KG

# Introduction

This Software Development Handbook (SDH) is a guide to the software engineering
activities required for the pre-qualification of @`/glossary/rtems:/term`
@`/glossary/qdp:/plural`. It provides rules, best practices, and reference
material for qualification engineers.

This handbook is intended for engineers who want to extend or improve the
package itself. This includes fixing bugs in qualified RTEMS code or validation
test suites, extending the set of pre-qualified RTEMS features and
@`/glossary/api:/term` functions or macros, and working on the
[qualification toolchain](https://github.com/specthings).

Users of a package, or developers who want to build applications on top of
qualified RTEMS, should instead refer to the @`doc-package-manual:/cite-long`.

```{admonition} How to read this document
---
class: note
---
Do not read this entire handbook from beginning to end. Read only the
chapters or sections relevant to your work:

{ref}`SdhIntroQualification`
: If you are new to qualification, start here. These sections provide a
  foundational introduction to the concepts and processes used throughout
  this handbook.

{ref}`SpecificationItems`
: If you want to work on RTEMS pre-qualification, read the introduction to
  the concept of specification items.

{ref}`DocumentationGuidelines`
: If you want to write or edit a document such as this SDH, read the
  documentation style guidelines first.

{ref}`QualEngEnvironment`
: If you want to modify pre-qualified RTEMS source code, start by learning
  how to set up the qualification engineering environment.

{ref}`QualificationEngineering`
: If you want to pre-qualify additional RTEMS features or work on code for
  already qualified features, this chapter introduces the complete
  qualification workflow.

{ref}`InterfaceItems`
: If you want to pre-qualify additional RTEMS functions or macros, this
  chapter explains how to specify their APIs.

{ref}`ActionRequirements`
: If you want to write requirements and validation tests, follow this
  step-by-step guide.

{ref}`ToolsBuildDebugSimulation`
: If you need information about build, debugging, or simulation tools,
  start with these sections.

{ref}`ToolsSpecification`
: If you want to edit or understand specification items, you should be
  familiar with the specification tools described here.
```

(SdhIntroQualification)=

## Understand qualification

Qualification is intended to demonstrate that a product can be used in
safety-critical applications. The development process and the product must
undergo a certification by an authority and must follow a standard that defines
rules and requirements which must be met.

## Understand pre-qualification

Usually only a whole product is qualifiable (software and hardware together,
for example an airplane or a satellite). Parts of a product, such as RTEMS,
cannot be qualified on their own.

Some items are (possibly) part of many products. To avoid the effort to qualify
such a part over and over again for each product in which it is used, the item
can be *pre-qualified*.

Pre-qualified means that the qualification of such an item, here RTEMS, is
prepared as far as possible. This encompasses, for example, specifying the
software, writing all requirements, testing the software and providing all
documentation demanded by the qualification standard. As a result, the user of
such a pre-qualified part has significantly less effort to qualify the whole
product.

## Understand qualification standards

Software qualification is governed by standards that define what a product must
do to demonstrate that it is safe, reliable, and fit for its intended use. The
applicable standard depends on the industry and the type of product.

Some examples are:

- **Space**

  - Europe/ESA: @`/ref/ecss/e-st-40c-r1:/cite-long`,
    @`/ref/ecss/q-st-80c-r2:/cite-long`
  - USA/NASA: @`/ref/std/npr-7150-2d:/cite-long` and
    @`/ref/std/nasa-std-8739.8:/cite-long`

- **Aircraft:** @`/ref/std/do-178:/cite-long`

- **Medical electrical equipment:** @`/ref/std/iec-60601:/cite-long`

- **Automotive:** @`/ref/std/iso-26262:/cite-long` and
  @`/ref/std/iec-61508-1:/cite-long`, @`/ref/std/iec-61508-3:/cite-long`

Although these standards differ considerably in their details, they follow a
similar principle. They define requirements and processes that the product and
its development must satisfy. They also specify which documents, tests,
analyses, and other evidence must be produced to demonstrate compliance.

A qualification or certification authority typically accompanies the project
throughout its development and reviews this evidence. At the end of the
process, the authority certifies that the product meets the applicable standard
and is therefore qualified for its intended use.

## Understand the ECSS standard

For European space applications, the relevant standards are provided by
[ECSS (European Cooperation for Space Standardization)](https://ecss.nl/standards/),
with the European Space Agency (ESA) acting as the certification authority.

ECSS consists of a large collection of standards. For software, two of the most
relevant are:

- @`/ref/ecss/e-st-40c-r1:/cite-long`

- @`/ref/ecss/q-st-80c-r2:/cite-long`

The ECSS standards essentially consist of numbered clauses that define the
requirements a software project must follow. They also specify the documents,
tests, and other evidence that a project must produce to demonstrate compliance
with these requirements.

ECSS also provides handbooks that explain and provide guidance on how to
interpret and apply the standards.

Because software products vary widely while the standard is generic, the
standard is individually tailored to each project. This tailoring defines how a
project intends to meet the ECSS clauses. For RTEMS, the tailoring is in a
document named *ECSS Compliance Matrix*.

You do not need to read these standards before continuing with this book. They
are extensive documents, and this book provides the instructions needed to work
on pre-qualifying RTEMS without requiring you to study the standards first.

## Understand ECSS software criticality categories

Not all software has the same importance for the safety and success of a space
mission. ECSS therefore assigns software to one of four criticality categories
(see @`/ref/ecss/q-st-80c-r2:/cite-long` annex D). The following table is
simplified:

| Criticality category | Severity     | Example                                      |
| -------------------- | ------------ | -------------------------------------------- |
| A                    | Catastrophic | Loss of human life or environmental disaster |
| B                    | Critical     | Loss of the spacecraft, the mission, or both |
| C                    | Major        | Significant mission impairment               |
| D                    | Minor        | Small mission impairment                     |

The category is assigned based on the most critical function implemented by the
software. Hardware, other software, or operational procedures that can prevent
or mitigate the consequences of a software failure can also influence the
classification. The method to assess the criticality of a product is a
(Software) Failure Modes, Effects and Criticality Analysis
(@`/glossary/sfmeca:/term`) (see @`/ref/ecss/q-st-30c-r1:/cite-long`).

The major difference between category B and C is that B requires an
*Independent Software Verification and Validation (ISVV)* additionally.
@`/glossary/isvv:/term` means that a team totally independent from the original
developers, usually another company, reviews and checks the software and its
development evidence once more, rather than relying solely on the developers'
own verification activities.

For the pre-qualification of RTEMS, the criticality category is usually B
provided an ISVV is conducted.

## Understand the history of RTEMS pre-qualification

RTEMS pre-qualification has evolved considerably over the years. For the first
@`/glossary/esa:/term` project, EDISOFT pre-qualified RTEMS 4.8.

A major lesson from this project was that a largely manual qualification
process makes updates expensive. Repeating the qualification for a new RTEMS
version or extending it to additional features would require a significant
amount of manual work.

The subsequent qualification activities therefore had a second goal: to build a
*qualification toolchain (QT)*. Wherever possible, the toolchain automates the
qualification process, including running tests on simulators and target
hardware, collecting code coverage, checking results, generating traceability
matrices and documentation, and building the package. This makes it much more
practical to maintain and extend the pre-qualification.

The publicly available packages have evolved as follows:

| package | RTEMS | Time         | Cat | Target                         | Main change                                                              |
| ------- | ----- | ------------ | --- | ------------------------------ | ------------------------------------------------------------------------ |
| **3**   | 6     | 2019 -- 2021 | C   | GR712RC, GR740, SMP            | package by *EDISOFT, embedded brains, LERO, Jena-Optronik, CISTER*.      |
| **4**   | 6     | 2022         | C   | GR712RC, GR740, SMP            | Updated RTEMS baseline, and integrated validation tests                  |
| **5**   | 6     | 2021 -- 2024 | B   | GR712RC, GR740, SMP            | ISVV by *Critical Software* and upgraded to ECSS category B              |
| **6**   | 6.0   | 2024         | B   | GR712RC, GR740, SMP/UNI        | RTEMS baseline, uni-processor coverage (UNI becomes category B)          |
| **6.1** | 6.0   | 2025         | B   | GR712RC, GR740, SMP/UNI        | Updated package and RTEMS baseline, further corrections and improvements |
| **6.2** | 6.2   | 2026         | B   | GR712RC, GR740, SMP/UNI        | Bug fixes, Update to RTEMS 6.2, user can run tests on HW                 |
| **7**   | 7     | 2025 -- now  | B   | GR712RC, GR740, GR765, SMP/UNI | Update to newest RTEMS, add support for GR765, add POSIX API, new ISVV   |

These packages are publicly available at no cost from
[https://rtems-qual.io.esa.int/](https://rtems-qual.io.esa.int/).

```{admonition} Other architectures, device drivers and maintenance
Professional support as well as packages for other architectures,
third-party libraries, and qualified device drivers, are available from
commercial service providers. See the **Service and Maintenance** section
in the @`doc-package-manual:/cite-long` for
details. If you want to support RTEMS maintenance see
[RTEMS Foundation](https://www.rtemsfoundation.org/).
```

## Understand the RTEMS pre-qualified API

Not all features of RTEMS have been pre-qualified. Instead only those parts of
the API which are mostly used in space applications have been selected for
qualification. This pre-qualified API subset has been extended over time.

The pre-qualified API (or pre-qualified *feature set*) is listed in the
@`doc-package-manual:/cite-long` section *Pre-Qualified Interfaces*.

Technically, whether source code is in the pre-qualified subset or not is
defined on a per-file basis. See {ref}`QualEngAddFunction` for details.

(SdhIntroRTEMSPreQualification)=

## Understand RTEMS pre-qualification projects

RTEMS pre-qualification is carried out within an *ESA project*. This is
important because ESA acts as the certification authority: it needs to review
the qualification activities from the initial planning through to the final
review and ultimately approve the results. ECSS software engineering and
product assurance explicitly cover processes throughout the software life
cycle.

```{admonition} You cannot qualify finished software afterwards
---
class: warning
---
It is tempting to first finish the software and then contact ESA and ask
for it to be qualified. In general, this does not work.

Qualification is not only about testing the finished product. ECSS also
specifies **how the software is developed, reviewed, tested, and
documented**. ESA therefore needs to be involved from the beginning,
including the planning phase. This allows ESA to identify and correct
approaches that do not comply with the applicable ECSS requirements while
the project can still react to them.
```

A qualification project is usually carried out by several companies working
together, with one company acting as the *lead*. The project starts with a
*Kick-off Meeting (KoM)* and is divided into several phases. At the end of each
phase, the project partners and ESA hold a *joint review meeting*.

Before each review, the project plan defines which work has to be completed and
which documents and software have to be provided to ESA. ESA reviews these
artifacts and records any findings as *Review Item Discrepancies (RIDs)*. The
project partners address the RIDs and provide the required corrections. In this
way, ESA continuously monitors and guides the qualification project.

During the project, the partners typically perform a wide range of activities:
they modify and review code, add specifications, requirements, and tests, work
towards the required code-coverage goals, investigate hardware errata, and
perform their own *Product Assurance (PA)* activities. Important documents
include:

- *Software Development Plan (SDP)*
- *Test Reports (TR)*
- *Software Verification Report (SVR)*, and
- *Software Product Assurance Milestone Report (SPAMR)*, among others.

ESA reviews the documentation and source code, executes the tests on its own
hardware, and even re-builds the package. The latter therefore contains the
evidence that the qualified RTEMS configuration meets the applicable ECSS
requirements.

Unlike some other certification schemes, ESA does not issue a separate
certificate stating that a package is "pre-qualified". Instead, ESA records
internally that the package has undergone the required qualification process.
When a space project later uses that package, the qualification work covered by
it does not have to be repeated.

## Find an overview of the content of a package

The actual content of a package varies. The chapter *Software configuration
item overview* of the @`doc-package-manual:/cite-long` provides a complete list
for the package you are using.

## Understand the role of ESA in product qualification

Imagine a company that wants to build a new satellite with application software
running on top of RTEMS. At the end of the project, the company wants ESA to
qualify the satellite according to the ECSS standard.

This can only happen as part of an ESA project, with ESA involved from the
beginning. For the software, the project first needs to establish, among other
things:

- a Software Development Plan (SDP) (see *Annex O* of
  @`/ref/ecss/e-st-40c-r1:/cite-long`),
- the software criticality category (see @`/ref/ecss/q-st-30c-r1:/cite-long`),
- the requirements baseline, and
- the tailoring of the ECSS requirements to the project (see *Annex D* of
  @`/ref/ecss/q-st-80c-r2:/cite-long`).

From this point on, the project proceeds through regular progress meetings with
ESA and a series of phases, each ending in a *Joint Review Meeting*. For
software, these reviews typically include:

- Software Requirements Review (SRR)
- Preliminary Design Review (PDR)
- Critical Design Review (CDR)
- Qualification Review (QR)
- Acceptance Review (AR)
- Final Review (FR)

For each review, the company provides ESA with the documents, software, and
other artifacts required by the applicable ECSS standards (see *Annex F* of
@`/ref/ecss/q-st-80c-r2:/cite-long`). ESA reviews these artifacts against the
applicable ECSS requirements and raises discrepancies where corrections are
needed. The project addresses these findings before proceeding.

At the **Final Review**, ESA assesses the completed qualification evidence and,
if all requirements have been satisfied, can approve the satellite as qualified
according to the applicable ECSS requirements.

```{admonition} How does a user of a package benefit from the pre-qualification?
---
class: note
---
Imagine a company developing an application that uses pre-qualified RTEMS
functionality as its operating system within an ESA project. Because the
relevant RTEMS configuration has already been pre-qualified, much of the
required documentation, testing, and verification evidence already exists.

The application project still needs to, among other things:

- establish the requirements baseline for the application,
- develop the application using the pre-qualified RTEMS functions and
  macros,
- write and execute unit and validation tests for the application,
- meet the applicable code-coverage goals for the application code,
- perform the required application code reviews,
- measure or calculate timing and memory consumption where required,
  using the RTEMS measurement data documented in the RTEMS SVR and SCF
  where applicable,
- re-run the package test suite on the hardware used by the project to
  demonstrate that the qualified RTEMS configuration works correctly on
  that hardware,
- produce the ECSS documentation required for the application,
  referencing the corresponding RTEMS qualification documentation where
  appropriate,
- perform Product Assurance (PA) activities for the development process
  of the application, resulting in an application SPAMR, and
- perform Independent Software Verification and Validation (ISVV) where
  required.

The important point is that **only the whole device (hardware,
application and RTEMS software) can be qualified; not RTEMS or the
application alone**. The package provides already-qualified evidence
that the project can reuse, significantly reducing the amount of
qualification work that has to be performed again.
```

Note there is no requirement that European space projects adhere to ECSS. A
project is free to decide whether it adopts ECSS or not. Yet, ESA will likely
require ECSS when it finances the project.
