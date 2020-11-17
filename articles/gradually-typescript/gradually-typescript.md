---
title: "Gradually TypeScript"
date: Last Modified
description:
    Lots of people migrate their projects to TypeScript in one great lurch
    forward. If you don't have that luxury, you're like me, and if you're like
    me, you're probably looking for some guidance. I hope this series of posts
    is that for you.
layout: layouts/article.njk
tags: post
featuredImg: /swift/wow-a-prototype.jpg
featuredImfgAlt: ""
permalink: "writing/gradually-typescript/"
---

# {{ title }}

I'm migrating a super large codebase over to TypeScript at
[my job](https://www.etsy.com/people/salemhilal). I generally love projects like
these because they're full of measurable, incremental improvements, and those
feel good. At the same time, I'm kinda scared of this one. There are almost 13
thousand files that need to eventually become TypeScript. There are also
somewhere between 300 and 500 engineers who need to learn how to write and
understand TypeScript. On top of all that, this is the first large-scale
migration I've planned and lead myself.

To stay sane and to help solidify my thoughts, I'm keeping a journal of useful
patterns and weird edge cases. I'm writing here (as opposed to publishing them
somehow through work) because none of these ideas are fully baked. Publishing
them through work means I have to pretend like I know what I'm talking about,
and I do not.

For a few reasons, this migration has to happen gradually (at least at first).
There's a lot of writing about how TypeScript can be adopted gradually. I
haven't found a ton of guides that talk about writing code for the in-between
period when a codebase is both full of TypeScript and untyped madness. I'm
calling this series "Gradually TypeScript."

## Articles in this series

Here's everything I've written so far, in no particular order.

{%- for post in collections.gradually-typescript %}

-   [{{post.data.title}}]({{post.url | url}})

{%- endfor %}

<!-- -   [Adding types to React components without removing their propTypes.](/writing/gradually-typescript/dont-use-proptypes-inferprops/) -->
<!-- To come
-   How I'm selling TypeScript to other people.
-   What am I reading and what am I trying to get other people to read.
    -   executeprogram
    -   essential typescript
-   Using Babel to compile TypeScript, and the caveats that come with it.
    -   you can't redeclare types and constants if they have the same name
    -   no enums
    -   super fast builds
-   Adding tests for types with ESLint
-   Use Generics with the goal of making return types easier -->
