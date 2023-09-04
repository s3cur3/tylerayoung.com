---
title: Vanilla CSS is Fine, Actually
layout: post
authors: ['tyler']
categories: ["Programming", "CSS"]
image: /assets/images/san-diego-2018/2018-12-23-14.17.38-1_hdr-sailboat.jpg
excerpt_separator: <!-- more -->
---

# TODO: Orthogonality is good: something that changes only color can combine with something that makes it a round rect

Over the years, I've done CSS in all the popular ways:

- 1990s-style vanilla CSS
- Preprocessor languages like Sass and Less
- CSS-in-JS
- Tailwind
- "Modern" vanilla CSS

If there's one thing I'm sure of, it's that 1990s-style vanilla CSS (including uses of Sass/Less that are morally equivalent) is the only _wrong_ way.

(Bear with me, I know I'm directly contradicting the title... we're getting there.)

<!-- more -->

What do I mean by that? The defining characteristic of the way we used to write CSS was a heady optimism about the beauty and expressiveness of the "cascading" part of cascading style sheets. "You see, each selector applies on top of past selectors, leading you to ever increasing levels of specificity..."

It was a dream, a beautiful dream. In practice, though, it led to the absolute worst kind of spaghetti code. It didn't even take much scale to get there---a small webapp, or even a small ecommerce site is enough to make this approach hell to maintain.

## Why did vanilla CSS suck?

Consider the most innocuous example:

```css
p { 
  margin-bottom: 1rem;
}
```

Like [Gwen Stacy plummeting from the Brooklyn Bridge](https://en.wikipedia.org/wiki/The_Night_Gwen_Stacy_Died), you're already dead, you just haven't yet hit the ground. What's the very next thing you do? Something like this:

```css
.footer p {
  font-size: 0.8rem;
  margin-bottom: 0;
}
```

Already, the overriding has begun, and as you pile on more and more places where you need a paragraph tag, you'll be engaging in a constant tug-of-war where you rely on properties to cascade from previous selectors, but new use cases require you to override those. In some cases, you'll *believe* it's safe to change the lower-precedence selectors... only to find (a month later) that in so doing, you broke some far distant layout that depended on those far-removed properties.

As the site continues to grow, you find more and more places where you pile on conflicting selectors and have to figure out why the results aren't what you expected. Suppose you have two classes used across your site, `.emphasis` and `.callout`.

```css
p {
  color: #000;
}

.emphasis {
  font-style: italic;
  padding: 4px;
  color: #fff;
  background: #f00;
}

.callout {
  border-radius: 8px;
  padding: 8px;
  background: #ccc;
}
```

Now you want an emphasized callout. In the dream of the '90s, you'd apply both to your container... but what will the colors end up being, and the padding? I know! I'll solve this with a specific rule for that combination!

```css
.emphasis.callout {
  padding: 8px;
  color: #fff;
  background: #f00;
}
```

Now we're on the path of increasing specificity. Soon we'll have a new, more specific variant for what happens when it's shown atop a dark background (`.dark .emphasis.callout`), another for when it's calling out a quotation (`blockquote.emphasis.callout`), and so on. We eventually come to writing something like `.landing-page .sidebar > .emphasis.callout` and throw in some inline `!important` when the weight of it all becomes too much to bear. Each step of the way, we tie more and more bits of the stylesheet together with hidden links where it's _incredibly_ difficult to verify where, if ever, a component specifically depends on some aspect of a rule.

This impossible-to-track interdependence---or more accurately, breaking the interdependence once and for all---is what got people excited about Tailwind and CSS-in-JS.

But it doesn't have to be this way!

## Style sheets without cascading

It's possible to write vanilla CSS in a way that gives you the advantages of Tailwind... but it just takes discipline. Specifically, it takes the discipline to avoid nesting or combining specifiers, instead preferring to _copy_ the relevant portions of thier contents into new specifiers. This is a component-focused approach. Instead of the mess above, you might end up with:

- `.emphasis`,
- `.callout`,
- `.emphasized-callout`,
- `.quoted-emphasized-callout`,
- `.landing-page-sidebar-emphasized-callout`,
- ...and so on.

You might lean on Sass or Less mixins to shuttle around the various "core" parts of these specifiers, but in my experience it's way more common to break working styles when maintaining this type of sharing than it is to fail to update places you should have when you just copied and pasted. Instead, CSS variables for sizes and colors get you a long way toward the benefits of code sharing.

I mentioned in this model, you'll want to avoid nesting. Something like `.landing-page .sidebar > .emphasis.callout` is the absolute worst case, and there's no excuse for it. There may, however, be a case for judicious application of nesting. My general guidelines are:

- Anything more than two levels deep of descendent specifiers is a smell
- Direct child specifiers (`>`) are a smell

```css
/* This is fine */
.alert p {
  color: #c00;
}

/* Ehhh... can we get a new class added to the span instead? */
.alert p span {
  font-style: italic;
}

/* What would you say you're doing here, bud? */
.alert > header > div {
  width: 100%;
}
```

By focusing on styles for whole components (and the very limited set of things those components can contain), we isolate components from being broken by changes to other similar bits of code.

## Why do this in 2023?

I should address the elephant in the room, Tailwind. I'm not a Tailwind hater---I like it, in fact, and hope to get to use it more in the future---but even when extracting out components for reuse (rather than styles), I find myself getting bogged down when a style gets complex. Applying 30 classes to a single tag is far from an extraordinary number of style rules to need, and I find it very difficult to parse all those on a single line. There's something about having the rules laid out vertically in CSS definitions that makes them more approachable for me.

(Plus, it's kind of nice that vanilla CSS has no dependencies.)

## Conclusion

A lot of conference speakers like to bang the drum about how great it is to simplify your code, but those talks tend to have more in common with paying lipservice to a shared dogma than providing concrete, actionable advice—to say nothing of a discussion of the tradeoffs. This style of CSS has clear tradeoffs—Copypasta! Less (theoretical) flexibility!—but I do believe the simplification is worth it.

Of course, as someone who writes frontend code really only as a means to an end, it strikes me as very possible there's a name for this style that I'm just not aware of. I certainly didn't invent it---I've picked up these disciplines from working with people who write much better CSS than me. [BEM (Block, Element Modifier)](https://getbem.com) is a related idea I rather like, but I've not used it enough in anger to have really internalized its "flow."









