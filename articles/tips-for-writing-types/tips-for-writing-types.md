---
title: "Gradually TypeScript: Tips for writing types for your libraries."
date: 2020-11-25T17:00
description:
    You can ease TypeScript adoption by adding types for some of your more
    commonly-used modules. Writing types from scratch can be somewhat tricky,
    though. Here's a handful of tips that I find useful.
layout: layouts/article.njk
tags:
    - post
    - gradually-typescript
featuredImg: /swift/wow-a-prototype.jpg
featuredImfgAlt: ""
permalink: "writing/gradually-typescript/tips-for-writing-types/"
---

# {{ title }}

_This is part of a series on gradual TypeScript migration. You can find a list
of all the posts in the series [here](/writing/gradually-typescript/)._

When moving a repository from one language to another, it pays to make the new
language easy to adopt. Other engineers aren’t going to be as excited about
TypeScript as you are if it makes their jobs harder. For an engineer writing
TypeScript for the first time, their experience will depend a lot on the types
for the modules that they need to use.

Modules with good types are much easier to use than even the best untyped
modules. Good types can make writing code and exploring a codebase easier. They
can act as documentation and allow editors to better understand your code. At
the same time, types can complicate what would otherwise be simple code. Without
a little care, types can lead to lots of casting, complex generics, or unhelpful
hints. The types for your repository’s most used modules can make a big
difference in the success of a migration to TypeScript.

Writing useful types for modules can be rough, especially for older, more
complicated ones. Here are some tips I’ve found useful for writing types for
libraries from scratch. In general, I aim for usefulness and practicality over
type perfection.

---

## Add types that make the library useful.

Migrating to TypeScript should make writing reliable code easier, not harder.
When adding types to a module, it helps to think about what assumptions the
module makes first. Your types should not only enforce those assumptions, they
should take advantage of them.

Take this function, for example.

```ts
function getKeyFromObj(obj, key, defaultVal = null) {
    return obj[key] || defaultVal;
}
```

We could start by adding some types to this function that specify the bare
minimum that it needs to work:

```ts
function getKeyFromObj(
    obj: Record<string, unknown>,
    key: string,
    defaultVal = null
) {
    return obj[key] || defaultVal;
}
```

These types make the TypeScript checker satisfied by removing the implicit `any`
in the arguments, but they don't make this function easy to use. Specifically,
the return type doesn't infer anything useful from the function's inputs:

```ts
const t = getKeyFromObj({ hey: "there" }, "hey");
t.length; // ERROR: Object is of type 'unknown'.
(t as string).length; // Works, but gross.
```

In order for developers to use anything from `getKeyFromLength`, they'd have to
cast it into an appropriate type. That's confusing, and it takes typechecking
out of TypeScript's hands.

Instead, it helps to think about how this function gets used and to write types
that make using it easy. We could use a
[generic type](https://www.typescriptlang.org/docs/handbook/generics.html) `T`
to infer additional information about `obj`, which in turn would make our
function's return type smarter.

```ts
function getKeyFromObj<T>(
    obj: Record<string, T>,
    key: string,
    defaultVal: T | null = null
): T | null {
    return obj[key] || defaultVal;
}
```

The generic here lets us infer what the value of `obj` is supposed to be _and_
it lets us make sure that our `defaultVal` makes sense as well. Better yet, this
function makes sure that we check if the return type is `null`, too:

```ts
const t = getKeyFromObj({ hey: "there" }, "hey", "sup");
t.length; // ERROR: Object is possibly 'null'.
t?.length; // No problems here
```

We can even go a step further. In the example above, we can see that `t` should
always be a string since its default value is a string (and not `null`). Let's
add an
[overloaded function definition](https://www.typescriptlang.org/docs/handbook/functions.html#overloads)
to take this into account.

```ts
// This definition is for when we provide a default value.
function getKeyFromObj<T>(
    obj: Record<string, T>,
    key: string,
    defaultVal: T
): T;

// This definition is for when no default value is provided.
function getKeyFromObj<T>(obj: Record<string, T>, key: string): T | null;

// This is the actual implementation - its type is hidden.
function getKeyFromObj<T>(
    obj: Record<string, T>,
    key: string,
    defaultVal: T | null = null
): T | null {
    return obj[key] || defaultVal;
}
```

Now, our return value is `null` only if it has to be. Our function's types are
as accurate as possible without forcing developers to jump through any weird
hoops.

```ts
const t = getKeyFromObj({ hey: "there" }, "hey", "sup");
t.length; // No problems here.
const v = getKeyFromObj({ hey: "there" }, "hey");
v?.length; // No problems here either.
```

This example is pretty contrived, but it's a solid demonstration of the
difference between types that just typecheck and types that leave things better
than they found them.

## Make bad inputs impossible.

It's one thing for types to handle all sorts of different inputs. It's another
thing entirely to make sure that some inputs aren't allowed in the first place.
Being specific about what inputs are valid makes your types, and therefore your
code, safer and more reliable.

One mantra that comes up a lot is to
"[make illegal states unrepresentable](https://lexi-lambda.github.io/blog/2020/11/01/names-are-not-type-safety)",
which really just means "incorrect inputs should be a type error." This makes a
lot more sense with an example.

Say we have a function that renders an html string for some reason.

```ts
function getHeyoElem(tag) {
    return `<${tag}>Heyoooo</${tag}>`;
}
```

In JavaScript, the `tag` element here could be a lot of things, many of which
aren't actually useful.

```ts
getHeyoElem(420); // "<420>Heyoooo</420>"
```

A first pass on adding types to this function might look like this:

```ts
function getHeyoElem(tag: string): string {
    return `<${tag}>Heyoooo</${tag}>`;
}
```

That would prevent a lot of incorrect inputs, but it'd still allow a lot of
others:

```ts
getHeyoElem("420"); // "<420>Heyoooo</420>"
```

This is a "representation of a bad state" — our function spits out something
that we don't want, even though our function is ostensibly typed well enough.

We can fix this by making our types stricter. Rather than accepting any string
at all, let's accept strings that we know to be valid:

```ts
function getHeyoElem(tag: "div" | "span" | "p"): string {
    return `<${tag}>Heyoooo</${tag}>`;
}
```

This type makes sure that `getHeyoElem` can't be called with invalid inputs. At
the same time, there are a _lot_ of HTML tags. Listing them all out would be
rough. We can cheat a little by using the
[same type definition that TypeScript uses](https://www.typescriptlang.org/docs/handbook/dom-manipulation.html)
for `Document.createElement`:

```ts
type HTMLTags = keyof HTMLElementTagNameMap; // "a" | "abbr" | "address" | ...
function getHeyoElem(tag: HTMLTags): string {
    return `<${tag}>Heyoooo</${tag}>`;
}
getHeyoElem("420"); // ERROR: Argument of type '"420"' is not assignable to parameter of type "a" | "abbr" | "address" | ...
getHeyoElem("div"); // "<div>Heyoooo</div>
```

By tightening the type for `tag`, we've made sure that our function only accepts
valid arguments and only returns valid HTML strings. Better yet, we don't need
any runtime checks. We can trust that to be true as long as our code typechecks
and compiles.

## When in doubt, type the API and leave the source code alone.

The simplest way to migrate a module to TypeScript is to rename it from `.js` to
`.ts` and add types until all of the errors go away. In an ideal world, this is
how migrating every module would go. In a practical world, however, there are
plenty of modules that are difficult to add types to. Imagine writing types for
something as complex as jQuery without pulling your eyes out.

Rather than spending hours converting a complex module to TypeScript, it makes
more sense to just type its API. This approach decouples the usefulness of
typing from the specifics of the underlying implementation. Remember, the goal
in adding types isn't perfection, it's usefulness and usability. Perfection is
just an occasional side-effect.

For example, say we have a module with a function that solves the traveling
salesman problem in linear time:

```ts
// TravelingSalesman.js
export default function linearTravelingSalesman(listOfCities) {
    // Left as an exercise to the reader
}
```

It's implementation is pretty complex, but we don't have to necessarily port it
to TypeScript. Instead, we can create an ambient type file for the module which
lets us specify types for its exports.

```ts
// TravelingSalesman.d.ts
declare module "TravelingSalesman" {
    interface Coordinate {
        latitude: number;
        longitude: number;
    }
    export default function linearTravelingSalesman(
        listOfCities: Coordinate[]
    ): Coordinate[];
}
```

With something as simple as that type definition, anyone who imports
`"TravelingSalesman"` will get types that they can use right away.

This is the exact same approach TypeScript uses to add types for browser APIs,
and it's also the same method that the
[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) project
uses to add types to untyped libraries (including every major version of jQuery,
bless them).

Obviously, this approach has some trade-offs. Your source and your types are
separate from each other, so if the source code changes, the types might need to
change as well. Plus, there's always the chance that the type definitions don't
actually match the implementation correctly.

It's probably not surprising that having tests for your module makes writing
types for it a lot easier. Tests give you some use cases to validate your types
against, which help make sure your module's behavior matches its types. You can
also add tests for your types as well (more on that in the next section).

One gotcha: if you have `skipLibChecks` enabled in your tsconfig.json, **disable
it while you write your ambient type definitions**. It's not uncommon to enable
this option in larger repositories since it can speed up builds and type
checking. However, enabling that option means any `.d.ts` types you write won't
be typechecked as you work on them. No matter how clever you are, writing
TypeScript without the type checker's help is a pretty scary thing to do.

For example, a simple typo might make a crucial type undefined. If a type isn't
defined and `skipLibChecks` is enabled, the undefined type defaults to `any`.

```ts
declare module "TravelingSalesman" {
    // Oops
    interface Coordinatw {
        latitude: number;
        longitude: number;
    }

    // Coordinate is undefined and resolves to `any`.
    // That means that this function silently accepts and returns any[]
    export default function linearTravelingSalesman(
        listOfCities: Coordinate[]
    ): Coordinate[];
}
```

Because `any` silently works anywhere you use it, a typo can silently break your
types. Plus, because `any` is assignable to any other type, any type tests you
have will probably continue to pass. I can tell you from experience that this is
not a good time, especially when working on a library with a lot of
interconnceted types.

Once you're happy with your types, you can always re-enable `skipLibChecks` and
go about your day. To make sure that your types don't re-break in the future,
add a few tests that fail if any of your types become `any` (again, more on that
in the next section).

## Test your code _and_ your types.

Tests make every part of programming easier, and types are no exception to this
rule. Luckily, writing tests for types are even easier than writing tests for
the underlying code — you only have to validate the shape of your code, not how
it specifically behaves.

Let's say we have a function that takes a Unix timestamp (i.e. the number of
seconds since January 1st, 1970 UTC) and formats it into a human-readable
string:

```ts
function formatDateFromUnixTimestamp(time: number): string {
    // complex date-formatting logic
}
```

Testing this function might be complicated; dates are always full of edge cases,
all of which probably need a test. However, testing the _types_ of this function
is really, really easy.

Assuming your tests
[can be written in TypeScript](https://jestjs.io/docs/en/getting-started#using-typescript),
making assertions about your types is as simple as assigning the results of your
function to a typed constant:

```ts
const t: string = formatDateFromUnixTimestamp(42013376969);
```

As long as `formatDateFromUnixTimestamp` accepts types and returns strings, that
line of code won't have an error.

TypeScript has a `noUnusedLocals` option that marks unused variables as errors.
If you have that option set to `true`, `t` in the above example would be an
error. It can help to have a function that will "use" constants created for type
tests:

```ts
const use = (...args: unknown[]) => {};
const t: string = formatDateFromUnixTimestamp(42013376969);
use(t);
```

It usually makes sense to migrate both a module and its tests at the same time.
As you do, it's common to run into tests that validate things that TypeScript
could validate for you:

```ts
// Assert that bad inputs throw exceptions
expect(() => {
    formatDateFromUnixTimestamp(null);
}).toThrow();
```

If I'm using TypeScript, calling `formatDateFromUnixTimestamp` with `null` would
be a type error. However, if I'm using that function in an un-migrated
JavaScript file, I could still run into that exception. It makes sense to leave
that test in, but it makes writing tests in TypeScript tricky:

```ts
expect(() => {
    formatDateFromUnixTimestamp(null);
    // ERROR: Argument of type 'null' is not assignable to parameter of type 'number'.
}).toThrow();
```

In these cases, I use
[`@ts-expect-error` comments](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-9.html)
to tell TypeScript that there is supposed to be a type error and it is safe to
ignore.

```ts
expect(() => {
    // @ts-expect-error: invalid inputs should cause exceptions.
    formatDateFromUnixTimestamp(null);
}).toThrow();
```

These annotations are cool because they create errors if the code they annotate
doesn't have any problems. This can help detect changes in your type
definitions:

```ts
// @ts-expect-error
// ^ ERROR: Unused '@ts-expect-error' directive.
formatDateFromUnixTimestamp(1337);
```

If you use
[typescript-eslint](https://github.com/typescript-eslint/typescript-eslint) to
lint your TypeScript code, consider using the
[`ban-ts-comment` rule](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin/docs/rules/ban-ts-comment.md)
to ensure that `@ts-expect-error` annotations have comments:

```ts
// The next line has a linting error:
// @ts-expect-error
formatDateFromUnixTimestamp(null);

// No linting error on this next line.
// @ts-expect-error: I want this to break
formatDateFromUnixTimestamp(null);
```

Last but not least, it can be useful to write a few type tests that look for
`any`. As I mentioned in the last section, if you have `skipLibChecks` enabled
and you have types defined separately from their source code, those types can
become `any` if there are problems with them. Checking that those types don't
become `any` can save you a lot of grief.

Detecting `any` can be somewhat unintuitive, but
[this StackOverflow post](https://stackoverflow.com/a/55541672/444912) has an
interesting solution to the problem:

```ts
type IfAny<T, Y, N> = 0 extends 1 & T ? Y : N;
type IsAny<T> = IfAny<T, true, false>;

const t: IsAny<any> = true;
const u: IsAny<"hello"> = false;
const v: IsAny<0> = false;
```

In short, the only thing that will make `0` extend the union of `1` and
something else is `any`. This works because `any` is flexible in ways that other
types are not. Specifically, any union type that has an `any` as part of the
union is treated as `any` itself. Does `0` extend `1 & 0`? Nope. Does `0` extend
`1 & any`? You betcha.

In practice, tests against `any` look like this:

```ts
const t: IsAny<ReturnType<typeof formatDateFromUnixTimestamp>> = false;
```

If that `IsAny` type resolves to `true`, we'd get `const t: true = false`, and
that would be a type error.

## More readings to make you better at this hairy topic.

Writing types for libraries mostly takes a lot of practice. No one blog post is
going to give you all the tools you need to be good at it. However, reading a
lot of things from a variety of authors with a breadth of experience might be
worth your while. Here are a few I've gotten a lot out of:

-   Alexis King's
    "[Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)"
    and
    "[Names are not type safety](https://lexi-lambda.github.io/blog/2020/11/01/names-are-not-type-safety/)"
    were really helpful and fun to read. They're ostensibly about Haskell, but
    they both cover how types can make your code both simpler and safer.
-   TypeScript's
    "[Performance](https://github.com/microsoft/TypeScript/wiki/Performance)
    document touches on what makes types performant from the compiler's
    perspective. The whole document is super worth reading in general, but the
    first section in particular is a must-read before writing complex types.
-   A. Sharif
    [has some notes](https://dev.to/busypeoples/notes-on-typescript-inferring-react-proptypes-1g88)
    on the types for the PropTypes library and how it's able to turn a React
    PropTypes definition object into a TypeScript type. It's a specific library
    type with a specific use case, but it demonstrates how powerful TypeScript
    types can be.
