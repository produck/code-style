# Directory-Style Partial-Constructor Organization Specification

> Engineering conventions for assembling a constructor from a directory of
> small, single-purpose modules instead of accumulating one large file.

## 1. Status of This Document

This document is the directory-style partial-constructor organization
specification. Adopting documents cite it by that name.

It is a draft. It generalizes conventions that an existing implementation
arrived at over time and then compressed into a set of current conclusions.
**Provisional** marks a rule that has been adopted in practice but whose
criterion is not yet settled.

A defined term whose everyday sense differs from the sense given in
Section 3 is written in _italics_ at every occurrence, so that a term of
art stays distinguishable from ordinary prose.

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD
NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted
as described in RFC 2119 and RFC 8174.

Enforcement is by convention alone. No rule here is checked by the language
or by a compiler, and none is expected to be. A convention may instead be
checked in tests, to whatever depth a supporting base provides, and that
checking is erased from the shipped artifact. This document names no such
base and requires none.

Every requirement is addressed to the engineer who writes or reviews the
code. Where a rule is not self-evidently right, the criterion that produced
it follows as **Rationale**. A rule whose criterion has been lost decays
into folklore, so the criterion is normative context rather than
decoration.

## 2. Scope

- The shape of a module: its directories, its files, and the references
  between them.
- The internal member-key system and its visibility levels.
- How a constructor is split into an abstract subject and one or more
  concrete counterparts.
- How an object is constructed, and how a dependency is attached after
  construction.
- Which parts of a module are public surface and which are maintenance
  surface.
- What enforces a rule: convention, and optionally a checking base whose
  checks are erased before the code ships.

## 3. Definitions

A subject is described in three layers. Where the thing under discussion is
unambiguous, the rules use the short forms constructor, directory, module,
and member. The sections below are ordered from the outermost scope
inward, and so is each list.

### Package

**Package** — the publishable unit that contains one or more modules. A
constraint that no single module can satisfy alone is stated of the
package.

- **Package export** — the surface a package exposes to its consumers.

### Subject

**Subject** — the unit of design: one constructor, which the module is
dedicated to. A subject is either abstract or concrete.

- **Subject constructor** — the core layer: the subject as a constructible
  value.
- **Subject directory** — the form layer: the directory that defines the
  subject constructor and every part of its assembly.
- **Subject module** — the synthesis layer: the subject constructor, its
  subject directory, and the other elements of the assembly, taken
  together.
- **Subject export** — `index.mjs`, the surface a module exposes to the rest of
  the package.
- **Abstract** — `_Abstract.mjs`, the abstract subject: it declares the
  contract and is not itself constructible.
- **Concrete** — `_Concrete.mjs`, the concrete subject: a constructible
  derivation of the abstract one.
- **Symbol table** — `_Symbol.mjs`, the one place where a module defines its
  own member keys.
- **Subject member** — one member of a subject, reached through one key: a
  string when the member is public, a `Symbol` otherwise.
- **Alias table** — `A`, short names for the module's own keys.
- **Borrowing table** — `_External.mjs`, the one place where a module
  imports keys that other modules own.
- **Borrowed alias table** — `_A`, short names for the borrowed tables.

### Member accessibility

A member's level is the answer to one question: who may reach it.

| Level     | Form               | Reached by                       |
| --------- | ------------------ | -------------------------------- |
| private   | `Symbol('.#name')` | the subject itself               |
| protected | `Symbol('.$name')` | any other subject in the package |
| abstract  | `Symbol('._name')` | the concrete                     |
| public    | `'name'`           | any consumer                     |

**Internal** — the private and protected levels together. Internal means the
package is the limit of reach: an internal key is never carried by the
package export. Both internal levels are keyed by a `Symbol`; an abstract
member is keyed by a `Symbol` too, yet it is the contract surface, not an
internal member.

**Exported** — the abstract level alone. An abstract key is carried by the
package export, so the concrete can reach the members it must implement even
from outside the package. No other level crosses the package boundary.

## 4. Subject Module Layout

**LAY-1.** A module MUST define exactly one subject constructor.

**LAY-2.** A _subject directory_ MUST contain `index.mjs` and `_Symbol.mjs`.

**LAY-3.** A _subject directory_ MUST contain exactly one of `_Abstract.mjs` and
`_Concrete.mjs`. The two MUST NOT coexist in the same directory. The kind of
the subject follows from which file is present: `_Abstract.mjs` means an
abstract subject, `_Concrete.mjs` means a concrete subject.

**LAY-4.** A _subject directory_ MUST contain `_External.mjs` if and only if
the module borrows keys or tables owned by another module.

**LAY-5.** A _subject directory_ MAY contain additional files or directories,
each owning one named concern. Such an element MUST be declared outside the
rules of this document, and the exemption MUST be recorded where it lives.

**LAY-6.** The directory path IS the namespace. Short names MUST NOT be made
globally unique; the same name in two modules denotes two different things.

> **Rationale**
> Local names stay short only if collisions are tolerated, and a collision is
> harmless when the path disambiguates it. Because the module is the namespace,
> a _symbol table_ MUST NOT limit its own key count.

**LAY-7.** A derived constructor MUST be placed in a directory parallel to,
and a sibling of, its abstract base's _subject directory_. It MUST NOT be
nested inside that directory.

**LAY-8.** Nesting downward inside a _subject directory_ MUST be reserved for
subjects that the outer one uses; a subject that the outer one does not use
MUST NOT be placed inside its directory. Being used is necessary for nesting
and not sufficient for it.

> **Rationale**
> The tree is read from the outside in: a directory states what it is about,
> and everything below it is detail a reader may leave unread. Nesting by use is
> what keeps each level answerable on its own, instead of unfolding every part
> of the subject at the top.

**LAY-9.** Single-file exception (**provisional**). A module MAY be one
`PascalCase.mjs` file placed beside the _subject directories_, if and only if
all three of the following hold: nothing derives from it, it owns no key,
and it needs no independent subject export. Owning even one key
disqualifies it.

> **Rationale**
> The three conditions are exactly the parts that a directory would add. The
> exception trades structure for readability, so it MUST NOT be granted to a
> module that has any of them.

**LAY-10.** A filename that begins with `_` is a reserved file of this
document. Its name is fixed and MUST NOT be replaced: the author does not
choose it. The reserved files are:

| File            | Carries               | Rule  |
| --------------- | --------------------- | ----- |
| `_Symbol.mjs`   | the _symbol table_    | LAY-2 |
| `_Abstract.mjs` | the abstract subject  | LAY-3 |
| `_Concrete.mjs` | the concrete subject  | LAY-3 |
| `_External.mjs` | the _borrowing table_ | LAY-4 |

> **Rationale**
> `index.mjs` carries no mark because the platform, not this document, fixes
> its name; the single file of the exception carries none because its author
> names it.

## 5. Symbol System

**SYM-1.** An internal member key MUST be a `Symbol`, and MUST NOT be a
string.

**SYM-2.** A `Symbol` key MUST be assigned one of the three levels below.

| Level     | Descriptor | Instance table | Static table |
| --------- | ---------- | -------------- | ------------ |
| private   | `.#name`   | `I`            | `S`          |
| protected | `.$name`   | `$I`           | `$S`         |
| abstract  | `._name`   | `_I`           | `_S`         |

**SYM-3.** A `Symbol` key MUST be given the narrowest level that satisfies
all of its readers.

**SYM-4.** A method symbol MUST end with `()`; a field symbol MUST NOT.

**SYM-5.** Every segment of a `Symbol` consumption expression MUST be written
in all uppercase, with words separated by `_`. A segment MAY be abbreviated
only through the closed whitelist.

**SYM-6.** A `Symbol` key that holds a constructor MUST end with `_CTOR`.

**SYM-7.** An abbreviation MUST come from a closed whitelist. The whitelist
currently contains `CTOR` and nothing else; adding an entry is a change to
this document.

> **Rationale**
> Directory structure does not shorten a name, so abbreviations are needed; an
> open set is a private cipher. A closed set is what lets every reader learn it
> once.

**SYM-8.** Inside a table, an author MAY nest namespaces freely in order to
group keys by category, and the category vocabulary is not fixed by this
document. Only a leaf is a key; every node above it is a namespace.

**SYM-9.** Only the module that owns a key may declare it. A derived
constructor MAY override a protected member declared within its own family.
A member reached from outside the package MUST be public or abstract.

**SYM-10.** A symbol's descriptor and its alias key are two ledgers of the
same fact. Renaming one without renaming the other MUST be treated as a
defect.

**SYM-11.** An alias is evaluated after the fact: it is justified when the full
expression would otherwise break a coding convention (the line width among
them), and when every reference resolves to the key it names. Whether to set
one is not settled in advance; aliases are appended as the need appears.

> **Rationale**
> The question an alias answers is not how much it saves, but whether the code
> can be written within the conventions it must meet at all. It is therefore
> judged where it is used, not approved where it is declared.

**SYM-12.** An alias is an alternative to the full expression, not a
replacement for it: the site that reads a key MAY use either, as it needs.

## 6. Symbol Table

**STB-1.** A symbol MUST be defined in the `_Symbol.mjs` of the module that
owns it. That definition is the single source of truth for its meaning, and
every reference MUST resolve to it.

**STB-2.** `_Symbol.mjs` MUST NOT export a top-level space other than the six
tables of SYM-2 and `A`.

**STB-3.** `_Symbol.mjs` MUST be a pure leaf. It MUST NOT import a module of
the same package, and it MUST NOT reference another _symbol table_, in any
package; every reference to a key that another module owns MUST be made in
`_External.mjs`. A helper MAY be imported; a helper is a function, not a key
or a table.

**STB-4.** An exported table MAY be frozen, recursively, before it leaves the
module.

> **Rationale**
> The tables are shared objects once they leave the module, so a stray write
> would add a key to every reader's view; freezing turns that write into a
> failing assignment. The cost is a recursive freeze at definition time and the
> helper it needs imported into `_Symbol.mjs`.

**STB-5.** A module that owns keys SHOULD export an _alias table_ `A` from
`_Symbol.mjs`. `A` MUST contain only that module's own keys.

**STB-6.** The structure of `A` is not required to match the paths of the
keys it aliases.

## 7. External Table

**ETB-1.** A module that borrows SHOULD export a _borrowed alias table_ `_A`
from `_External.mjs`. `_A` MUST contain only the tables the module borrows
directly. A borrowed table whose name is already short MUST be imported by
name instead of being aliased.

**ETB-2.** A module MUST take every key it owns through its own
`./_Symbol.mjs`, and every key it borrows through its own `./_External.mjs`.
An import of another module's `_Symbol.mjs` MUST be written in the importing
module's own `_External.mjs` and nowhere else, and the table so imported MUST
be re-exported from that file. Another module's `_External.mjs` MUST NOT be
imported.

**ETB-3.** The package's reference graph MUST be acyclic, and every upward
reference MUST be made in `_External.mjs`.

> **Rationale**
> The leaf property is what makes a cycle structurally impossible rather than
> merely avoided. A module that reaches upward from its own _symbol table_
> violates the property even when no cycle exists today, because the next
> borrowed table closes one.

## 8. Abstract and Concrete

**ABS-1.** A constructor that a downstream author is expected to derive from
MUST be declared abstract through the shared abstract layer. Documenting it
as abstract in prose MUST NOT be used as a substitute.

**ABS-2.** Every abstract member MUST declare a contract for what it returns,
including whether it MAY answer a promise.

**ABS-3.** The contract surface of a family is exactly its abstract members.
Those members are the concrete's obligation; every other member in the
module is maintenance surface.

**ABS-4.** An abstract subject MUST NOT be constructible directly.
Constructing it MUST fail.

**ABS-5.** A derived constructor MAY be produced without `class` and
`extends` syntax, by any means that yields the same derivation relation.
Doing so is permitted and does not conflict with the purpose of this
specification, but it is not RECOMMENDED: it is laborious to write and to
read. The RECOMMENDED form is `class ... extends`.

## 9. Construction

**CTR-1.** A constructor that needs the construction target MUST capture it
with `new.target`, and MUST NOT read `this.constructor`. Which member holds
the captured target is not fixed by this document.

> **Rationale**
> The target is a fact about one moment, the call that built the object, and
> `new.target` is the only expression that reports it at that moment.
> `this.constructor` is a property lookup that any assignment can shadow, so it
> reports what the prototype chain says rather than what ran. An internal member
> is a safe home for the reference because it is not carried by the package
> export.

**CTR-2.** A constructor MUST accept the smallest argument set that is
stable for the whole family. A secondary dependency MUST NOT be added as a
further constructor argument.

**CTR-3.** A dependency that cannot be passed to the constructor MUST be
attached through an explicit protected member before the object is reachable
from the public surface.

**CTR-4.** A guard MUST be justified by a reachable state, not by caution.
Where an operation is single-shot because its only caller is single-shot,
the guard MUST NOT be added; where the set of callers is not closed, the
guard MUST be.

> **Rationale**
> An unreachable guard is a dead branch, and a dead branch survives review as
> if it were protection. The criterion for adding one is therefore the caller
> set, not the risk.

## 10. Public Surface

**PUB-1.** The package export MUST be the only export space of a package, and
everything a consumer may reach MUST be organized there. A raw _symbol table_
or _borrowing table_ MUST NOT be carried by it.

**PUB-2.** A member that the concrete must read or write beyond the abstract
contract MUST be exposed through the package export as a grouped symbol
namespace containing abstract keys only.

**PUB-3.** The _alias tables_ MUST NOT be reachable from the package export.

**PUB-4.** When the concrete needs a capability that only an internal member
can provide, promoting the capability to a public member SHOULD be preferred
over exposing the internal member.

> **Rationale**
> Symbols are the maintenance surface; public members are the promise surface.
> Exposing an internal member converts an internal change into a breaking
> change.

## 11. Subject Export

**SUB-1.** A subject export MUST carry the subject constructor, as `Abstract`
when the subject is abstract and as `Concrete` when it is concrete.

**SUB-2.** A subject export MUST NOT carry a _symbol table_ or a _borrowing
table_.

**SUB-3.** A subject that the module uses MUST be carried by the subject
export as a namespace under that subject's own name.

**SUB-4.** A subject export MAY carry anything else, provided that no other
export takes the name `Abstract` or `Concrete`.

## 12. Naming

**NAM-1.** A name MUST be checked against the mainstream practice of its
domain before it is adopted. A name that is defensible only inside this
package MUST NOT be adopted.

**NAM-2.** A term that denotes one thing MUST NOT be used for another. Where
two concepts are close, the package MUST fix one word per concept and
record both.

**NAM-3.** A role name and the name of a concrete MUST be distinguished. A
role name carries the contract vocabulary and MUST NOT be renamed when the
concrete, the package, or the product changes.

**NAM-4.** A hook that the framework calls MUST be named as a verb phrase; a
hook that the framework reads MUST be named as a noun phrase.

**NAM-5.** A package that introduces domain vocabulary MUST record it as a
glossary, one entry per term, stating the distinction that makes the term
necessary.

## 13. Conformance

**CON-1.** A package conforms only if all of the following hold: its
reference graph is acyclic; every subject export carries its subject
constructor; no _symbol table_ imports anything else from the package; every
abstract member declares a contract; every borrowed table is re-exported from
`_External.mjs`.

## 14. Open Questions

- The single-file exception has a three-condition criterion but no threshold
  for "extreme simplification". Whether the criterion is sufficient, or
  needs a stated maximum size, is not settled.
- Which coding conventions can justify an alias is not enumerated, and where
  the boundary lies is left to review.
