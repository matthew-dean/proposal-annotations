# Notes as Types

First, we assume you've taken the time to read the proposals that this one extends, such as:
- https://github.com/tc39/proposal-type-annotations
- https://github.com/tc39/proposal-decorators/blob/master/EXTENSIONS.md

The controversial [Type Annotations Proposal](https://github.com/tc39/proposal-type-annotations) adds a lot of syntax that the authors propose should all be dismissed "as comments" by a JavaScript engine, adding over a dozen new "comment types", that not only is inconsistent with itself (in terms of a comment syntax), but also is defined in a way that adds no usefulness at runtime.

Instead, if you've looked at the [syntax of this proposal](./README.md), you may be starting to get a sense of how we could define type annotations that are meaningful for a type-checking system, like TypeScript or Flow, but do so without polluting the JavaScript language. 

We'll use the examples / language of the original type annotations proposal using our metadata decorator syntax, to see how we can achieve the same goals without syntax pollution. In other words, consider much of the following to be quoted / referenced from the existing type annotations proposal, but edited to demonstrate decorator syntax.

## Proposal

### Type Notes

Type notes allow a developer to explicitly state what type a variable or expression is intended to be.

```ts
let #string x;

x = "hello";

x = 100;
```

In the example above, `x` is annotated with the type `string`. Tools such as TypeScript can utilize that type, and might choose to error on the statement `x = 100`; however, a JavaScript engine that follows this proposal would execute every line here without error. This is because metadata annotations do not change the semantics of a program, and are equivalent to comments.

Notes can also be placed on parameters to specify the types that they accept, and following the end of a parameter list to specify the return type of a function (as a sanity check by the type-checker).

```ts
#(return boolean)
function equals(#number x, #number y) {
    return x === y;
}
```

Here we have specified `number` for each parameter type, and `boolean` for the return type of `equals`.

### Type Declarations

Much of the time, developers need to create new names for types so that they can be easily referenced without repetition and so that they can be declared recursively.

One way to declare a type - specifically an object type - is with an interface.

```ts
#(
  interface Person {
    name: string
    age: number
  }
)
```

This could be applied to an object like:
```ts
const #Person person = {}
```
A type-checking system could exhaustively check if the defined object including the properties `name` and `age`.

Here we can see something like a type alias:

```ts
let #boolean CoolBool
```

### Classes with Type annotations

Classes in Stage 3 of the TC39 already allow decorators, so this proposal needs to do very little.

```ts
class Person {
    #string name;
    constructor(@'string' name) {
        this.name = name;
    }

    #string
    getGreeting() {
        return `Hello, my name is ${this.name}`;
    }
}
```

For most type-checkers, annotated class members would contribute to the type produced by constructing a given class.
In the above example, a type-checker could assume a new type named `Person`, with a property `name` of type `string` and a method `getGreeting` that returns a `string`;
but like any other syntax in this proposal, these annotations do not weigh into the runtime behavior of the program.

### Kinds of Types

This section is ommitted because it's really only important to the type-checking system.

### Parameter Optionality

Again, we don't need to define anything in the syntax space, but we'll offer an example for a syntax that something like TypeScript could use. Here, we've added a question mark to designate to TypeScript that the parameter is optional.

```ts
function split(#string str, #string? separator) {
    // ...
}
```

### Importing and Exporting Types

As projects get bigger, code is split into modules, and sometimes a developer will need to refer to a type declared in another file.

Types declarations can be exported by prefixing them with the `export` keyword.

```ts
export let #boolean SpecialBool;

#(
  interface Person {
    name: string
    age: number
  }
)

export let #Person person
```

### [Removed]

The Microsoft proposal had a lot of syntax for `import type` and `export type`. This is meta-information that could be implemented at the compiler level, based on notes, and shouldn't be added at the JS level.
 
### Type Assertions

Instead of decorating the variable, a type-checking system could have you decorate the value in order to assert its value.

```ts
const point = #(as { x: 'number', y: 'number' }) JSON.parse(serializedPoint);
```

At runtime, this is syntactic sugar for:
```ts
const #(as { x: 'number', y: 'number' }) point = JSON.parse(serializedPoint);
```

However, a system like TypeScript could use different inference behavior depending on the position of the note (before the variable identifier or before the value).

#### Non-Nullable Assertions

One of the most common type-assertions in TypeScript is the non-null assertion operator.
It convinces the type-checker that a value is neither `null` nor `undefined`.
For example, one can write `x!.foo` to specify that `x` cannot be `null` nor `undefined`.

Here's how we might do this with a note:

```ts
// assert that we have a valid 'HTMLElement', not 'HTMLElement | null'
document.getElementById("entry")#!.innerText = "...";
```

### Generics

In modern type systems, it's not enough to just talk about having an `Array` - often, we're interested in what's *in* the `Array` (e.g. whether we have an `Array` of `strings`).
*Generics* give us a way to talk about things like containers over types, and the way we talk about an `Array` of `strings` might be by writing `#(Array<string>)`.

#### Generic Declarations

```js
// Something like `type Foo<T> = T[]`
#(type Foo<T> = T[])

// Something like `interface Bar<T> { x: T }`
#(interface Bar<T> { x: T })
```

Functions and classes can also have type parameters, though variables and parameters cannot.

```ts
#<T>
function foo(#T x) {
    return x;
}

#<T>
class Box {
    #T value;
    constructor(#T value) {
        this.value = value;
    }
}
```
Combining generics and other function meta-information:
```ts
#(<T>foo(#T x): T)
function foo(x) {
    return x;
}
```

### Generic Invocations

One can explicitly specify the type arguments of a generic function invocation or generic class instantiation [in TypeScript](https://www.typescriptlang.org/docs/handbook/2/functions.html#specifying-type-arguments).

```ts
// TypeScript
add<number>(4, 5)
new Point<bigint>(4n, 5n)
```

The above but with notes syntax:

```ts
// Types Annotations - example syntax solution
add#<number>(4, 5)
new Point#<bigint>(4n, 5n)
```

As always, these type arguments would have no effect during the JavaScript runtime.

### `this` Parameters

An example idea for typing the `this` value.

```ts
// or @{ this: SomeType } -- up to TypeScript or like systems
#(this SomeType)
function sum(#number x, #number y) {
    // ...
}
```

## Omissions

This document has the same omissions as [Microsoft et. al's types proposal](https://github.com/tc39/proposal-type-annotations), mostly because there aren't examples.

But, that said! Because there is very little added to the syntax space, and we're only extending the existing annotations proposal, then we can actually do _more_ than the old proposal, but exactly what that might be is out of the scope of this proposal.