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
package export, while an abstract key is. Both internal levels are keyed by a
`Symbol`; an abstract member is keyed by a `Symbol` too, yet it is the
contract surface, not an internal member.

## 4. Subject Module Layout

**LAY-1.** A module MUST define exactly one subject constructor.

**LAY-2.** A module directory MUST contain `index.mjs` and `_Symbol.mjs`.

**LAY-3.** A module directory MUST contain exactly one of `_Abstract.mjs` and
`_Concrete.mjs`. The two MUST NOT coexist in the same directory.

**LAY-4.** A module directory MUST contain `_External.mjs` if and only if
the module borrows keys or tables owned by another module.

**LAY-5.** A module directory MAY contain additional files, each owning one
named concern — for example a parser, a checker, or an event vocabulary.
Such a file MUST be declared outside the rules of this document, and the
exemption MUST be recorded where the file lives.

**LAY-6.** The directory path IS the namespace. Short names MUST NOT be made
globally unique; the same name in two modules denotes two different things.

**Rationale.** Local names stay short only if collisions are tolerated, and
a collision is harmless when the path disambiguates it. Because the module
is the namespace, a _symbol table_ MUST NOT limit its own key count.

**LAY-7.** A derived constructor MUST be placed in a directory parallel to,
and a sibling of, its abstract base's subject directory. It MUST NOT be
nested inside that directory.

**LAY-8.** Nesting downward inside a module directory MUST be reserved for
constructors that are not in a derivation relation with the subject
constructor — for example an abstract member of the subject constructor's
own family.

**LAY-9.** Single-file exception (**provisional**). A module MAY be one
`PascalCase.mjs` file placed beside the subject directories, if and only if
all three of the following hold: nothing derives from it, it owns no key,
and it needs no independent subject export. Owning even one key
disqualifies it.

**Rationale.** The three conditions are exactly the parts that a directory
would add. The exception trades structure for readability, so it MUST NOT be
granted to a module that has any of them.

**LAY-10.** A filename that begins with `_` is a reserved file of this
document. Its name is fixed and MUST NOT be replaced: the author does not
choose it.

**Rationale.** `index.mjs` carries no mark because the platform, not this
document, fixes its name; the single file of the exception carries none
because its author names it.

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

**Rationale.** The criterion is deliberately quantitative. It is what
rejects an alias that is longer than its original, and what keeps an alias
from being applied to a name that is used once.

**ALI-4.** A module MUST obtain every own key from its own `./_Symbol.mjs`
and every borrowed table from its own `./_External.mjs`. A direct import of
another module's `_Symbol.mjs` MUST be confined to `_External.mjs`.

**ALI-5.** `A` and `_A` MUST be applied at the definition site as well: once
a key has an alias, the owning module MUST use the alias when it reads the
key itself.

**ALI-6.** The package's reference graph MUST be acyclic. `_Symbol.mjs`
defines and references nothing; every upward reference MUST be made in
`_External.mjs`.

**Rationale.** The leaf property is what makes a cycle structurally
impossible rather than merely avoided. A module that reaches upward from its
own _symbol table_ violates the property even when no cycle exists today,
because the next borrowed table closes one.

**ALI-7.** A symbol's descriptor and its alias key are two ledgers of the
same fact. Renaming one without renaming the other MUST be treated as a
defect.

## 7. Abstract and Concrete

**ABS-1.** A constructor that a downstream author is expected to derive from
MUST be declared abstract through the shared abstract layer. Documenting it
as abstract in prose MUST NOT be used as a substitute.

**ABS-2.** Every abstract member MUST declare a contract for what it returns,
including whether it MAY answer a promise.

**ABS-3.** An abstract subject MUST NOT be checked at load time and MUST NOT
block `extends` or any equivalent derivation. The check MUST happen lazily,
on the first access of a member that has no implementation.

**ABS-4.** Where a hard constraint is required, it MUST be requested
explicitly by wrapping the derived constructor, instead of relying on the
lazy check. The lazy check and the hard constraint are two different
guarantees and MUST NOT be conflated.

**ABS-5.** The contract surface of a family is exactly its `_I` and `_S`
members. Those members are the concrete's obligation; every other member in
the module is maintenance surface.

**ABS-6.** An abstract subject MUST NOT be constructible directly.
Constructing it MUST fail.

**ABS-7.** A derived constructor MAY be produced without `class` and
`extends` syntax, by any means that yields the same derivation relation.
Doing so is permitted and does not conflict with the purpose of this
specification, but it is not RECOMMENDED: it is laborious to write and to
read. The RECOMMENDED form is `class ... extends`.

## 8. Construction

**CTR-1.** A constructor MUST capture the construction target with
`new.target` and store it in the shared constructor key. It MUST NOT read
`this.constructor`.

**Rationale.** Static strategy hooks are resolved against the captured
construction target, so the mapping is fixed once, at construction time.
`this.constructor` is a property lookup on an instance and can be shadowed,
so it MUST NOT be used as a substitute.

**CTR-2.** A constructor MUST accept the smallest argument set that is
stable for the whole family. A secondary dependency MUST NOT be added as a
further constructor argument.

**CTR-3.** A dependency that cannot be passed to the constructor MUST be
attached through an explicit protected member before the object is reachable
from the public surface.

**CTR-4.** A static normalisation hook MUST be a method. It MUST NOT be a
getter whose value is the normalised result.

**Rationale.** The framework invokes hooks as calls. A getter answers a
value, so answering an array where a function is expected fails at the call
site.

**CTR-5.** A family whose constructor takes no argument MUST answer an empty
argument list from its parameter hook, so that no argument set need be
attached.

**CTR-6.** A value that a caller may change later MUST have exactly one write
point, and that write point MUST NOT be the constructor. A constructor MUST
NOT duplicate a value that the configuration surface already owns.

**CTR-7.** A guard MUST be justified by a reachable state, not by caution.
Where an operation is single-shot because its only caller is single-shot,
the guard MUST NOT be added; where the set of callers is not closed, the
guard MUST be.

**Rationale.** An unreachable guard is a dead branch, and a dead branch
survives review as if it were protection. The criterion for adding one is
therefore the caller set, not the risk.

## 9. Public Surface

**PUB-1.** The package export MUST be the only export space of a package, and
everything a consumer may reach MUST be organized there. A raw _symbol table_
or _borrowing table_ MUST NOT be carried by it.

**PUB-2.** A member that the concrete must read or write beyond the abstract
contract MUST be exposed through the package export as a grouped symbol
namespace containing `_I` and `_S` only.

**PUB-3.** The _alias tables_ MUST NOT be reachable from the package export.

**PUB-4.** When the concrete needs a capability that only an internal member
can provide, promoting the capability to a public member SHOULD be preferred
over exposing the internal member.

**Rationale.** Symbols are the maintenance surface; public members are the
promise surface. Exposing an internal member converts an internal change into
a breaking change.

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

- The abbreviation whitelist holds one entry. Whether it SHOULD stay closed
  at that size, or grow a fixed set, is not settled.
- The single-file exception has a three-condition criterion but no threshold
  for "extreme simplification". Whether the criterion is sufficient, or
  needs a stated maximum size, is not settled.
- The alias criterion is quantitative, but the boundary between "many
  consumption sites" and "few" is left to review.
