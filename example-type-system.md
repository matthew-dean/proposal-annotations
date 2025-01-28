# Hash comments example - a type system

Many examples and descriptions in this example come from the [type annotations proposal](https://github.com/tc39/proposal-type-annotations), so to properly credit those authors, they are also listed here:

> **Authors:**

> - Gil Tayar
> - Daniel Rosenwasser (Microsoft)
> - Romulo Cintra (Igalia)
> - Rob Palmer (Bloomberg)
> - ...and a number of contributors, see [history](https://github.com/tc39/proposal-type-annotations/commits/master).

### Type annotations

Below is a simple hash comment which could be interpreted by a type system for type linting checks.

```ts
let x #|string;

x = "hello";

x = 100;
```

In the example above, `x` is commented with `#|string`. A type linting system could utilize that type, and might choose to error on the statement `x = 100`; however, a JavaScript engine that follows this proposal would execute every line here without error, because to the JS engine, `#|string` is only a comment.

_Note that whether or not `#|string` is treated as meaningful to the type linter is up to the features of the type linter. The type linter could opt to throw a parsing error for this hash comment, and could instead require something like `#:string`. Regardless, either one is a valid hash comment, and so would be ignorable by any JavaScript parser._

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
function equals(x#:number, y#:number) #:boolean {
    return x === y;
}
```

Here we have specified `number` for each parameter type, and `boolean` for the return type of `equals`, if that's how the type system chose to interpret those. Note, a meta-language could also support something like this:

```ts
function equals#<T extends number | boolean>(x#:T, y#:(number | boolean)) #:boolean {
    return x === y;
}
```

### Type Declarations

Much of the time, developers need to create new names for types so that they can be easily referenced without repetition and so that they can be declared recursively.

One way to declare a type - specifically an object type - is with an interface. This is trivial to capture with a hash comment.

```ts
#(interface Person {
  name: string
  age: number
})
```

This could be applied to an object like:

```ts
const person#:Person = {}
```

as well as (depending on the meta-language)

```ts
const #|Person person = {}
```

A type-checking system could exhaustively check if the defined object including the properties `name` and `age`.

Here we can see something like a type alias:

```ts
#<type CoolBool = boolean>
```

### Classes with Hash Comments

Here's an example of a full class annotated with hash comments.
```ts
class Person {
    name #:string;
    constructor(name #:string) {
        this.name = name;
    }

    getGreeting() #:string {
        return `Hello, my name is ${this.name}`;
    }
}
```

For most type-checkers, annotated class members would contribute to the type produced by constructing a given class.
In the above example, a type-checker could assume a new type named `Person`, with a property `name` of type `string` and a method `getGreeting` that returns a `string`;
but like any other syntax in this proposal, these annotations do not weigh into the runtime behavior of the program.


### Parameter Optionality

Because hash comments allows a number of "outer-level" special characters, including `?`, a type linter could include `?` to denote optionality of a parameter, per this example

```ts
function split(str #:string, separator #:string?) {
    // ...
}
```

### Importing and Exporting Types

As projects get bigger, code is split into modules, and sometimes a developer will need to refer to a type declared in another file.

Hash comments could make for an explicit syntax for a type system.

Just like JavaScript, type declarations could be exported by prefixing them with the `export` keyword.

```ts
#<type SpecialBool = boolean>

export { #(SpecialBool) }
```

From a JavaScript perspective, there's no syntax issue here. According to the JS engine, it's just an export statement with an empty block.

Note: there would also be nothing stopping a meta-language from allowing this instead:

```ts
#<export type SpecialBool = boolean>
```

#### Leading commas in import / export

One thing TC39 could consider is loosening import / export to allow leading commas, for better DX in a case like this:

```ts
#<type SpecialBool = boolean>

const MY_CONSTANT = 150

export { #|SpecialBool, MY_CONSTANT }
```
The above would technically be a syntax error, because the JavaScript engine would read this as `export { , MY_CONSTANT }` and currently, leading commas are not allowed. This restriction, however, could be loosened to allow hash comments in these lists.

That said, this would be a reasonable approach:
```ts
export { #(FooType, BarType,) MY_CONSTANT }
```

In this case, the comma is within the ignorable hash comment, so a parsing error would not be thrown.

#### Imports example

The following could import a variable export of `parseSourceFile`, and a type export of `SourceFile`.

```ts
import { parseSourceFile, #(SourceFile) } from "./parser.js";
```

_Note: the above would NOT a JavaScript syntax error because imports and exports allow TRAILING commas which, again, may be counter-intuitive to developers._
 
### Type Assertions

Here are some different hash comment syntax possibilities for a type system to assert a type.

```ts
let value = 1 #!number;
```
```ts
let value = 1 #<as number>;
```
```ts
let value = #<number> 1;
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
function foo#<T>(x #:T) {
    return x;
}

class Box#<T> {
    value #:T;
    constructor(value #:T) {
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
function sum#<this: SomeType>(x #:number, y #:number) {
    // ...
}
```

## Extra snippets

With the versatility of Hash comments, there are many other ways in which a type system could establish a sub-syntax for other features. Here are some examples:

### Function Overloads
```ts
#<function foo(x: number): number>
#<function foo(x: string): string>
function foo(x #:(string | number)) #:(string | number) {
    if (typeof x === number) {
          return x + 1
    }
    else {
        return x + "!"
    }
}
```

### Class and Field Modifiers


```ts
class Point {
    #|public #|readonly x #:number
}
```

A type system could also do something like:

```ts
class Point {
  #<number> x #|public #|readonly 
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
function foo({ name #:string, age #:number })
```

Destructuring and aliasing to another variable name would also be fine (demonstrating a different syntax approach by the meta-language):
```ts
function foo({ name #(string) : localName, age #(number): localAge, ...rest #(any[]) })
```

This is obviously a non-goal, but potentially, this proposal could not only make JavaScript development easier, but could offer improvements for current meta-languages.
