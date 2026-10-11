# Normalizer Construction Pattern Specification

> Engineering conventions for writing a normalizer — the one function that
> turns a many-shaped input into the single internal shape the rest of a
> system runs on.

## 1. Status of This Document

This document is the normalizer construction pattern specification. Adopting
documents cite it by that name.

It is a draft. It generalizes a working implementation that arrived at this
form under pressure and then had the form read back off it.

Origin: an options normalizer in a service package — the function that maps a
consumer-supplied options object into the record an implementation layer reads
(`produck/kitty`, `packages/workflow/src/Exchange/Implement/Normalize.mjs`).

**Provisional** marks a rule that has been adopted in practice but whose
criterion is not yet settled. **Optional** marks an element the pattern lets
an adopting implementation leave out. **Open** marks a question this draft
refuses to answer, listed in Section 11 for whoever continues the work.

A word defined in Section 3 is written in _italics_ where it is meant as that
term, and left plain where it is meant in its everyday sense. The sibling
specification counts marked occurrences beside each definition; this draft
leaves those counts out, because the rule set is still moving.

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

A normalizer is the seam where a system stops accepting shapes and starts
having one. Everything downstream reads the internal shape and nothing else,
so the normalizer is also the only place where a shape can be admitted, and
the only place where the cost of admitting it is paid.

This document covers:

- The form of the normalizer's code: its blocks, their nesting, and their
  boundaries.
- The isolation of one _unifying step_ from another, and what that isolation
  is worth.
- How a _polymorphic node_ and a _collection node_ are written.
- What a normalizer MUST NOT be driven by, and why.

This document does not cover:

- The internal shape itself. What the system needs is the system's design, not
  this document's.
- Where the normalizer is called, or what the caller does with a throw.
- Leaf-signature validation: this document is about the _form_ of the mapping,
  not about how strictly any one leaf is checked.
- Collecting a shape out of a normalizer. The pattern neither requires nor
  forbids it; Section 10 is a memo, not a requirement.

## 3. Definitions

**Normalizer** — the one function that maps an _input_ into the _internal
shape_: it reads the input, builds one _node_ per step, and returns the whole
shape at once. It is a unifier, not a validator: validation is what reading
the input happens to do along the way.

**Input** — the value handed to the normalizer at one position: the whole
object at the root, a group inside it, or an element of a collection.

**Internal shape** — the value the normalizer returns and the rest of the
system consumes. It has exactly one form per node; the input may have several.

**Node** — one position in the internal shape: a group or a leaf.

**Block** — the brace-delimited body that owns one node: the `if` that admits
the node's forms, the bindings for those forms, and the write into the target.
A block is the unit of this document.

**Block scope** — the lexical scope the block creates. All short local names
of a block live and die there.

**Unifying step** — one pass over one node: admit a form, read it, write the
target. A block is one unifying step.

**Form** — one accepted shape of an input for a node. A node with more than
one form is polymorphic.

**Polymorphic node** — a node whose input may arrive in more than one form
(an object or a list of pairs; a string or a list of strings; a value or a
getter).

**Collection node** — a node whose input is a list, walked element by element.

**Error path** — the string that names where the input was rejected, at the
precision of the leaf. It is the normalizer's only user-visible output on
failure.

**Literal path** — an error path written where the rejection happens, as a
string literal or a template whose only interpolations are indices.

## 4. Form

**NRM-1.** A normalizer MUST be one function with one exit. Every path either
writes its node and continues, or throws.

> **Rationale**
> A half-built shape that is returned is worse than a throw, because nothing
> downstream can tell it apart from a finished one. One exit is what makes
> "the shape is complete or the call failed" a property instead of a promise.

**NRM-2.** The mapping MUST be written as nested blocks, one block per node.
A block MUST NOT be shared between two nodes.

**NRM-3.** A block MUST own exactly one target, and MUST write only that
target. A block MUST NOT write a node that a sibling owns.

**NRM-4.** A block MUST bind both sides of its step: the input form under a
raw name, and the target under the node's own name.

> **Rationale**
> The block is the write surface and the read surface at once. Binding both
> with short names is what lets a reader see, inside one brace pair, what came
> in and what goes out — and it is what lets the raw name be reused at every
> depth instead of being made unique per node.

**NRM-5.** (**provisional**) Within a block, the raw input SHOULD be bound
under the target's own name with a leading underscore (`header` / `_header`),
and the nested group names SHOULD be reused as-is at each depth.

**NRM-6.** A rejection MUST use a _literal path_, spelling the position of
the offending input from the normalizer's own argument outward.

**NRM-7.** A rejection MUST describe the form that was expected, and MUST NOT
attempt to repair the input.

> **Rationale**
> Repair is a second unifier living under the first: the caller cannot tell
> whether the shape came from the caller's intent or from the normalizer's
> guess. Refusing is the only answer that keeps the mapping honest.

## 5. Isolation

**NRM-8.** A block MUST NOT rely on a binding it did not make. Names of a
sibling block MUST NOT be reachable from it.

> **Rationale**
> This is the property that pays for the expanded form. A block that forgets
> to bind its own input does not silently reuse its neighbour's: it fails at
> the point of use, loudly, in the block that is incomplete. The isolation is
> not a style preference; it is a check the language performs for free.

**NRM-9.** Bindings MUST NOT be made globally unique to avoid collisions.

> **Rationale**
> Uniqueness is a tax paid to a scope problem that braces already solve. A
> name made unique for the whole function also makes it unreadable in the
> block that uses it, and it survives the deletion of the block it was named
> for.

**NRM-10.** A block MUST end its own step: it writes its target, or it throws.
A block MUST NOT leave its node in a state for a later block to finish.

**NRM-11.** An exception MUST be raised from inside the block that owns the
node, so the raising site and the owning scope are the same place.

> **Rationale**
> The error path and the code path stay the same fact. There is no separate
> table of paths to keep in step with the code, so there is nothing to drift.

## 6. Polymorphic Input

**NRM-12.** Every form of a polymorphic node MUST be handled inside the block
that owns that node, as sibling branches of the same block.

**NRM-13.** A form MUST NOT be delegated to a shared abstraction that the
other nodes also use.

> **Rationale**
> The normalizer is expected to accumulate exceptions — one node gains a
> polymorphic form, another gains a default, a third merges two inputs. A
> shared abstraction must be given a vocabulary before it can grow; an
> exception that does not fit the vocabulary forces either a wider abstraction
> (one node's local need becomes a global change, and every existing node is
> re-read against new semantics) or a special case outside it (the abstraction
> and the behaviour part ways). Both are worse than the repetition saved.

**NRM-14.** The unify order inside a block MUST be explicit: when two forms
can produce the same node, the block MUST state which one is read first and
what the later one does to what the earlier one wrote.

> **Rationale**
> "Merge" is a sequence, and a sequence is visible in statements. A data
> description of the same thing would have to invent merge semantics, which is
> the beginning of a language inside a language.

## 7. Collection Input

**NRM-15.** A collection node MUST be walked with a loop whose body is a
block, so that each element gets its own scope and its own bindings.

**NRM-16.** A rejection inside a collection MUST name the element's index in
its _literal path_.

> **Rationale**
> Per-iteration scope is what keeps one element's temporary bindings from
> reaching the next element, and the index is what makes the failure of
> element 40 distinguishable from the failure of element 1 without a debugger.

## 8. Prohibited: a data-driven shape

**NRM-17.** A normalizer MUST NOT be driven by a description of the shape
(declarative schema, shape table, node tree, or any structure whose data
states which node takes what). The mapping MUST be written in statements.

> **Rationale**
> A description is a second statement of the shape, and it is a weaker one:
> it can hold what its vocabulary names, and nothing else. The normalizer's
> whole difficulty is the case its vocabulary does not name — that is what a
> _polymorphic node_ is. Worse, the drift between a description and the code
> it drives is silent: the description still reads as true while the code has
> moved on.
>
> The isolation of Section 5 is also lost. A description runs in one scope, so
> recovering per-node scope means wrapping each node in a closure or
> generating code — re-implementing, outside the language, what braces give
> for free.

**NRM-18.** A normalizer MUST NOT import the tables that declare the shape it
maps (symbol tables, key tables, member tables). It names what it reads.

> **Rationale**
> Driving the mapping from the tables of the layer being implemented makes the
> normalizer depend on the layer's internals, and makes the tables the single
> source of truth for two different questions — which keys exist, and in what
> form an input may carry them. The second question is not the tables'.

## 9. Worked Example

The shape of one block, a plain group:

```js
if (isPlainObject(_header)) {
  const header = (_options.request.header = {});

  const { get: _get, keys: _keys } = _header;

  if (typeof _get === 'function') {
    header.get = _get;
  } else {
    ThrowTypeError('args[0].request.header.get', 'function');
  }
} else {
  ThrowTypeError('args[0].request.header', 'plain object');
}
```

A second form added later, inside the same block, with the same names reused:

```js
} else if (Array.isArray(_header)) {
  const header = (_options.request.header = {});

  for (const [index, entry] of _header.entries()) {
    if (!Array.isArray(entry) || entry.length !== 2) {
      ThrowTypeError(`args[0].request.header[${index}]`, 'pair');
    }

    // The same leaf reading, in the same block, per element.
  }
} else {
  ThrowTypeError('args[0].request.header', 'plain object');
}
```

Both edits are local: the node's block is where the node's exception lives,
and no sibling block, and no description of the shape, is touched.

## 10. Memo: the optional structure tracker

This section is a memo, not a requirement. It records where the introspection
cost of Section 8 can be paid back, and on what terms.

A _tracker_ is a mechanism that watches a normalizer run and writes down the
structure it saw. It is **Optional** in the sense of Section 1: a repository
may ship no tracker at all. It is **runtime-erasable**: nothing about it needs
to survive into the artifact that runs.

**TRK-1.** A tracker MUST NOT be required by a normalizer. A normalizer MUST
read the same, and be correct on its own, whether a tracker is attached or
not.

**TRK-2.** A tracker MUST be erasable at runtime: the shipped artifact MUST be
able to exclude the tracker, its instrumentation, and the cases that drive it,
without editing the normalizer.

**TRK-3.** A tracker SHOULD live outside the normalizer — a harness over the
call (a recording proxy over the input and over the returned shape) rather
than a notification line added to each block.

> **Rationale**
> A notification line is paid for inside the one place this pattern exists to
> keep clean. The harness pays nothing there, and the receiver identities in
> its log already recover the nesting: reads on the root are the root's keys,
> and reads on the object a key returned are that node's keys.

**TRK-4.** A tracker's completeness MUST be certified by branch coverage of
the normalizer, every branch taken. A form or a rejection that no case
exercises is not collected, and coverage is what says so out loud.

> **Rationale**
> A happy path alone sees neither the rejection branches nor the other forms
> of a polymorphic node. Each one needs a case that reaches it, and branch
> coverage is the existing tool that names the ones that are missing. The
> failure mode is the point: a branch added without a case lowers coverage and
> is heard, where a description that fell behind its code is silent.

**TRK-5.** A tracker's output MUST be derived: code to report, never report
to code. A normalizer MUST NOT read back what a tracker wrote.

> **Rationale**
> The pattern gave up a description so that there would not be two statements
> of the shape. A report that is read back is that description returning by
> the side door.

What a tracker sees: the keys read at each node, the order of the reads, the
type of each value, the _error path_ of each rejection, and the internal node
each form was written into. What it cannot see on a happy path alone: the
rejection branches and the other forms — those need cases. Static scanning
complements it in the other direction, seeing branches that no run reached.

## 11. Open Questions

1. **Counts**: adopt the sibling specification's term-italic counting device,
   or leave it to the sibling alone?
2. **Naming**: is NRM-5 a rule or a habit? If a rule, is the leading
   underscore the only allowed raw-name convention?
3. **File name**: does the pattern fix the file (`Normalize.mjs`), or name
   files by node (`Request.mjs`)?
4. **Splitting**: may one normalizer's blocks live in several files, and if
   so, what carries the target between them?
5. **Output freezing**: must the returned shape be frozen? What about the
   groups inside it?
6. **Unrecognized keys**: dropped, merged, or rejected? The normalizer
   decides, but is a default required, and is silence allowed?
7. **Leaf strictness**: how deep may a leaf check go (`typeof` only, or owns
   shape)? Whose contract is it — this document's or the implementing
   layer's?
8. **Async**: may a normalizer be async, and may a _form_ be a promise?
9. **Recursion**: may a form branch call the normalizer again for a nested
   node, or MUST nesting stay lexical?
10. **Relation to the sibling**: does this document need an ID prefix of its
    own, and is it cited by the partial-constructor specification or beside
    it?

11. **Tracker form**: harness over the call, or instrumentation inside the
    blocks? This draft recommends the harness (TRK-3) and does not settle it.

## 12. Precedence

A repository may add stricter rules. Where a repository's rules conflict with
this document, the repository's rules win, and the exception is recorded with
the code that takes it.
