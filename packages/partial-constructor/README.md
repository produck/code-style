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
choose it.

> **Rationale**
> `index.mjs` carries no mark because the platform, not this document, fixes
> its name; the single file of the exception carries none because its author
> names it.

## 5. Symbol System

**SYM-1.** An internal member key MUST be a `Symbol`, and MUST NOT be a
string.

**SYM-2.** A symbol MUST be defined in the `_Symbol.mjs` of the module that
owns it. That definition is the single source of truth for its meaning, and
every reference MUST resolve to it.

**SYM-3.** A `Symbol` key MUST be assigned one of the three levels below.

| Level     | Descriptor | Instance table | Static table |
| --------- | ---------- | -------------- | ------------ |
| private   | `.#name`   | `I`            | `S`          |
| protected | `.$name`   | `$I`           | `$S`         |
| abstract  | `._name`   | `_I`           | `_S`         |

**SYM-4.** A `Symbol` key MUST be given the narrowest level that satisfies
all of its readers.

**SYM-5.** A method symbol MUST end with `()`; a field symbol MUST NOT.

**SYM-6.** A `Symbol` key that holds a constructor MUST end with `_CTOR`.

**SYM-7.** An abbreviation MUST come from a closed whitelist. The whitelist
currently contains `CTOR` and nothing else; adding an entry is a change to
this document.

> **Rationale**
> Directory structure does not shorten a name, so abbreviations are needed; an
> open set is a private cipher. A closed set is what lets every reader learn it
> once.

**SYM-8.** An exported table MUST be frozen, recursively, before it leaves
the module.

**SYM-9.** `_Symbol.mjs` MUST be a pure leaf. It MUST NOT import any module
of the same package; it MAY import a third-party helper.

**SYM-10.** Only the module that owns a key may declare it. A derived
constructor MAY override a protected member declared within its own family.
A member reached from outside the package MUST be public or abstract.

**SYM-11.** A subject export MUST NOT carry a _symbol table_.

## 6. Aliases and the Reference Graph

**ALI-1.** A module that owns keys SHOULD export an _alias table_ `A` from
`_Symbol.mjs`. `A` MUST contain only that module's own keys.

**ALI-2.** A module that borrows SHOULD export a _borrowed alias table_ `_A`
from `_External.mjs`. `_A` MUST contain only the tables the module borrows
directly. A borrowed table whose name is already short MUST be imported by
name instead of being aliased.

**ALI-3.** An alias MUST NOT be introduced unless it is shorter than the
expression it replaces. Between two qualifying forms, prefer the one with
the smaller product of name length and number of consumption sites.

> **Rationale**
> The criterion is deliberately quantitative. It is what rejects an alias that
> is longer than its original, and what keeps an alias from being applied to a
> name that is used once.

**ALI-4.** A module MUST obtain every own key from its own `./_Symbol.mjs`
and every borrowed table from its own `./_External.mjs`. A direct import of
another module's `_Symbol.mjs` MUST be confined to `_External.mjs`.

**ALI-5.** `A` and `_A` MUST be applied at the definition site as well: once
a key has an alias, the owning module MUST use the alias when it reads the
key itself.

**ALI-6.** The package's reference graph MUST be acyclic. `_Symbol.mjs`
defines and references nothing; every upward reference MUST be made in
`_External.mjs`.

> **Rationale**
> The leaf property is what makes a cycle structurally impossible rather than
> merely avoided. A module that reaches upward from its own _symbol table_
> violates the property even when no cycle exists today, because the next
> borrowed table closes one.

**ALI-7.** A symbol's descriptor and its alias key are two ledgers of the
same fact. Renaming one without renaming the other MUST be treated as a
defect.

## 7. Abstract and Concrete

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

## 8. Construction

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

## 9. Public Surface

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

## 10. Naming

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

## 11. Conformance

**CON-1.** A package conforms only if all of the following hold: its
reference graph is acyclic; every subject export carries constructors only;
no _symbol table_ imports anything else from the package; every abstract
member declares a contract; every borrowed table is re-exported from
`_External.mjs`.

## 12. Open Questions

- The single-file exception has a three-condition criterion but no threshold
  for "extreme simplification". Whether the criterion is sufficient, or
  needs a stated maximum size, is not settled.
- The alias criterion is quantitative, but the boundary between "many
  consumption sites" and "few" is left to review.
