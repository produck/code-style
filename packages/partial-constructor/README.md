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

A word defined in Section 3 is written in _italics_ where it is meant as that
term, and left plain where it is meant in its everyday sense. Only the marked
occurrences are counted, so the count beside a definition in Section 3 is the
number of times the term is used as a term outside that section.

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD
NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted
as described in RFC 2119 and RFC 8174.

Enforcement is by convention alone. No rule here is checked by ECMAScript
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

- The shape of a _module_: its directories, its files, and the references
  between them.
- The _internal_ member-key system and its visibility levels.
- How a _constructor_ is split into an _abstract_ _subject_ and one or more
  _concrete_ counterparts.
- How an object is constructed, and how a dependency is attached after
  construction.
- Which parts of a _module_ are _public_ surface and which are maintenance
  surface.
- What enforces a rule: convention, and optionally a checking base whose
  checks are erased before the code ships.

## 3. Definitions

A subject is described in three layers. Where the thing under discussion is
unambiguous, the rules use the short forms constructor, directory, module,
and member. The sections below are ordered from the outermost scope
inward, and so is each list.

Each term below carries a superscript reference count: the number of marked
occurrences of the term outside this section. Section 3 defines and never
marks; the count is zero for a term that no rule uses.

### Package

**Package**<sup>14</sup> — the publishable unit that contains one or more
modules. A constraint that no single module can satisfy alone is stated of the
package.

- **Package export**<sup>3</sup> — the surface a package exposes to its
  consumers.

### Subject

**Subject**<sup>14</sup> — the unit of design: one constructor, which the module
is dedicated to. A subject is either abstract or concrete.

- **Subject constructor**<sup>12</sup> — the core layer: the subject as a
  constructible value.
- **Subject directory**<sup>15</sup> — the form layer: the directory that
  defines the subject constructor and every part of its assembly.
- **Subject module**<sup>27</sup> — the synthesis layer: the subject
  constructor, its subject directory, and the other elements of the assembly,
  taken together.
- **Subject export**<sup>6</sup> — `index.mjs`, the surface a module exposes to
  the rest of the package.
- **Abstract**<sup>19</sup> — `_Abstract.mjs`, the abstract subject: it declares
  the contract and is not itself constructible.
- **Concrete**<sup>8</sup> — `_Concrete.mjs`, the concrete subject: a
  constructible derivation of the abstract one.
- **Derivation**<sup>6</sup> — a subject built on another subject's contract,
  in the sense of `class ... extends`: it takes the abstract keys of that base
  as its obligation, and is either concrete, or abstract with that obligation
  not yet fully discharged.
- **Symbol table**<sup>7</sup> — `_Symbol.mjs`, the one place where a module
  defines its own member keys.
- **Subject member**<sup>12</sup> — one member of a subject, reached through one
  key: a string when the member is public, a `Symbol` otherwise.
- **Alias table**<sup>2</sup> — `A`, short names for the module's own keys.
- **Borrowing table**<sup>3</sup> — `_External.mjs`, the one place where a
  module imports keys that other modules own.
- **Borrowed alias table**<sup>1</sup> — `_A`, short names for the borrowed
  tables.

### Consumption

**Consumption side**<sup>0</sup> — everything that reads a key, as opposed to
the module that declares it.

- **Consumer**<sup>3</sup> — the code that depends on a subject module, by
  deriving from it or by using it.
- **Consumption point**<sup>5</sup> — one place in the code that reads a key.
- **Consumption expression**<sup>1</sup> — the expression written at such a
  place: the path of tables and namespaces, ending in the key.

### Member

A member's accessibility is the answer to one question: who may reach it. A
member sits at exactly one of the levels below.

- **Private**<sup>2</sup> — reached by the subject itself, and by nothing else.
- **Protected**<sup>4</sup> — reached by any other subject in the package.
- **Abstract member**<sup>6</sup> — reached by a derivation of the declaring
  subject.
- **Public**<sup>6</sup> — reached by any consumer; the member is keyed by a
  string, not a `Symbol`.

**Internal**<sup>7</sup> — the private and protected levels together. Internal
means the package is the limit of reach: an internal key is never carried by the
package export. Both internal levels are keyed by a `Symbol`; an abstract member
is keyed by a `Symbol` too, yet it is the contract surface, not an internal
member.

## 4. Subject Module Layout

**LAY-1.** A _module_ MUST define exactly one _subject constructor_.

**LAY-2.** A _subject directory_ MUST contain `index.mjs` and `_Symbol.mjs`.

**LAY-3.** A _subject directory_ MUST contain exactly one of `_Abstract.mjs` and
`_Concrete.mjs`. The two MUST NOT coexist in the same _directory_. The kind of
the _subject_ follows from which file is present: `_Abstract.mjs` means an
_abstract_ _subject_, `_Concrete.mjs` means a _concrete_ _subject_.

**LAY-4.** A _subject directory_ MUST contain `_External.mjs` if and only if
the _module_ borrows keys or tables owned by another _module_.

**LAY-5.** A _subject directory_ MAY contain additional files or directories,
each owning one named concern. Such an element MUST be declared outside the
rules of this document, and the exemption MUST be recorded where it lives.

**LAY-6.** The _directory_ path IS the namespace. Short names MUST NOT be made
globally unique; the same name in two _modules_ denotes two different things.

> **Rationale**
> Local names stay short only if collisions are tolerated, and a collision is
> harmless when the path disambiguates it. Because the _module_ is the
> namespace, a _symbol table_ MUST NOT limit its own key count.

**LAY-7.** A _derivation_ MUST be placed in a _directory_ parallel to, and a
sibling of, its _abstract_ base's _subject directory_. It MUST NOT be nested
inside that _directory_.

**LAY-8.** Nesting downward inside a _subject directory_ MUST be reserved for
_subjects_ that the outer one uses; a _subject_ that the outer one does not use
MUST NOT be placed inside its _directory_. Being used is necessary for nesting
and not sufficient for it.

> **Rationale**
> The tree is read from the outside in: a _directory_ states what it is about,
> and everything below it is detail the engineer may leave unread. Nesting by
> use is what keeps each _directory_ answerable on its own, instead of
> unfolding every part of the _subject_ at the top.

**LAY-9.** Single-file exception (**provisional**). A _module_ MAY be one
`PascalCase.mjs` file placed beside the _subject directories_, if and only if
all three of the following hold: nothing derives from it, it owns no key,
and it needs no independent _subject export_. Owning even one key
disqualifies it.

> **Rationale**
> The three conditions are exactly the parts that a _directory_ would add. The
> exception trades structure for readability, so it MUST NOT be granted to a
> _module_ that has any of them.

**LAY-10.** A filename that begins with `_` is a reserved file of this
document. Its name is fixed and MUST NOT be replaced: the author does not
choose it. The reserved files are:

| File            | Carries                  | Rule  |
| --------------- | ------------------------ | ----- |
| `_Symbol.mjs`   | the _symbol table_       | LAY-2 |
| `_Abstract.mjs` | the _abstract_ _subject_ | LAY-3 |
| `_Concrete.mjs` | the _concrete_ _subject_ | LAY-3 |
| `_External.mjs` | the _borrowing table_    | LAY-4 |

> **Rationale**
> `index.mjs` carries no mark because the platform, not this document, fixes
> its name; the single file of the exception carries none because its author
> names it.

## 5. Symbol System

**SYM-1.** An _internal_ member key MUST be a `Symbol`, and MUST NOT be a
string.

**SYM-2.** A `Symbol` key MUST be assigned one of the three `Symbol` levels
below.

| Level       | Descriptor | Instance table | Static table |
| ----------- | ---------- | -------------- | ------------ |
| _private_   | `.#name`   | `I`            | `S`          |
| _protected_ | `.$name`   | `$I`           | `$S`         |
| _abstract_  | `._name`   | `_I`           | `_S`         |

Who may reach a member at each level:

| Level       | Form               | Reached by                            |
| ----------- | ------------------ | ------------------------------------- |
| _private_   | `Symbol('.#name')` | the subject itself                    |
| _protected_ | `Symbol('.$name')` | any other subject in the package      |
| _abstract_  | `Symbol('._name')` | a derivation of the declaring subject |
| _public_    | `'name'`           | any consumer                          |

**SYM-3.** A `Symbol` key MUST be given the narrowest level that satisfies
all of its _consumption points_.

**SYM-4.** A method symbol MUST end with `()`; a field symbol MUST NOT.

**SYM-5.** Every segment of a `Symbol` _consumption expression_ MUST be
written in all uppercase, with words separated by `_`. A segment MAY be
abbreviated only through the closed whitelist.

**SYM-6.** A `Symbol` key that holds a _constructor_ MUST end with `_CTOR`.

**SYM-7.** An abbreviation MUST come from a closed whitelist. The whitelist
currently contains `CTOR` and nothing else; adding an entry is a change to
this document.

> **Rationale**
> Directory structure does not shorten a name, so abbreviations are needed; an
> open set is a private cipher. A closed set is what lets the engineer learn
> it once.

**SYM-8.** Inside a table, an author MAY nest namespaces freely in order to
group keys by category, and the category vocabulary is not fixed by this
document. Only a leaf is a key; every node above it is a namespace.

**SYM-9.** Only the _module_ that owns a key may declare it. A _derivation_
MAY override a _protected_ _member_ declared within its own family.
A _member_ reached from outside the _package_ MUST be _public_ or _abstract_.

**SYM-10.** A symbol's descriptor and its alias key are two ledgers of the
same fact. Renaming one without renaming the other MUST be treated as a
defect.

**SYM-11.** An alias is evaluated after the fact: it is justified when the full
expression would otherwise break a coding convention (the line width among
them), and when every _consumption point_ resolves to the key it names.
Whether to set one is not settled in advance; aliases are appended as the need
appears.

> **Rationale**
> The question an alias answers is not how much it saves, but whether the code
> can be written within the conventions it must meet at all. It is therefore
> judged where it is used, not approved where it is declared.

**SYM-12.** An alias is an alternative to the full expression, not a
replacement for it: a _consumption point_ MAY use either, as it needs.

## 6. Symbol Table

**STB-1.** A symbol MUST be defined in the `_Symbol.mjs` of the _module_ that
owns it. That definition is the single source of truth for its meaning, and
every reference MUST resolve to it.

**STB-2.** `_Symbol.mjs` MUST NOT export anything other than the six
tables of SYM-2 and `A`.

**STB-3.** A key MUST have at least one _consumption point_. A key that is
written and never read MUST be removed.

> **Rationale**
> A key that nothing reads is not maintenance surface but dead weight: it
> survives review because a declaration looks like a decision. The criterion is
> the _consumption point_, the same one that decides whether a guard is added.

**STB-4.** `_Symbol.mjs` MUST be a pure leaf. It MUST NOT import a _module_ of
the same _package_, and it MUST NOT reference another _symbol table_, in any
_package_; every reference to a key that another _module_ owns MUST be made in
`_External.mjs`. A helper MAY be imported; a helper is a function, not a key
or a table.

**STB-5.** An exported table MAY be frozen, recursively, before it leaves the
_module_.

> **Rationale**
> The tables are shared objects once they leave the _module_, so a stray write
> would add a key wherever they are consumed; freezing turns that write into a
> failing assignment. The cost is a recursive freeze at definition time and the
> helper it needs imported into `_Symbol.mjs`.

**STB-6.** A _module_ that owns keys SHOULD export an _alias table_ `A` from
`_Symbol.mjs`. `A` MUST contain only that _module_'s own keys.

**STB-7.** The structure of `A` is not required to match the paths of the
keys it aliases.

## 7. External Table

**ETB-1.** A _module_ that borrows SHOULD export a _borrowed alias table_ `_A`
from `_External.mjs`. `_A` MUST contain only the tables the _module_ borrows
directly. A borrowed table whose name is already short MUST be imported by
name instead of being aliased.

**ETB-2.** A _module_ MUST take every key it owns through its own
`./_Symbol.mjs`, and every key it borrows through its own `./_External.mjs`.

**ETB-3.** An import of another _module_'s `_Symbol.mjs` MUST be written in the
importing _module_'s own `_External.mjs` and nowhere else.

**ETB-4.** A _module_'s `_External.mjs` MUST re-export every table it imports.

**ETB-5.** Another _module_'s `_External.mjs` MUST NOT be imported.

**ETB-6.** The _package_'s reference graph MUST be acyclic, and every upward
reference MUST be made in `_External.mjs`.

> **Rationale**
> The leaf property is what makes a cycle structurally impossible rather than
> merely avoided. A _module_ that reaches upward from its own _symbol table_
> violates the property even when no cycle exists today, because the next
> borrowed table closes one.

## 8. Abstract and Concrete

**ABS-1.** A _constructor_ that a _consumer_ is expected to derive from
MUST be declared _abstract_ through the shared _abstract_ layer. Documenting it
as _abstract_ in prose MUST NOT be used as a substitute.

**ABS-2.** Every _abstract member_ MUST declare a contract for what it
returns, including whether it MAY answer a promise.

**ABS-3.** The contract surface of a family is exactly its
_abstract members_. A _derivation_ takes them on as an obligation; every
other _member_ in the _module_ is maintenance surface.

**ABS-4.** A _constructor_ with an unimplemented _abstract member_ MUST be
_abstract_ itself.

> **Rationale**
> Implementing part of a contract does not discharge the rest, so the
> _derivation_ that leaves an _abstract member_ unimplemented is not
> constructible. The converse does not hold: a contract not yet designed leaves
> no unimplemented member behind, and the subject may still be _abstract_.

**ABS-5.** A _consumer_ MUST NOT instantiate the _abstract_ _subject_ directly.
ECMAScript stops no such call, so the prohibition is a convention.
Constructing the _abstract_ _subject_ SHOULD fail.

> **Rationale**
> Making the call fail is a technical measure, not a free one: ECMAScript
> supplies none, so without a shared helper it costs a guard in every _abstract_
> _constructor_, and the guard ships. A _package_ that reads the cost as too
> high settles for the convention.

**ABS-6.** A _derivation_ MAY be produced without `class` and `extends` syntax,
by any means that yields the same derivation relation.
Doing so is permitted and does not conflict with the purpose of this
specification, but it is not RECOMMENDED: it is laborious to write and to
read. The RECOMMENDED form is `class ... extends`.

## 9. Construction

**CTR-1.** A _constructor_ that needs the construction target MUST capture it
with `new.target`, and MUST NOT read `this.constructor`. Which _member_ holds
the captured target is not fixed by this document.

> **Rationale**
> The target is a fact about one moment, the call that built the object, and
> `new.target` is the only expression that reports it at that moment.
> `this.constructor` is a property lookup that any assignment can shadow, so it
> reports what the prototype chain says rather than what ran. An _internal_
> _member_ is a safe home for the reference because it is not carried by the
> _package_ export.

**CTR-2.** A _constructor_ MUST accept the smallest argument set that is
stable for the whole family. A secondary dependency MUST NOT be added as a
further _constructor_ argument.

**CTR-3.** A dependency that cannot be passed to the _constructor_ MUST be
attached through an explicit _protected_ _member_ before the object is reachable
from the _public_ surface.

**CTR-4.** A guard MUST be justified by a reachable state, not by caution.
Where an operation is single-shot because its only caller is single-shot,
the guard MUST NOT be added; where the set of callers is not closed, the
guard MUST be.

> **Rationale**
> An unreachable guard is a dead branch, and a dead branch survives review as
> if it were protection. The criterion for adding one is therefore the caller
> set, not the risk.

## 10. Public Surface

**PUB-1.** The _package export_ MUST be the only export space of a
_package_, and everything a _consumer_ may reach MUST be organized there. A raw
_symbol table_ or _borrowing table_ MUST NOT be carried by it.

**PUB-2.** A _member_ that the _concrete_ must read or write beyond the
_abstract_ contract MUST be exposed through the _package export_ as a
grouped symbol namespace containing _abstract_ keys only.

**PUB-3.** The _alias tables_ MUST NOT be reachable from the
_package export_.

**PUB-4.** When the _concrete_ needs a capability that only an
_internal_ _member_ can provide, promoting the capability to a _public_ _member_
SHOULD be preferred over exposing the _internal_ _member_.

> **Rationale**
> Symbols are the maintenance surface; _public_ _members_ are the
> promise surface.
> Exposing an _internal_ _member_ converts an _internal_ change into a breaking
> change.

**PUB-5.** The _abstract_ level MUST be the only level that crosses the package
boundary, so that a _derivation_ outside the _package_ can reach the
_abstract members_ it must implement.

## 11. Subject Export

**SUB-1.** A _subject export_ MUST carry the _subject constructor_, as
`Abstract` when the _subject_ is _abstract_ and as `Concrete` when it is
_concrete_.

**SUB-2.** A _subject export_ MUST NOT carry a _symbol table_ or a
_borrowing table_.

**SUB-3.** A _subject_ that the _module_ uses MUST be carried by the
_subject export_ as a namespace under that _subject_'s own name.

**SUB-4.** A _subject export_ MAY carry anything else, provided that no other
export takes the name `Abstract` or `Concrete`.

## 12. Naming

**NAM-1.** A name MUST be checked against the mainstream practice of its
domain before it is adopted. A name that is defensible only inside this
_package_ MUST NOT be adopted.

**NAM-2.** A term that denotes one thing MUST NOT be used for another. Where
two concepts are close, the _package_ MUST fix one word per concept and
record both.

**NAM-3.** A role name and the name of a _concrete_ MUST be distinguished. A
role name carries the contract vocabulary and MUST NOT be renamed when the
_concrete_, the _package_, or the product changes.

**NAM-4.** A hook that the framework calls MUST be named as a verb phrase; a
hook that the framework reads MUST be named as a noun phrase.

**NAM-5.** A _package_ that introduces domain vocabulary MUST record it as a
glossary, one entry per term, stating the distinction that makes the term
necessary.

## 13. Conformance

**CON-1.** A _package_ conforms only if all of the following hold: its
reference graph is acyclic; every _subject export_ carries its
_subject constructor_; no _symbol table_ imports anything else from the
_package_; every _abstract member_ declares a contract; every borrowed table
is re-exported from `_External.mjs`.

## 14. Open Questions

- The single-file exception has a three-condition criterion but no threshold
  for "extreme simplification". Whether the criterion is sufficient, or
  needs a stated maximum size, is not settled.
- Which coding conventions can justify an alias is not enumerated, and where
  the boundary lies is left to review.
