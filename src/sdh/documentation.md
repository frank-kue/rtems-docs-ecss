% SPDX-License-Identifier: CC-BY-SA-4.0

% Copyright (C) 2026 embedded brains GmbH & Co. KG

(DocumentationGuidelines)=

# Documentation guidelines

This document describes how to write and format documentation within the RTEMS
ECSS documentation project. It covers formatting, style, licensing, custom
markup extensions, and building the documentation locally.

## File formats

The project uses two markup formats:

- **MyST (Markdown)**: Used for all new documentation files.

- **reStructuredText (reST)**: Maintained for existing documentation files.

All new documentation files shall use MyST Markdown.

## File templates and headers

Every documentation file shall start with a license and copyright header. The
preferred license for documentation is `CC-BY-SA-4.0`.

### MyST header template

For MyST Markdown (`*.md`) files, the header shall be written as Markdown
comments starting with `%` on a line by itself:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
% SPDX-License-Identifier: CC-BY-SA-4.0

% Copyright (C) <YEARS> <COPYRIGHT HOLDER>
```

```{raw} latex
\end{footnotesize}
```

### reST header template

For reStructuredText (`*.rst`) files, the header shall be written as reST
comments starting with `..`:

```{raw} latex
\begin{footnotesize}
```

```{code-block} rst
---
linenos:
---
.. SPDX-License-Identifier: CC-BY-SA-4.0

.. Copyright (C) <YEARS> <COPYRIGHT HOLDER>
```

```{raw} latex
\end{footnotesize}
```

Replace `<YEARS>` with the first year of substantial contribution to the file,
and optionally the last year of modification if they differ (for example
`2020, 2026`). Replace `<COPYRIGHT HOLDER>` with the name of the copyright
holder.

## Style and formatting guidelines

Adhering to style guidelines is critical to ensure readability, consistent
rendering, and clean version control diffs.

### Line length

To maintain readability and clean version control, limit the line length of
text and code:

- **MyST (Markdown)**: Text in `*.md` files shall wrap at 79 characters, which
  is automatically enforced by `mdformat`.

- **reStructuredText (reST)**: Text in `*.rst` files shall wrap at 79
  characters.

- **Exceptions**: Long URLs or specific code blocks where wrapping breaks the
  syntax.

### Heading hierarchy and casing

- **Casing**: All section headers and titles shall use **sentence case**. Do
  not use [title case](https://en.wikipedia.org/wiki/Title_case).

  - Correct: `# Set up the build environment`

  - Incorrect: `# Set Up The Build Environment`

- **Phrasing**: Section headings shall use the imperative/advice form (a bare
  infinitive verb), not a question. Do not phrase a heading as "How to ...",
  with or without a trailing question mark.

  - Correct: `## Create a feature branch`

  - Incorrect: `## How to create a feature branch?`

  Per the
  [Google developer documentation style guide](https://developers.google.com/style/headings),
  a task-based heading shall start with a bare infinitive (for example
  `Create an instance`, not `Creating an instance`). A heading opening with
  "How to" does not start with a bare infinitive either, so it violates the
  same rule.

- **MyST headings**: Use the standard Markdown heading markers:

  - Level 1: `# Heading`

  - Level 2: `## Heading`

  - Level 3: `### Heading`

  - Level 4: `#### Heading`

- **reST headings**: Use the following decorators for headings:

  - Level 1 (Title): Asterisks (`*`) under the text.

    ```{raw} latex
    \begin{footnotesize}
    ```

    ```{code-block} rst
    ---
    linenos:
    ---
    Title text
    **********
    ```

    ```{raw} latex
    \end{footnotesize}
    ```

  - Level 2 (Section): Equal signs (`=`) under the text.

    ```{raw} latex
    \begin{footnotesize}
    ```

    ```{code-block} rst
    ---
    linenos:
    ---
    Section text
    ============
    ```

    ```{raw} latex
    \end{footnotesize}
    ```

  - Level 3 (Subsection): Dashes (`-`) under the text.

    ```{raw} latex
    \begin{footnotesize}
    ```

    ```{code-block} rst
    ---
    linenos:
    ---
    Subsection text
    ---------------
    ```

    ```{raw} latex
    \end{footnotesize}
    ```

  - Level 4 (Sub-subsection): Tildes (`~`) under the text.

    ```{raw} latex
    \begin{footnotesize}
    ```

    ```{code-block} rst
    ---
    linenos:
    ---
    Sub-subsection text
    ~~~~~~~~~~~~~~~~~~~
    ```

    ```{raw} latex
    \end{footnotesize}
    ```

Ensure the decorator line is exactly the same length as the heading text.

### Indentation and whitespace

- Use spaces only. Do not use tab characters.

- **MyST (Markdown)**: Do not hand-format indentation in `*.md` files. Run
  `mdformat` and let it decide the exact spacing; it does not always use 2
  spaces per level (for example, list continuation lines are aligned to the
  width of the list marker).

- **reStructuredText (reST)**: Use 4 spaces for one indentation level in
  `*.rst` files, since there is no automatic formatter enforcing indentation
  for this format. List continuation lines shall be aligned to the width of the
  list marker.

- Avoid trailing whitespace at the end of lines.

- Do not use more than one blank line in a row.

- Separate list items by one blank line.

- Separate code blocks by one blank line from the surrounding text.

### reST substitutions

Do not use the
[reST substitutions](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html#substitutions).
In particular, do not add new items to `src/common/include/abbreviations.rst`.

## Document generation and specification items

Documents are generated using
[specmake](https://github.com/specthings/specmake) tools.

The document generation is defined by specification items. Specification items
play a key role in the document generation, the software specification, the
build of software components, and the package building. It is important to
understand the concepts outlined in {ref}`SpecificationItems`. The format of
specification items is defined by a hierarchical type system presented in
{ref}`SpecificationItemHierarchy`. The documentation generator verifies the
specification item format before specification items are used to build
documents. Specification items may contain text values in reStructuredText or
MyST format.

Each documentation source file belongs to a document item. This item is the
current item of the variable substitution in the file. The identifier `.` in a
substitution denotes it. The item UID of a package-level document is
`/pkg/deployment/<document>`.

This section writes a substitution in the `$${<uid>:<attribute-path>}` form. In
MyST text, prefer the `` @@`<uid>:<attribute-path>` `` form, see
{ref}`SpecificationItems`.

### Documentation variables and placeholders

In general, for documentation variables and placeholders use the
`<my-placeholder>` notation.

For command-line examples, use the `$${MY_PLACEHOLDER}` notation (upper case,
like a shell variable). Please note that a `$${` sequence is subject to the
variable substitution by the documentation build system. For documentation
variables and placeholders, you have to escape the `$$`. For example, in the
documentation sources write `$$$${MY_PLACEHOLDER}` so that it gets displayed as
`$${MY_PLACEHOLDER}`.

### Section cross-references

To link to a heading anywhere in the documentation, first give it an anchor,
separated from the heading by exactly one blank line:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
(MyAnchorName)=

## My section
```

```{raw} latex
\end{footnotesize}
```

Then link to it from anywhere else, including from other files, using the
`{ref}` role:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
See {ref}`MyAnchorName`.
```

```{raw} latex
\end{footnotesize}
```

By default, the link text is the target heading's own text. To use different
link text, add it before the anchor name, separated by a space:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
See the {ref}`previous chapter <MyAnchorName>`.
```

```{raw} latex
\end{footnotesize}
```

Do not supply custom link text that merely repeats the target heading's own
text; omit it and rely on the default.

Anchor names shall be written in `PascalCase` and shall be unique across the
entire documentation project, not just within one file: all chapters are
compiled into a single document, so a name collision between two files' anchors
is a build error, not merely a local one.

Never use plain Markdown cross-references (for example, a heading link such as
`[My section](#my-section)`, or a relative link to another file such as
`[the how-to](specification-how-to.md)`) to refer to a section. These rely on
the generated slug or file path, which silently breaks when a heading is
reworded or a file is renamed or moved. The `{ref}` role must be used for every
section cross-reference instead, since it resolves the anchor at build time and
fails the build if the target no longer exists, rather than producing a
silently dead link.

### Glossary references and terms

A glossary term reference has two effects. First, it adds the term to the
glossary of the document. The document-specific glossary therefore lists
exactly the terms which the document references. Second, it prints the term
with a link to the term definition in the glossary.

The link highlights the term visually. This effect competes with the emphasize
and strongly emphasize text roles. Use a glossary term reference at most once
per section. Reference a commonly known term only once per chapter or even once
per document.

To reference a glossary term, use the following syntax:

- Singular term: `$${/glossary/<term-id>:/term}`

- Plural term: `$${/glossary/<term-id>:/plural}`

MyST examples:

- `` ... using the @@`/glossary/rtems:/term` operating system. ``

- `` ... for various @@`/glossary/api:/plural`. ``

reST examples:

- `... using the $${/glossary/rtems:/term} operating system.`

- `... for various $${/glossary/api:/plural}.`

Glossary terms are defined by @`/spec/glossary-term:/spec-name` items. Glossary
terms shall be a member of a @`/spec/glossary-group:/spec-name` item through a
@`/spec/glossary-member:/spec-name` link. The general glossary of terms is
represented by the `/glossary/group` item.

The RTEMS ECSS documentation set has three main locations for glossary term
items:

1. The specification set within the RTEMS sources: `rtems/spec/glossary/`. The
   RTEMS specification has to be self-contained, since the RTEMS documentation
   sources come from it. A @`/glossary/qdp:/term` build has both the RTEMS and
   the RTEMS ECSS documentation sources.

2. The specification set within the RTEMS ECSS documentation sources:
   `rtems-ecss-docs/spec/glossary/`. The glossary terms defined here shall not
   overlap with the glossary terms of the RTEMS sources. A QDP build should
   have exactly one glossary item for each term.

3. The dummy software specification set within the RTEMS ECSS documentation
   sources: `rtems-ecss-docs/dummy-software/spec/glossary/`. This set makes it
   possible to build the RTEMS ECSS documentation set without a full QDP build.
   It shall only contain the glossary term definitions which the documents
   strictly require. For a QDP build, the RTEMS sources normally provide these
   definitions. A QDP build does not use this set.

(DocumentCitations)=

### Document citations

A short citation prints the reference code of a document. A long citation
prints the full title and the reference code. Both add the document to the
bibliography of the citing document. You can cite @`/spec/reference:/spec-name`
and @`/spec/pkg-sphinx-document:/spec-name` items.

Use the following syntax:

- Short citation: `$${<uid>:/cite}`

- Long citation: `$${<uid>:/cite-long}`

The `<uid>` is an absolute or a relative item UID. A relative UID resolves
against the directory of the document item. The package-level documents are
siblings in `/pkg/deployment/`, so a document of this level cites another one
by its name alone.

Examples:

- `... see the $${/pkg/deployment/doc-package-manual:/cite-long}.`

- `... see the $${doc-package-manual:/cite}.`

- `... as defined in the $${/ref/ecss/e-st-40c-r1:/cite-long}.`

Some documents exist for each component of the package. The item UID of such a
component-specific document depends on its component. A package-level document
therefore has no single UID to cite. Cite component-specific documents through
a citation group, which yields one citation for each component.

A citation group collects the citations of the items which link to
`/pkg/component` with the @`/spec/citation-member-role:/spec-name`. The
`citation-group-key` attribute of the link gives the key of the group. The
specification items in `spec/pkg/template/<component>/` define the available
keys. Use the following syntax:

- Flat citation group: `$${/pkg/component:/cite-group:<citation-group-key>}`

- Citation group as a list:
  `$${/pkg/component:/cite-group:<citation-group-key>,flat=0}`

A citation group which targets a @`/spec/pkg-sphinx-document:/spec-name` item
may address a specific area of the cited document. The `label`, `name`, and
`path` attributes of the link define this area.

Example:

- `... in the $${/pkg/component:/cite-group:<citation-group-key>}.`

A citation with an unknown item UID fails the build. The error names the item
UID and the document item it resolved against. A citation group with an unknown
key prints nothing. The build logs a warning and succeeds, so this mistake is
easy to miss.

### Reference documents

A reference item is the bibliographic record of a work. Use it to cite a work
which has no @`/spec/pkg-sphinx-document:/spec-name` item. Examples are a
standard, a research paper, and a document which the package builds with
another tool.

The item UID starts with `/ref/`. The rest of the path groups the works, for
example by standards body, by project, or by kind. Most reference items are in
the specification item directory `spec/ref/`.

The @`/spec/reference:/spec-name` item type defines these attributes:

- `title`: the title of the work. It may contain a variable substitution.

- `reference-type`: the kind of the reference, for example `manual` or
  `article`. It selects the refinement which defines the remaining attributes,
  for example @`/spec/reference-manual:/spec-name`.

- `work-url`: the location of the work. It may point to a document of the
  package or to a public location.

- `work-hash`: the SHA256 hash value of the work, or `null`.

To add a reference, create an item file in the group which fits the work. Cite
it as described in {ref}`DocumentCitations`.

### Component paths and inputs

To resolve build-time paths or inputs, use the relative component syntax:

- Path relative to the target directory of the document currently generated:
  `$${.:/component/documentation-directory:relpath %(*:/directory)}`

- Archive name: `$${.:/input/archive/file:basename}`. This is the standard
  idiom for referencing the built package archive's filename, used for example
  in the package manual introduction and inventory chapters.

### Conditional blocks

You can conditionally include documentation blocks using the
push/pop-enabled-by syntax:

Example in MyST:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
@@{.:/push-enabled-by:pkg.feature.qual}
This text is only included if qualification features are enabled.
@@{.:/pop-enabled-by}
```

```{raw} latex
\end{footnotesize}
```

## Code blocks and command examples

### Code blocks formatting

- In MyST, use `{code-block} <language>` syntax with the `linenos:` option for
  code blocks.

- In reST, use the `.. code-block:: <language>` directive with the `:linenos:`
  option.

- The LaTeX output renders code blocks at the normal font size, which often
  overflows the page width or spills onto extra pages. Wrap every code block in
  a `footnotesize` environment using raw LaTeX directives immediately before
  and after it.

Example in MyST:

```{raw} latex
\begin{footnotesize}
```

````{code-block} none
---
linenos:
---
```{raw} latex
\begin{footnotesize}
```

```{code-block} c
---
linenos:
---
int main( void )
{
  return 0;
}
```

```{raw} latex
\end{footnotesize}
```
````

```{raw} latex
\end{footnotesize}
```

### Command-line examples

For commands executed in a shell:

- Use MyST `{code-block} none` (or reST `.. code-block:: none`) with the
  `linenos:` option.

- Prefix command lines with `$$` to distinguish them from the command output.

- For documentation variables or placeholders used in command-line examples,
  use the `$${MY_PLACEHOLDER}` notation (upper case, like a shell variable). In
  case copy and paste is blindly used, this still gives a valid shell syntax
  with hopefully undefined variables.

Example in MyST:

```{raw} latex
\begin{footnotesize}
```

````{code-block} none
---
linenos:
---
Let `$$$${foobar_directory}` be the foobar directory. Execute the following
command in a shell:

```{code-block} none
---
linenos:
---
$ cd $$$${foobar_directory}/some/more
$ command with args
output of command
```
````

```{raw} latex
\end{footnotesize}
```

## Tables

The LaTeX output renders reST tables at their natural column widths, which
often overflows the page width. For every `.. table::` directive:

- Always add the `:class: longtable` option.

- Always add a `:widths:` option with one integer per column, and make the
  integers sum to 100.

Example:

```{raw} latex
\begin{footnotesize}
```

```{code-block} rst
---
linenos:
---
.. table::
    :class: longtable
    :widths: 80,20

    +---------+---------+
    | Column1 | Column2 |
    +=========+=========+
    | Value1  | Value2  |
    +---------+---------+
```

```{raw} latex
\end{footnotesize}
```

## Diagrams

Diagrams are generated from source rather than committed as raw images.

- Store diagram sources in `src/common/images/`, using the `.puml` extension
  for [PlantUML](https://plantuml.com/) and the `.dot` extension for
  [Graphviz](https://graphviz.org/). The `Makefile` in that directory builds a
  `.png` and a `.pdf` for every diagram source found there.

- Every diagram source shall start with the same license and copyright header
  as other files, written as PlantUML/Graphviz comments starting with `'`
  (PlantUML) or `//` (Graphviz) on a line by itself.

- Reference a diagram from a document with a `{figure}` directive using a `.*`
  extension, so the correct format is picked for each output. For example:

  ```{raw} latex
  \begin{footnotesize}
  ```

  ````{code-block} none
  ---
  linenos:
  ---
  ```{figure} ../images/my-diagram.*
  ---
  alt: A short description of what the diagram shows
  width: 70%
  ---
  Diagram caption
  ```
  ````

  ```{raw} latex
  \end{footnotesize}
  ```

- Use a PlantUML mindmap (`@startmindmap`) to show an item's attributes
  alongside the activities used to write it, as in
  `action-requirement-workflow.puml` and `interface-item-workflow.puml`. Use a
  PlantUML component diagram (`@startuml` with `[Component]` boxes) to show
  system components and the relationships between them, as in
  `workspace-application-components.puml`.

## Local validation and formatting

Before submitting your documentation changes, perform formatting checks and
verify the documentation build locally.

### Auto-formatting

Format all `*.md` files using the `mdformat` tool to ensure it adheres to the
wrapping and layout rules configured in `.mdformat.toml`. Run the following
command:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ mdformat src/sdh/documentation.md
```

```{raw} latex
\end{footnotesize}
```

To only check the formatting without modifying the file:

```{raw} latex
\begin{footnotesize}
```

```{code-block} none
---
linenos:
---
$ mdformat --check src/sdh/documentation.md
```

```{raw} latex
\end{footnotesize}
```

### Verification build

To verify that the documentation builds correctly and all references are
resolved, run `make`.
