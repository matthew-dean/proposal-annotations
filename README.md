# ECMAScript proposal: Notes (Annotations)

Notes are extremely versatile, making them suitable for anything, from simple documentation to type checking by a type checker before runtime (by a system like TypeScript or Flow) to type checking at runtime.

Because of their versatility, this is considered a suitable replacement for the [controversial Type Annotations proposal by Microsoft and others](https://github.com/tc39/proposal-type-annotations).

In this proposal, we call this feature notes for simplicity, but it is conceptually identical to annotations.


## Status

**Stage:** 0

**Authors:**

- Matthew Dean
- ...and other contributors welcome

## Prior work

There is an existing proposal to add annotations to decorators here: https://github.com/tc39/proposal-decorators/blob/master/EXTENSIONS.md

However, that suffers from some pitfalls making it unsuitable for annotating type information.

## Assumptions

This proposal starts with some assumptions:

1. We should be able to annotate almost any function, parameter, object property, or variable declaration (identifier).
2. This annotation (note) should be very minimal.
3. The note runtime output should have a predictable structure (a string), and should be an empty string if there is no annotation.
4. Notes should be referenceable at runtime with minimal syntax.


## Syntax Rules

Rules for notes.
1. Notes start with `#`.
2. Notes can be followed by an identifier, a `()` or `<>` block, or the characters `?` or `!`.
3. When followed by a block, the note terminates at the end of the (outer) block. When followed by an identifier, the note terminates at the end of the identifier, with the exception that the identifier or block is followed by a `?` or `!`, which is included in the note, and terminates the note.
4. Like CSS custom properties, note have few restrictions, other than:
  a. inner block starts `{` `[` `<` `(` must have valid matching block ends (`}` `]` `>` `)` respectively)
  b. string starts `'` and `"` are parsed as a string literal with valid string contents and string ends.

Example:

```js
// Note that this has no particular meaning to the runtime,
// but is useful for, say, a type system.
#(return number)
function add(#number one, #number two) {
  return one + two
}
```

## Runtime effect

### `noteof`

To get notes at runtime, an author would write `noteof foo`. Why?

1. A form like `foo[Symbol.notes]` cannot work if `foo` is undefined or not object-like (but has an attached note).
2. Similarly, a global namespace object like `Note` with `Note.get(foo)` functioning as a kind of WeakMap also is illogical, since variables are passed by value, and what we want to annotate is the variable, so we really need a new syntax keyword.

For a function, we should add a `paramaters` property to the `Function` prototype (TBD), to support logical retrieval of notes.
```js
// "(return number)"
noteof add
// "number"
noteof add.parameters[0]
```

At a static code analysis stage, those annotations could be used to infer and lint types.

### Un-attached annotations
Annotations that are followed with multiple line-breaks are not attached to the next class / function / variable but are instead
attached to the module.

```js
#(This is a file annotation)

function someFunc() {}

noteof someFunc // ''
noteof import.meta // '(This is a file annotation)'
```

## Let annotations
Variables can also have attached annotations.

```js
let #num x = 1;

noteof x // 'num'
```

Note that the above type annotation would have no effect at runtime, but could allow a type-checking system to enforce the type through static analysis (or runtime helpers).

## Const decorators
One can also attach a note to a const.

```js
const #num x = 1;
```

## How could we use this proposal?

The simplest way to use this metadata annotations is to provide "structured documentation", but probably one of the most powerful ways to use this proposal is for type-checking systems.

[Let's check it out!](./notes_as_types.md)
