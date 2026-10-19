# Import Statement Writing Specification

> Engineering conventions for writing the import statements of a module: what
> a statement may name, which file it may reach, and what it may not express.

## 1. Status of This Document

This document is the import statement writing specification. Adopting
documents cite it by that name.

It is a draft. It generalizes the import graph of an existing implementation,
where a module reaches another module's keys through one file and another
module's subject through another.

**Provisional** marks a rule that has been adopted in practice but whose
criterion is not yet settled.

A word defined in Section 3 is written in _italics_ where it is meant as that
term, and left plain where it is meant in its everyday sense. The sibling
specification counts the marked occurrences beside each definition; this draft
leaves those counts out while the rule set is still moving.

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT,
RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as
described in RFC 2119 and RFC 8174.

Enforcement is by convention alone. No rule here is checked by ECMAScript or
by a compiler, and none is expected to be.

Every requirement is addressed to the engineer who writes or reviews the code.
Where a rule is not self-evidently right, the criterion that produced it
follows as **Rationale**. Where a rule rests on a judgment the code does not
state, the question it rests on follows as **Boundary**.

## 2. Scope

An import statement is where a module declares what it depends on, so it is
also the only place where that dependence is visible to a reader, and the only
place where a decision taken inside another module becomes binding. A
statement that names a file directly turns that file's path into a promise;
one that reaches past an export space hides a dependence the owner can no
longer rearrange.

This document covers:

- The statements that bring another module's keys and subject into this one,
  and the file each statement is written in.
- Which of another module's files a statement is allowed to name.
- The order the statements appear in, and how they are separated.
- The form each statement takes.

This document does not cover:

- Where a module and its files are placed. The sibling specification states
  that.
- A dynamic import, `import()`. It runs at a moment this document cannot see,
  it may have side effects, and those effects are the runtime's to settle.

## 3. Definitions

**Standard library** — the modules the language and the platform provide: the
farthest thing a module can name, and the one no edit to this package changes.

**Third-party package** — a package this one depends on: outside this package,
and changed only by upgrading it.

**Local file** — a file inside this package: the nearest thing a module names,
and the one the package is free to move.

## 4. Import Order

**ORD-1.** The import statements of a module MUST run from the abstract to the
concrete: the farther a name is from this module's own code, the earlier its
statement appears.

> **Boundary**
> The question is which statement is farther from this module's code. Between
> two domains, ORD-2 answers it. Inside the _standard library_ and inside a
> _third-party package_ the author answers it. Between _local file_ statements
> the depth of `../` is the scale: a statement that reaches further out comes
> first.

**ORD-2.** The statements MUST be divided into five domains, and the domains
MUST appear in this sequence: the type domain, the side-effect domain, the
_standard library_, then a _third-party package_, then a _local file_.

**ORD-3.** The statements of one domain MUST be contiguous, and one blank line
MUST separate one domain from the next.

**ORD-4.** A type-only statement, `import type … from '…'`, forms the type
domain: it is not divided into domains of its own, and inside it the statements
run from the far to the near.

**ORD-5.** A side-effect statement, `import '…'`, forms the side-effect domain
the same way: not divided, and running from the far to the near.

## 5. Import Form

**FRM-1.** An import statement SHOULD take the namespace form — `import * as
name from '…'` — unless the module it names states how it is to be imported.

> **Rationale**
> The namespace form keeps the names a module brings out of the way of the names
> this module already has: every use is qualified, so two modules may export the
> same name without either being renamed. A named import puts the name into this
> module's own scope, and a collision has to be undone by hand.

> **Boundary**
> The question is whether the target fixes the form. No criterion is given for
> answering it: the situation decides, and what reads better for the person
> writing the code comes before consistency between modules. The goal above —
> no name collides — holds whichever form is chosen.
