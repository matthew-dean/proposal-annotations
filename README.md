# ECMAScript proposal: Hash Comments

This proposal expands JavaScript's [Hashbang comment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#hashbang_comments) syntax as a more generalized "Hash comment" syntax, with unique parsing rules, for the purpose of:
- enabling developers to add simple annotations to their JavaScript code,
- embedding a full "meta-language" or superset of JavaScript _without_ triggering a parsing error in JavaScript
- allowing embedding known tokens (like other comments) within a comment without triggering a comment end

## Status

**Stage:** 0

**Authors:**

- Matthew Dean
- and other contributors, see [history](https://github.com/matthew-dean/proposal-hash-comments/commits/main).

## Why expand comments?

There have been various iterations of superset languages of JavaScript, such as [TypeScript](https://www.typescriptlang.org/), [Flow](https://flow.org/), or [Babel](https://babeljs.io/)-based languages. In addition, there are [metaprogramming-like libraries](https://github.com/GoogleFeud/ts-macros), which allow compile-time transformations or other contextual transformations.

Usually, these additions to JavaScript create the side-effect of making JavaScript un-parseable by JavaScript interpreters. What this proposal seeks to do is allow metaprogramming and superset languages while maintaining valid JavaScript.

## Proposal

This proposal expands upon [Hashbang `#!` comments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#hashbang_comments), which are currently narrowly defined as:
- Limited to the very start of a file
- Beginning with `#!` and allowing any content until the end of the first line

This proposal expands this definition to be of "Hash comments", which is parsed by these rules:
1. A hash comment starts with `#`
2. A hash comment must be followed by one or more of these special characters: `:`, `^`, `&`, `*`, `|` AND/OR the start of an angle-bracket (`<` ... `>`) or parenthetical (`(` ... `)`) block (see: [Blocks](#blocks))
3. A JavaScript identifer (e.g. `myIdentifier`) w/
   a. zero or one `?`
   a. zero or one `!`
4. Note that if the hash comment is at the very beginning of the file, starting with `#!`, then the whole line is still discarded as a comment.

So, the grammar of hash comments looks like:

hash_comment       = `#` ((pre_special ident) | (pre_special? block)) post_special?
pre_special        = `:` | `^` | `&` | `*` | `|`
post_special       = (`?` `!`?) | (`!` `?`?)
outer_block        = paren_block | angle_block
inner_block        = outer_block | bracket_block | curly_block
tokens_and_blocks  = js_tokens | inner_block | unknown_token
paren_block        = `(` tokens_and_blocks* `)`
angle_block        = `<` tokens_and_blocks* `>`
bracket_block      = `[` tokens_and_blocks* `]`
curly_block        = `{` tokens_and_blocks* `}`

In the above grammar, `js_tokens` are any known JavaScript tokens. For example, `/* this comment */` is a known token.

### Blocks

Much like (and inspired by) CSS custom properties, block contents in a hash comment have looser parsing rules. Essentially, any _known_ or _unknown_ token is allowed, but block characters with "opening" block characters _must_ have matching "closing" characters.

#### Why are tokens important? (i.e. why is a new comment syntax necessary?)

Let's say you want to embed a hash comment in JavaScript that represents a superset language construct. This construct will be read by metaprogramming system / compiler, but we want it to be ignored by JavaScript.

With existing JavaScript comments, at first glance, we could do something like this:
```ts
/*
interface Program {
  static void Main(string[] args)
}
*/
```
Our IDE may even color this code according to definition of this meta-language's interface. However, let's say we want to allow JavaScript comments within this block:
```ts
/*
interface Program {
  /* put a comment here about this method */
  static void Main(string[] args)
}
*/
```
What we've done is unexpectedly close our comment by including a comment in our meta language.

What hash comments allow you to do is use "nested blocks" and known tokens to avoid the "token soup" problem of [past "ignorable syntax" proposals](https://github.com/tc39/notes/blob/21ff7b482a627bf86ea0981eac60ceb5924ed1f1/meetings/2023-03/mar-22.md#type-annotations-delimiting-concerns).

With hash comments, we can include other JS tokens because JS is still parsing known (and unknown) tokens; it's just discarding them from the final program.

Therefore, this causes no parsing issues:
```ts
#(
    interface Program {
        /* put a comment here about this method */
        static void Main(string[] args)
        #(we can even add another hash comment)
    }
)
```

### Use cases for hash comments

Like CSS custom properties, the use cases for hash comments are numerous and endless. 

#### A type system

The most notable and common use case for hash comments might be creating a system of type-checking (as linting) with type annotations that are ignored by JavaScript. See [this doc](./example-type-system.md) for an example of a complete type system using hash comments.

#### A macro system

```ts
#|macro
function test(value #:string, thing #:Save<number>) {
    if (value === "yes") thing = 1;
    else if (value === "no") thing = 0;
    return thing;
}
expect(test#|!("yes", 343)).to.be.equal(1)
```

## Final thoughts

**Q: Could this be folded into the Type Annotations proposal?**

Possibly! They have some overlapping goals, such as establishing parsing rules for ignorable syntax. However, this proposal takes a much simpler approach, and doesn't seek to use TypeScript as a starting point, so they may diverge too much.

**Q: If hash comments are generic, couldn't two different JS files potentially be interpreted by two different meta-languages?**

That's always a possibility, but this proposal doesn't seek to resolve it. TypeScript already allows developers to type-check / not type-check JS files by default in a project, and to opt-in / opt-in individual `.js` files, and presumably, other meta-languages might have similar mechanisms.

It would be recommended that differing systems that "interpret" hash comments in a meaningful way develop a standard (not unlike JSDoc) to specify the "hash comment language".

For example:
```ts
// my-typeyscript-file.js
#|typeyscript

// Now all the hash comments in this file should be interpreted by the TypeyScript build checker
```

```ts
// my-typeyscript-file.js
#|flowy

// Now all the hash comments in this file should be interpreted by the Flowy build checker
```