# ECMAScript proposal: Hash Comments

This proposal expands JavaScript's [Hashbang comment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#hashbang_comments) syntax as a more generalized "Hash comment" syntax, with unique parsing rules, for the purpose of:
- enabling developers to add simple annotations to their JavaScript code,
- developing a full, expressive type-system, and/or
- adding any "contextual" information meaningful for a language that transpiles to JavaScript, or to a JavaScript AST (outside of the browser)

This proposal also extends and replaces the (somewhat controversial) [Type Annotations Proposal](https://github.com/tc39/proposal-type-annotations), which instead adds over a dozen new "comment types" that are not consistent and not easily generalizable outside of TypeScript.

Like the replaced proposal, this proposal would allow any JavaScript "meta-language" like TypeScript to run without transpilation. In doing so, it could remove the need for TypeScript itself (although this is not an intended goal).

## Status

**Stage:** 0

**Authors:**

- Matthew Dean
- and other contributors, see [history](https://github.com/matthew-dean/proposal-hash-comments/commits/main).

In addition, many examples and language in this proposal comes from the type annotations proposal, so to properly credit those authors, they are also listed here:

> **Authors:**

> - Gil Tayar
> - Daniel Rosenwasser (Microsoft)
> - Romulo Cintra (Igalia)
> - Rob Palmer (Bloomberg)
> - ...and a number of contributors, see [history](https://github.com/tc39/proposal-type-annotations/commits/master).

> **Champions:**

> - Daniel Rosenwasser (Microsoft)
> - Romulo Cintra (Igalia)
> - Rob Palmer (Bloomberg)

## Context & History

While the type annotations proposal has a number of flaws, much of the research and reasoning for the proposal is sound. We would recommend looking at:
- [Motivation: Unfork JavaScript](https://github.com/tc39/proposal-type-annotations/tree/main?tab=readme-ov-file#motivation-unfork-javascript)
- [Community Usage and Demand](https://github.com/tc39/proposal-type-annotations/tree/main?tab=readme-ov-file#community-usage-and-demand)
- [Trends in JavaScript Compilation](https://github.com/tc39/proposal-type-annotations/tree/main?tab=readme-ov-file#trends-in-javascript-compilation)

## Proposal

This proposal expands upon [Hashbang `#!` comments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#hashbang_comments), which are currently narrowly defined as:
- Limited to the very start of a file
- Beginning with `#!` and allowing any content until the end of the first line

This proposal preserves the above. If a hash comment `#`:
1. is at the start of the file with no preceding whitespace, and
2. is followed by `!`
  then: the entire line is preserved as a hash comment

Outside of this condition, a hash comment is parsed by these rules:
1. A hash comment starts with `#`
2. A hash comment may then consume:
  a. an optional first `:` character, and
  b. a JavaScript identifier, at which point it stops parsing and returns the comment, or
  c. the special characters `!`, `?`, `^`, `&`, `*`, at which point it stops parsing and returns the comment (with the exception of the hashbang rule in the case of `!`), or
  d. an angle-bracket block (`<` ... `>`) (See: [Angle-bracket blocks](#angle-bracket-blocks))
3. Hash comments in the form of `#identifer` have two additional rules to dis-ambiguate from private class fields (See: [Classes with Hash Comments](#classes-with-hash-comments)):
  a. Within a class definition, a hash comment with an identifier may not precede a private field name. However, a hash comment of any type may follow a valid field name.
  b. Hash comments with an identifier may not follow a period `.`

### Angle-bracket blocks

Much like CSS custom properties, angle brackets have looser parsing rules. Essentially, any _known_ or _unknown_ token is allowed, but block characters with "opening" block characters _must_ have matching "closing" characters.

In other words, within an angle-bracket block (opened with `<`):
1. a `(` character must have a matching `)` before the closing `>`
2. a `[` character must have a matching `]` before the closing `>`
3. a `{` character must have a matching `}` before the closing `>`
4. an inner `<` character must have a matching `>` before the closing `>`
5. the hash value must not have an un-matched `>`, `]`, `)` or `}`

### Using hash comments

For the purpose of demonstrating the versatility of this proposal, many of the examples of the type annotation proposal are re-purposed here. Below is a simple hash comment.

```ts
let x #string;

x = "hello";

x = 100;
```

In the example above, `x` is commented with `#string`. Tools such as TypeScript can utilize that type, and might choose to error on the statement `x = 100`; however, a JavaScript engine that follows this proposal would execute every line here without error, because to the JS engine, `#string` is only a comment.

Here's a function example, which could be written in a variety of ways that could be meaningful to a JavaScript meta-language:

#### Example 1:

```ts
#<(x: number, y: number): boolean>
function equals(x, y) {
    return x === y;
}
```

#### Example 2:

```ts
function equals(x #number, y #number) #boolean {
    return x === y;
}
```

Here we have specified `number` for each parameter type, and `boolean` for the return type of `equals`, if that's how the type system chose to interpret those. Note, a meta-language could also support something like this:

```ts
function equals#<T extends number | boolean>(x#:T, y#:<number | boolean>) #:boolean {
    return x === y;
}
```

### Type Declarations

Much of the time, developers need to create new names for types so that they can be easily referenced without repetition and so that they can be declared recursively.

One way to declare a type - specifically an object type - is with an interface. This is trivial to capture with a hash comment.

```ts
#<interface Person {
  name #string
  age #number
}>
```

This could be applied to an object like:

```ts
const person #Person = {}
```

as well as (depending on the language)

```ts
const #Person person = {}
```

A type-checking system could exhaustively check if the defined object including the properties `name` and `age`.

Here we can see something like a type alias:

```ts
#<type CoolBool = boolean>
```

### Classes with Hash Comments

As noted above, classes have slightly more-specific parsing rules for parsing hash comments.

For the following, the first identifier would be considered the private field name, and the second would be the hash comment.

```ts
class Person {
  #name #string; // a private field of `#name`, commented with `#string`
}
```

If a JavaScript meta-language wished to, say, define types preceding names, this is still allowed:

```ts
class Person {
  #<string> #name;
}
```

Neither of the above 2 examples would cause a JavaScript error.

_Note: a linting rule might want to expressly forbid hash comments with plain identifiers in the class body and force only hash-comment blocks for more clarity, but that's left up to developers. Thoughts welcome on this particular point._

Here's an example of a full class annotated with hash comments.
```ts
class Person {
    name #string;
    constructor(name #string) {
        this.name = name;
    }

    getGreeting() #string {
        return `Hello, my name is ${this.name}`;
    }
}
```

For most type-checkers, annotated class members would contribute to the type produced by constructing a given class.
In the above example, a type-checker could assume a new type named `Person`, with a property `name` of type `string` and a method `getGreeting` that returns a `string`;
but like any other syntax in this proposal, these annotations do not weigh into the runtime behavior of the program.


### Parameter Optionality

Again, we don't need to define anything in the syntax space, but we'll offer an example for a syntax that a meta-language with a type system could use. Here, we've added a question mark to designate to a type system that the parameter is optional.

```ts
function split(str #string, separator #string?) {
    // ...
}
```

### Importing and Exporting Types

As projects get bigger, code is split into modules, and sometimes a developer will need to refer to a type declared in another file.

To allow this gracefully, this proposal loosens one current syntax restriction: that `export` and `import` statements allow leading commas. (They both already allow trailing commas.)

Just like JavaScript, type declarations could be exported by prefixing them with the `export` keyword.

```ts
#<type SpecialBool = boolean>

export { #SpecialBool }
```

From a JavaScript perspective, there's no syntax issue here. According to the JS engine, it's just an export statement with an empty block.

Note: there would also be nothing stopping a meta-language from allowing this instead:

```ts
#<export type SpecialBool = boolean>
```

However, the reason this proposal loosens `export` / `import` is an example like the following:

```ts
#<type SpecialBool = boolean>

const MY_CONSTANT = 150

export { #SpecialBool, MY_CONSTANT }
```

It's reasonable for a meta-language to want to use hash comments in import / export lists as imported and exported types. Currently, though, the above would evaluate in a JavaScript to `export { , MY_CONSTANT }` which, according to the specification, is currently a syntax error (I believe?).

#### Type-Only Named Bindings

The `import` clause would also benefit from hash comments in a type system. The following could import a variable export of `parseSourceFile`, and a type export of `SourceFile`.

```ts
import { parseSourceFile, #SourceFile } from "./parser.js";
```
 
### Type Assertions

> Type systems do not have perfect information about the runtime type of an expression. In some cases, they will need to be informed of a more-appropriate type at a given position. Type assertions - sometimes called casts - are a means of asserting an expression's static type.

```ts
// TypeyScript
let value = 1 #<as number>;
```

```js
// Flowy
let value = 1 #<value: number>;
```

#### Non-Nullable Assertions

> It's a common use case for type-assertions, the type-checker filters null out of nullable types, checking if value is neither `null` nor `undefined`.

For example, one can write `x#!.foo` to specify that `x` cannot be `null` nor `undefined`.

```ts
// assert that we have a valid 'HTMLElement', not 'HTMLElement | null'
document.getElementById("entry")#!.innerText = "...";
```

### Generics

In modern type systems, it's not enough to just talk about having an `Array` - often, we're interested in what's *in* the `Array` (e.g. whether we have an `Array` of `strings`).
*Generics* give us a way to talk about things like containers over types, and the way we talk about an `Array` of `strings` might be by writing `#<Array<string>>`.

#### Generic Declarations

This proposal doesn't care as to the specifics of generic _declarations_, as that's up to the type system, but these are ported from the type annotations proposal.

```ts
#<
    type Foo<T> = T[]

    interface Bar<T> {
        x: T;
    }
>
```

To use type paramaters on functions and classes, a meta-language could do this:

```ts
function foo#<T>(x #T) {
    return x;
}

class Box#<T> {
    value #T;
    constructor(value #T) {
        this.value = value;
    }
}
```
Again, the specifics here of what the generics would look like are irrelevant and not important to the proposal. To a JavaScript engine, these are all just hash comments and can be ignored.

### Generic Invocations

One can explicitly specify the type arguments of a generic function invocation or generic class instantiation, [in TypeScript](https://www.typescriptlang.org/docs/handbook/2/functions.html#specifying-type-arguments) or in [Flow](https://flow.org/en/docs/types/generics/).

```ts
add#<number>(4, 5)
new Point#<bigint>(4n, 5n)
```


### `this` Parameters

A type system could define a type specific to the `this` binding for a function. As these are just comments, they have no impact on runtime function invocation.

```ts
function sum#<this: SomeType>(x #number, y #number) {
    // ...
}
```

## Extra snippets

With the versatility of Hash comments, there are many other ways in which a type system could establish a sub-syntax for other features. Here are some examples:

### Function Overloads
```ts
#<function foo(x #number) #number>
#<function foo(x #string) #string>
function foo(x #<string | number>) #<string | number> {
    if (typeof x === number) {
          return x + 1
    }
    else {
        return x + "!"
    }
}
```

### Class and Field Modifiers

Note that preceding comments require `#<>`.

```ts
class Point {
    #<public> #<readonly> x #number
}
```

A type system could also do something like:

```ts
class Point {
  x #public #readonly #number
}
```

### Types for objects

One thing that this proposal could potentially do is fix a flaw in type systems that use `:` as a type assignment, which sometimes conflicts with JavaScript which already uses `:` for value assignment and destructuring syntax, making it sometimes wholly incompatible.

For example, you cannot define property types in TypeScript for an object passed into a function. The following fails because of destructing syntax. It is ambiguous if we're defining an object type or trying to assign `name` and `age` to the local variables `string` and `number`, respectively. 

```ts
function foo({ name: string, age: number })
```
In TypeScript, you often have to alias types or do a more verbose form of:
```ts
function foo({ name, age }: { name: string, age: number })
```

With hash comments, we can make this less verbose:
```ts
function foo({ name #string, age #number })
```

Destructuring and aliasing to another variable name would also be fine:
```ts
function foo({ name #string: localName, age #number: localAge, ...rest #<any[]> })
```

This is obviously a non-goal, but potentially, this proposal could not only make JavaScript development easier, but TypeScript as well.

## Final thoughts

**Q: Could this be folded into the Type Annotations proposal?**

By and large, the Type Annotations has many of the right ideas (a simple, expressive way to annotate a file in a meaningful way that doesn't add too much noise), but ultimately proposes a convoluted solution (offering over a dozen _specific_ syntax pieces to be treated as comments, instead of one).

So, since these proposals have many of the same goals, with the additional goal to add as little new syntax to JavaScript as possible, there's no reason why champions of that proposal would necessarily not be champions of this proposal. This is the reason why this proposal is a fork of that one -- it's essentially a simple syntax proposal for the original set of goals.

**Q: If hash comments are generic, couldn't two different JS files potentially be interpreted by two different meta-languages?**

That's always a possibility, but this proposal doesn't seek to resolve it. TypeScript already allows developers to type-check / not type-check JS files by default in a project, and to opt-in / opt-in individual `.js` files, and presumably, other meta-languages might have similar mechanisms.

It would be recommended that differing systems that "interpret" hash comments in a meaningful way develop a standard (not unlike JSDoc) to specify the "hash comment language".

For example:
```ts
// my-typeyscript-file.js
#:typeyscript

// Now all the hash comments in this file should be interpreted by the TypeyScript build checker
```

```ts
// my-typeyscript-file.js
#:flowy

// Now all the hash comments in this file should be interpreted by the Flowy build checker
```