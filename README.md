# ECMAScript proposal: Metadata Annotations

This proposal extends [the annotations extensions proposal for decorators](https://github.com/tc39/proposal-decorators/blob/master/EXTENSIONS.md). Annotations are extremely versatile, making them suitable for anything, from simple documentation to type checking by a type checker before runtime (by a system like TypeScript or Flow) to type checking at runtime.

Because of their versatility, this is considered a suitable replacement for the [controversial Type Annotations proposal by Microsoft and others](https://github.com/tc39/proposal-type-annotations).

## Status

**Stage:** 1

**Authors:**

- Matthew Dean
- ...and other contributors welcome

## Existing annotation syntax

In the TC39 decorators proposal, there is [a draft document proposing annotation syntax that this proposal extends from](https://github.com/tc39/proposal-decorators/blob/master/EXTENSIONS.md). Note that the scope of this document is limited to _metadata annotations_, and not decorators in general. For a description of how decorators that execute code at runtime might work for some of these, please see the [TC39 extensions proposal](https://github.com/tc39/proposal-decorators/blob/master/EXTENSIONS.md).

The TC39 proposed syntax is as follows:
```js
@{x: "y"}
function f() { }
```
It starts with `@` followed by an object. And is proposed to be referenced with something like
```js
f[Symbol.annotations][0].x  // "y"
```

## This proposal's syntax

This proposal starts with similar assumptions:

1. We should be able to decorate almost any function, parameter, or variable declaration with any annotation.
2. This decoration should be very minimal.
3. The annotation decorations should have a predictable structure (an array).
4. Metadata annotation decorations should be referenceable at runtime with minimal syntax.

Following the above guidelines, this proposal adds two forms of annotations by allowing strings and arrays as annotations, and not just objects:
```js
// existing syntax
@{ x: 'y' } 
// new syntax
@'x' // or double-quoted @"x" or as a template string @`x`
@['x']

// question: should there be a shorthand for simple string identifiers?
// like (all equal to @'x'):
//   @#x
//   @@x
```
Also, this proposal tries to keep the symbol consistent the symbol to retrieve metadata annotations (`Symbol.metadata`).

This would result in the following change:
```js
@'x'
function f() { }

f[Symbol.metadata][0] // "x"
```
Note that annotations are always a collection (an Array), because you can add multiple annotations.For example, a type-checking system might want to do something like:
```js
function f(@'number' @'string' p1) { }

f[Symbol.metadata][0] // 'number'
f[Symbol.metadata][1] // 'string'
```

## Parameter annotations

Like the TC39 extensions, as seen in the example above, parameters can be decorated (and have metadata annotations), like this:
```js
function f(@'x' arg) { return arg * 2 }
```

You could access the above like:
```js
// access the first parameter, and the first annotation
f[Symbol.metadata].parameters[0][0]  // 'x'
```

## Let decorators
Variables can also have attached metadata.

```js
let @'num' x = 1;

x[Symbol.metadata][0] // 'num'
```

Note that the above type annotation would have no effect at runtime, but could allow a type-checking system to enforce the type through static analysis (or runtime helpers).

## Const decorators
One can also attach metadata to a const.

```js
const @'num' x = 1;
```

## Value annotations

This may seem odd, but we could add metadata to something like a string expression or any other value. This would have (usually) no meaning at runtime, but could be used by a type-checking system as, say, file definitions:
```js
@'filetype' 'flow';
```
At runtime, this would be the equivalent of evaluating a plain string expression, like:
```js
'flow';
```
Technically, if this string were assigned to a variable, it would have the metadata `['filetype']`

Similarly, we could annotate a value like:
```js
let x = @'number' 5
```
The decorators proposal says that values can get wrapped in functions and returned, but _this_ proposal simply adds `[Symbol.metadata]: ['number']`.

This can be useful for type-checking "assertions" in TypeScript / Flow.

## How could we use this proposal?

The simplest way to use this metadata annotations is to provide "structured documentation", but probably one of the most powerful ways to use this proposal is for type-checking systems.

[Let's check it out!](./types_as_annotations.md)