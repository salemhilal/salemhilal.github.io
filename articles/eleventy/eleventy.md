---
title: A static blog probably doesn't need React.
date: Last Modified
description:
    If your goal is to make a personal blog, the right tool for the job is
    probably the most boring one.
layout: layouts/article.njk
tags: post
---

<!--
#### TODO:

-   Optimize images (jpg > png)
-   Prefetch links when you hover on them
-   Pull JS into its own file, optimize and bundle it, etc.
-   Figure out a footer
-->

# {{ title }}

I used to blog a whole lot. I started a Wordpress blog when I was in middle
school and ran it well through college. I spent more time maintaining the server
it ran on than I did actually writing anything. I liked tinkering, and I liked
computers. Now that I am a god-fearing engineer, I honestly want to spend as
little time as possible thinking about maintaining a website. I want to write
ideas down, and then I want them to show up on the internet.

At the same time, I still have standards. I want a site that I'd be proud of. I
want something that builds simply, performs well, and is easy to extend. I think
a lot about web performance and build systems at work --- this blog should be
proof that I'm worth my paycheck. It is with this attitude that I looked for
something to build my blog with.

I'm going to spoil the punchline a little bit: I did not build my blog as a
single-page application and I did not use a client-side framework. Instead, I
built a blog that relies almost entirely on HTML and CSS. There are [plenty of
critiques][spa-fatigue] on whether or not client-side web applications are
valuable in general, but I'm not here to talk about that. I'm here to argue that
if your goal is to build a blog, and you want to use something like React to do
it, you should think carefully about the costs that come with it. To illustrate
my point, I'm going to compare Gatsby, a very exciting React-based site-building
framework, with Eleventy, an incredibly boring one.

## Why not Gatsby?

If you ask Twitter for recommendations, someone will mention [Gatsby][gatsby].
Gatsby is a static-site generator built on top of React. React makes it
incredibly easy to develop rich applications or build out complex interactions.
A blog doesn't necessarily feel like it fits either of those qualifications, but
plenty of people have chosen Gatsby to run their blogs. As I mentioned, my goal
is to be able to write some markdown and have it show up painlessly on people's
screens. Gatsby is just as easy to use as any other static site generator, but
taking a closer look at how it runs in the browser didn't convince me that it
was a good fit. For the sake of this exploration, I picked apart the [Gatsby
project homepage][gatsby].

### Performance

Gatsby's homepage loads relatively quickly on my fast laptop with a fast
internet connection, but having "good performance" means it should also load
quickly for users without both of those luxuries. To start, I ran the mobile
[Lighthouse][lighthouse] test a few times on the page to get a sense of the
page's mobile performance. The test returned some less-than-ideal performance
marks, scoring around 74/100 fairly consistently. In particular, Gatsby choked
on both the "[Time To Interactive][tti]" (TTI) and the "[Max Potential First
Input Delay][mpfid]" (MPFID) measurements. TTI measures the time it takes for a
page to display useful content and be fully responsive to user inputs. MPFID is
a measure of the longest time a user would have to wait between when they tried
to interact with the page (a click or tap, for example) and when the page
actually responded to that interaction.

{% figure "/eleventy/gatsby-lighthouse.png" "I generated these scores from my laptop. Imagine what they might look like on a real phone." "A screenshot of Gatsby's lighthouse score, showing a performance score of 72 out of 100" %}

Why does Gatsby score so poorly on these measurements? For one, Gatsby takes
advantage of a React optimization called server-side rendering. Typically, React
applications can't display anything until both the React library and all of your
application code has been downloaded and run. Server-side rendering allows
applications to work around this constraint by rendering the application into
static HTML ahead of time. This HTML is served when the application is
requested. Once the application is displayed in the browser, its JavaScript is
downloaded and run, replacing the staticly-rendered site with a React-powered
one. This allows scores like First Contentful Paint and First Meaningful Paint
to be very low, as shown in the screenshot above. The tradeoff is that any
interactivity implemented in React still won't work until any JavaScript it
relies on has finished loading. This can take some time; React, as fast as it
may be, is still not an insignificant library to run, and while it executes, the
rest of the user experience might be slowed. The result is a page that feels
slow or even frozen to users, particularly those on slower devices.

Ironically enough, almost all of the functionality of the Gatsby homepage
remains if you disable JavaScript, and you save aroud a megabyte from the weight
of the initial page load by doing so. It takes care to create a site that
degrades gracefully when JavaScript is disabled, but if the site behaves almost
identically with and without a whole megabyte of JavaScript, you have to wonder
what benefit that JavaScript is actually providing.

### Browser support

[Gatsby's documentation][gatsby-support] claims that it supports whatever
browsers React supports, which is currently "At least IE9". At the time of
writing, Internet Explorer 11 actually renders Gatsby's homepage just fine, but
throws an exception (`fetch is undefined`) in the process. [Gatsby
Cloud][gatsby-cloud]'s homepage similarly loads as expected, but actually wipes
itself clean once that same error is thrown (presumably because the exception
lies somewhere in the rendering step).

{% figure "/eleventy/gatsby-cloud-ie11.png" "Minimalism at its purest." "A screenshot of Internet Explorer 11 loading gatsbyjs.com, which is blank. The console is shown with error messages." %}

Edge 16 doesn't even make it that far; when loading the Gatsby homepage, it ran
into unexplained errors trying to render the page, meaning nothing showed up at
all. For what it's worth, it did appear to load the Gatsby Cloud homepage just
fine.

{% figure "/eleventy/gatsby-edge-16.png" "To be fair, Edge 16 breaking isn't the most surprising thing ever." "A screenshot of Edge 16 with an error message stating that the page has a problem loading." %}

To be clear, I don't think IE11 or Edge 16 are particularly important browsers
to support. Neither of them represent a ton of traffic, and neither of them are
known for being reliable. At the same time, it's not hard to imagine a similar
bug in an old version of Mobile Safari or Samsung Internet. These particular
issues seem like bugs that I imagine will get fixed, but it's a good reminder
that with a reliance on JavaScript comes a reliance on polyfills and a careful
eye for browser support. Whatever value I'd get from a client-rendered blog
should be worth the headache that browser support entails.

### Mobile considerations

There's one brief but important part of performance that I want to mention:
network connectivity. Gatsby frequently loads additional components as you
browse around the site, and even as you scroll around an individual page. Here's
what my network tab looks like after scrolling to the bottom of the page:

{% figure "/eleventy/gatsby-component-requests.png" "For bonus points, check out the transfer size and the request count." "A screenshot of the network tab in Chrome's debugger on Gatsby's homepage, full of small fetch, json, and xhr requests." %}

These requests continue as you browse around the rest of the site; navigating to
"Docs" kicks off an additional 37 requests over the network, even with service
worker support and the browser cache enabled. They are exceptionally small, but
sporadic requests come with a cost: they [keep the antenna
active][mobile-battery] on mobile devices. In large applications, there's a ton
of value in loading JavaScript as it's necessary; it keeps the loading
experience light, it makes better use of the browser's cache, and it saves
bandwidth throughout a user's visit. However, as with everything, there is a
tradeoff. Network requests activate the mobile antenna, which in turn drains the
battery. No one likes to have their battery drained. In large applications,
there's a careful balance to be struck between loading content later and loading
it eagerly, but that's a consideration that's probably not important for a
personal blog. Unless I'm writing a literal novel, deferring the loading of my
content is likely an over-optimization for my blog.

## Eleventy (11ty) is great.

At this point, it should be clear that I didn't use Gatsby for this blog.
Instead, I built it using [Eleventy][11ty] (or 11ty). I started by looking for a
more [boring][boring] static-site generator than Gatsby. Eleventy came up
because like Gatsby, it's also trendy and the two get compared a lot. Even so,
it felt like the right fit for a bunch of reasons.

### It doesn't rely on a client-side library.

HTML and CSS are boring, but they are fast, work pretty much everywhere, are
[more resilient][minimal-html] to runtime errors, and are less intense to
process than JavaScript. On top of that, I know all too well how many different
and unexpected ways JavaScript can break. An unexpected JavaScript failure in
React could mean my blog doesn't render or navigate. Taking the time to remove
my reliance on JavaScript is a sort of insurance against these surprises, and it
saves me the trouble of monitoring client-side errors.

For comparison's sake, the 11ty homepage loads around 80kb (567kb if you scroll
to the bottom and load all the asynchronous content), has 6kb of JavaScript, and
has a beautiful Lighthouse score.

{% figure "/eleventy/11ty-lighthouse.png" "These scores make Dale Earnhardt Jr. look like drying paint." "A screenshot of Eleventy's Lighthouse score. Everything is perfect except for SEO, which scores 97 out of 100" %}

### It is really easy to extend.

I'm currently using Eleventy to minify and concatenate all of my CSS. I'm also
using a plugin to add syntax highlighting at build-time rather than at runtime,
which saves me plenty of space. Here, check it out:

```js
// This code snippet both shows off my syntax highlighting plugin and
// simultaneously demonstrates how to install it, which is extremely clever.
const highlight = require("@11ty/eleventy-plugin-syntaxhighlight");
module.exports = function (eleventyConfig) {
    eleventyConfig.addPlugin(highlight);
};
```

I also wanted to customize the HTML that the markdown engine spits out. I'm
using [Tachyons][tachyons] to style this site, which uses specifically-named
classes to style elements (like most CSS frameworks). I was able to add custom
classes to the output HTML by adding a plugin to the underlying markdown
library:

```js
const md = require("markdown-it");
const mdClass = require("@toycode/markdown-it-class");

module.exports = function (eleventyConfig) {
    eleventyConfig.setLibrary(
        "md",
        md().use(mdClass, {
            p: "f5 mb3 lh-copy o-90",
            ul: "f4 mb4 lh-copy o-90",
            h2: "mt5",
        })
    );
};
```

I know there are going to be times where I want to add a bit of interactivity to
a post, or when I'll want to add support for responsive images, and I'm not too
worried about Eleventy somehow preventing me from doing any of that.

### It is written in JavaScript.

Ok hear me out on this one. I'm comfortable enough with JavaScript to dig around
a bit when things don't work. I don't think JavaScript is a particularly
powerful or safe language, but it is one that I know fairly well. If I spent
most of my time writing Ruby or Go, I'm sure Jekyll or Hugo would work just as
well for me. If anything breaks when I'm trying to build my blog, I'm going to
be the one who has to fix it. And besides, if I start to run into the
limitations of the JavaScript language trying to build a static blog, I probably
have bigger problems on my hands.

## Use the right tool for the job.

If you want to play with a full fledged application that embodies the best that
web development has to offer, Gatsby is a fantastic choice. Gatsby takes
advantage of some bleeding-edge technologies and is clearly an impressive piece
of engineering. If you want to put a blog on the internet, you're probably
better off with something simpler. Adding JavaScript to the mix just increases
the list of things you need to worry about, and gets you little in return.

I could ride a helicopter to get to work, but wouldn't a bicycle be more fun?

[gatsby]: https://gatsbyjs.org/
[11ty]: https://11ty.dev/
[gatsby-cloud]: https://gatsbyjs.com/
[boring]: https://mcfunley.com/choose-boring-technology
[lighthouse]: https://developers.google.com/web/tools/lighthouse#devtools
[tti]: https://web.dev/interactive/
[mpfid]: https://web.dev/lighthouse-max-potential-fid/
[tachyons]: https://tachyons.io/
[gatsby-support]: https://www.gatsbyjs.org/docs/browser-support/
[fetch-support]: https://caniuse.com/#search=fetch
[minimal-html]: https://blog.notryan.com/013.txt
[mobile-battery]:
    https://www.oreilly.com/library/view/high-performance-browser/9781449344757/ch08.html
[spa-fatigue]: https://macwright.org/2020/05/10/spa-fatigue.html
