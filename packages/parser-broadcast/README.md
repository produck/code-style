# Directory Parser Broadcast Specification

> Engineering conventions for making every parser reachable through one fixed
> import, without a resolver and without a monorepo.

## 1. Status of This Document

This document is the directory parser broadcast specification. Adopting
documents cite it by that name.

It is a draft. It records a pattern that one implementation arrived at while
trying to stop parser imports from drifting, and it deliberately stops short of
the parts that implementation has not settled.

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT,
RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as
described in RFC 2119 and RFC 8174.

Enforcement is by convention alone. No rule here is checked by ECMAScript or by
a compiler. A supporting base MAY check any of them in tests; this document
names no such base and requires none.

Where a rule is not self-evidently right, the criterion that produced it
follows as **Rationale**. Where a rule rests on a judgment the code does not
state, the question follows as **Boundary**.

## 2. Scope

A parser is a small function that inspects a value and either rejects it or
answers something about it. In a code base of any size the same handful of
parsers is wanted from many places, and the imports that reach them are the
problem: each consumer writes a relative path whose depth depends on where the
parser happens to sit today. Nothing about that path is reviewable. A reviewer
sees `../../../Parser.mjs` and learns neither which parser is used nor whether
the depth is right.

This document fixes the import instead of the parser. Every consumer imports
the table of its own directory; every table re-exports its parent's. A parser
may then move up or down without touching a single consumption point, and the
path a reviewer reads is always the same shape.

The broadcast is scoped to a package. It assumes no workspace layout, no path
alias, no build step, and no resolver feature beyond plain relative specifiers.

## 3. Definitions

**Parser** — a function of the shape below, exported from a `_Parser.mjs`.

**Consumption point** — a statement that reads a parser in order to call it.

**Broadcast table** — the `_Parser.mjs` of one _module directory_.

**Module directory** — a directory that holds modules. The broadcast is
organized by it, not by module.

## 4. The Parser Shape

**PBR-1.** A parser MUST have the shape `(value, ...args) => any`.

> **Rationale**
> The first argument is the value under inspection. Everything after it is
> context a parser may use to build its message, such as the argument position
> being reported. The return type is `any` on purpose: a parser is not required
> to answer a boolean, and forcing one would rule out the most useful parsers.

**PBR-2.** A parser MAY transform the value it is given. A consumption point
MUST NOT assume that a parser returns its argument.

> **Rationale**
> The model is `parseInt(string)`: the caller passes text and receives a
> number, and `parseInt('12px')` is `12` rather than a rejection. A parser
> named for a shape is free to normalize that shape on the way out. Writing
> the shape rule as "validate and return the input" would quietly forbid this
> class of parser, so the document states the weaker shape and lets each
> parser say what it answers.

**PBR-3.** A consumption point MUST use the return value of a parser, and MUST
NOT reuse the value it passed in.

> **Rationale**
> This is the failure PBR-2 makes possible. When the argument is used instead
> of the answer, a transformation is discarded and nothing reports it: the
> code compiles, the tests that exercise the happy path pass, and the
> normalized value never reaches the caller. The fix is local and cheap, but
> only if the reviewer is looking for it.

**PBR-4.** A parser MAY answer `undefined`. That is how a check that carries no
value is expressed, and it is the only difference between a parser and an
assert.

> **Rationale**
> An assert is not a second kind of thing: it is a parser whose answer is
> never used. Calling it a parser keeps one shape, one file, and one import.
> The distinction survives where it matters — at the consumption point, where
> a discarded answer is either deliberate or a defect.

## 5. The Broadcast

**PBR-5.** The unit of the broadcast is the _module directory_. A table
describes one directory, not one module and not one package.

**PBR-6.** `_Parser.mjs` is a reserved file name of this document. Its name is
fixed and MUST NOT be replaced; the author does not choose it.

> **Rationale**
> The leading mark says the file is structure, not content, and it is the same
> mark other reserved files of a supporting base carry. An adopting document
> SHOULD add `_Parser.mjs` to its own table of reserved files.

**PBR-7.** A _module directory_ that defines a parser for use elsewhere, or
that contains a _consumption point_, MUST carry a _broadcast table_.

**PBR-8.** A _broadcast table_ MUST re-export its parent's table, and MUST do
so before declaring its own parsers:

```js
export * from '../Parser.mjs';

export function HttpMethod(value) { ... }
```

The table at the root of the package has no parent and MUST NOT begin with a
re-export.

**PBR-9.** A _consumption point_ MUST import the table of its own directory as
a namespace, and MUST NOT name any other directory's table:

```js
import * as P from './Parser.mjs';
```

> **Rationale**
> A fixed specifier is the whole point. It removes the depth arithmetic, it
> makes every consumption point look the same in review, and it means a
> parser that moves between levels changes no caller. Naming the table `P` is
> a convention of the adopting package; what this document requires is that
> one name is used for it everywhere.

**PBR-10.** A directory that neither defines a parser nor contains a
_consumption point_, but that sits between the two, MUST still carry a table
that only re-exports its parent.

> **Boundary**
> The question is whether to allow a table to skip an empty level and
> re-export the nearest occupied ancestor instead. Skipping removes those
> one-line files, but the specifier then varies in depth with the current
> location of the definition, and every later move re-derives it. A pattern
> that exists to stop path drift MUST NOT reintroduce drift in the place a
> reviewer is least likely to check.

**PBR-11.** A parser MUST be placed at the nearest common ancestor of its
_consumption points_.

> **Rationale**
> A parser reachable by two subtrees has to sit above both, or one of them
> cannot reach it at all through PBR-9. Placing it higher than the nearest
> common ancestor costs a needless level in every specifier below it; placing
> it lower makes some consumption point illegal.

## 6. Acyclicity

**PBR-12.** A _broadcast table_ MUST reference its parent and nothing else. It
MUST NOT reference a sibling, a child, or itself.

**PBR-13.** The broadcast graph MUST be acyclic.

> **Rationale**
> Two reasons, one mechanical and one structural. A cycle stops a bundler from
> concatenating the modules, so the tables become runtime objects rather than
> erased structure. And a cycle makes the initialization order of the tables
> observable, which turns a naming convenience into a source of load-order
> bugs. A rule that only points upward cannot produce either.

## 7. Naming

**PBR-14.** Parser names are the package's own business. This document
requires no registry, no prefix scheme, and no coordination between packages.

> **Rationale**
> The broadcast is a layout, not a vocabulary. A package that wants
> `HttpMethod` and a package that wants `Method` are both correct, and neither
> can break the other. What must be shared is the file name and the import
> specifier, and those are the two things this document fixes.

**PBR-15.** A declaration in a table MUST NOT reuse a name that an ancestor
table already exports.

> **Rationale**
> ECMAScript resolves this silently: the nearer declaration wins and nothing
> is reported. There is no error to catch, so the collision has to be caught
> in review, and it can only be caught if the rule says it is a defect. The
> practical guard is to prefer names that say what the parser answers over
> names that describe a category.

## 8. What a Reviewer Reads

Under this pattern a review of parser usage has two questions, and both are
answerable from the file under review:

- Does every consumption point import `./_Parser.mjs` and use the return value?
- Does the table it lands on expose the name, without merely shadowing a
  nearer one?

Everything else — where a parser lives, how deep the chain is, whether a level
is empty — is answered by the tables themselves, one level at a time, and none
of it appears in the code being reviewed.
