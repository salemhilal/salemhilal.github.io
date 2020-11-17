---
title: "Gradually TypeScript: An argument for migrating slowly."
date: Last Modified
description:
    A lot of very smart people have had success migrating to TypeScript quickly.
    I have a few reasons why that's probably not the right move for me, yet.
layout: layouts/article.njk
tags:
    - post
    - gradually-typescript
featuredImg: /swift/wow-a-prototype.jpg
featuredImfgAlt: ""
permalink: "writing/gradually-typescript/why-are-you-doing-this/"
---

# {{ title }}

_This is part of a series on gradual TypeScript migration. You can find a list
of all the posts in the series [here](/writing/gradually-typescript/)._

**In short:** I am gradually migrating a large codebase to use TypeScript. The
decision to migrate gradually doesn't make sense without some context, and this
post has that context.

## What do you know about migrations?

The team I'm part of, the Web Platform team, pretty much only does large
migrations like this. Last year we swapped our home-grown build system [for
Webpack][jsconf], and we rolled out support for ES6 syntax. This year, we
migrated from AMD-style imports and exports to ES6 modules. Both of these
changes required touching pretty much all of the JavaScript in our 10+ year old
codebase. TypeScript is just another one of these changes.

Plenty of other companies have adopted TypeScript in their codebases.
Adoptability is one of the core appeals of TypeScript. Lots of places have
painted pictures of what migrating their site to TypeScript looks like. Notably,
[AirBnB](https://medium.com/airbnb-engineering/ts-migrate-a-tool-for-migrating-to-typescript-at-scale-cd23bfeb5cc)
migrated all of their files to TypeScript at once, added comments to ignore any
compilation errors, and gradually improved their types over time. This strategy
essentially allowed them to adopt the language quickly while deferring the
benefits of strongly-typed code, which is a reasonable trade-off.

## Why is your TypeScript migration different than every other TypeScript migration?

There are a few ingredients that makes this particular migration different:

1. My team is driving the migration effort — this isn't (yet) a
   leadership-driven initiative. No one is questioning TypeScript's benefits,
   but there are other, [more][going-up] [important][going-down]
   [things][going-uhhh] going on right now.

2. Our codebase is a decade and a half old, and most of that was from a time
   when JavaScript wasn't important. We only added ES6+ syntax features to our
   codebase in the last year, every page in the site somehow depends on jQuery,
   and we're still chugging along with Backbone.js in some places. These things
   aren't _inherintly_ bad (the site definitel works), but they give you a sense
   of how fast things can change in the frontend world if you blink for too
   long.

3. While some people are familiar with TypeScript, that isn't yet true for most
   of our engineering org. People were pretty eager for ES6+ syntax because
   almost everyone had been using it at their old jobs or in their bootcamps.
   The same isn't true with TypeScript; only a handful of people have experience
   with it.

4. Like every e-commerce site, everyone in the product org is heads-down on
   holiday season work.

5. Our team has
   [something of a code freeze](https://codeascraft.com/2016/11/16/code-slush-holidays/)
   on any infrastructure work we'd be doing otherwise.

To me, this means that I have a month or two of time to spend preparing our
codebase for TypeScript, as long as I do it in a way that doesn't make people's
jobs difficult. Once the busy season cools off for product engineering, it'll be
easier to convince people to start adding types to all of the projects they're
working on. This probably means that AirBnB's "all-at-once" strategy is off the
table, at least for now.

In addition, not everywhere is AirBnB. These notes will hopefully help others
decide what migration strategy works best for them.

## What's the plan?

The migration plan is essentially this:

1. Add types to core libraries, shared React components, and utilities
   — essentially everything but actual product features.
2. Prepare educational materials, documentation, and resources for people to use
   to get up to speed.
3. Pair a lot with teams that are starting new tracks of work, and build a
   community of engineers that are willing to do the same.
4. Make converting a single file from JS to TS as easy as possible (idealy using
   [a codemod][ts-migrate]).

With any luck, enough of the tools that product engineers most commonly reach
for will be TypeScript-ified shortly after the holiday season ends, and in the
nearer-term, they'll benefit from some of the things TypeScript can offer to
IDEs, like better autocompletion, inline documentation, and deprecation
warnings.

[jsconf]:
    https://codeascraft.com/2020/04/06/developing-in-a-monorepo-while-still-using-webpack/
[going-up]:
    https://www.equities.com/news/etsy-inc-etsy-soars-9-34-on-november-11
[going-down]:
    https://www.fool.com/investing/2020/11/09/why-shares-of-chewy-wayfair-and-etsy-are-tumbling/
[going-uhhh]:
    https://www.theverge.com/2020/7/10/21318991/etsy-mesh-masks-coronavirus-poor-protection
[ts-migrate]: https://github.com/airbnb/ts-migrate
