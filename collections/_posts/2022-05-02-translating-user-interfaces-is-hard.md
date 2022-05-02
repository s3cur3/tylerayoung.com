---
title: "Translating User Interfaces Is Way Harder Than You Think (And You’re Probably Doing It Wrong)"
layout: post
authors: ['tyler']
categories: ["Programming"]
#image: "/assets/images/san-diego-2021/img_2701-edited.jpg"
#image: "/assets/images/rogers/img_2940-edited.jpg"
image: "/assets/images/san-diego-2018/2018-12-22-14.07.52-1-hippo-and-baby.jpg"
summary: Say you want to build a tool to handle translating a user interface. Whether in a native app or on the web, this seems like it should be pretty simple. You'll take each string in your UI, run it through a function that looks the string up in the user's preferred language, and you're done.
---

<style>
    .prose blockquote p:first-of-type::before, .prose blockquote p:last-of-type::before {
        content: '';
    }
</style>

Say you want to build a tool to handle translating a user interface. Whether in a native app or on the web, this seems like it should be pretty simple. You'll take each string in your UI, run it through a function that looks the string up in the user's preferred language, and you're done.

Here's a first pass:

```js
button.label = translate("Click here");
```

Resulting in something like:

> <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mb-1">Click here</button> <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mb-1">Klicke hier</button> <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mb-1">ここをクリック</button> (etc.)<a href="#footnote-1" id="footnote-1-return">¹</a>

Perfect translation for every language, solved! (Assuming our translators are good, at least.)

That is, well, until we start to use it a bit...

## Prologue: How hard could this be?

Suppose we laid out a title bar with a button in a single line. On phones, that means we're limited to the device width for certain layouts. This works fine in English:

<blockquote>
<div style="width: 320px; padding: 1rem; white-space: nowrap; overflow: hidden" class="rounded bg-blue-100">
<span class="text-xl text-default" style="font-style: normal;">Welcome to our app!</span> <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded ml-8">FAQ</button>
</div>
</blockquote>

It even works in German, where the translation of FAQ is... "FAQ." 😄

But what happens in Spanish?

<blockquote>
<div style="width: 320px; padding: 1rem; white-space: nowrap; overflow: hidden" class="rounded bg-blue-100">
<span class="text-xl text-default" style="font-style: normal;">Bienvenido a nuestra aplicación</span> <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded ml-8">Preguntas frecuentes</button>
</div>
</blockquote>

Gah! Our FAQ button got pushed off the edge of the screen. Okay, let's shorten the title to just "welcome" in Spanish:

<blockquote>
<div style="width: 320px; padding: 1rem; white-space: nowrap; overflow: hidden" class="rounded bg-blue-100">
<span class="text-xl text-default" style="font-style: normal;">Bienvenidos</span> <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded ml-8">Preguntas frecuentes</button>
</div>
</blockquote>

This brings us to Hard Lesson #1, even before we encounter challenges with translation itself:

> Assume every layout will need to wrap.<a href="#footnote-2" id="footnote-2-return">²</a>

This is a good practice for other reasons, too---once you've gone through the work of making sure your layouts can handle strings of much greater lengths than you originally designed for, it becomes way easier to also support larger base text sizes. Age 40 comes for us all, and being able to increase text size is one of the most commonly used accessibility features.

## Context matters... it super, duper matters.

So we fixed our layouts, and we rolled out our first round of translations to much fanfare. But now we're starting to hear complaints from users that the translations aren't very good. That can't be, though! All our translations were vetted by independent (and expensive) _reverse_ translation as well, so we could verify on the round trip that they meant the same thing.

What on earth could have happened?

Consider this scenario (unfortunately inspired by real translation mishaps I have perpetrated):

Like many apps, we have a "&larr; Back" button in the upper left that returns you to the previous screen. We also have a screen in our application that shows you four different views of a landmark: left, right, front, and (importantly) back.

Here's what our translators saw:

```
. . .
Landmark Name=
Height=
Width=
Left=
Right=
Front=
Back=
. . .
```

Our tool that generates the strings to be translated tries to be smart and save us money, so it will only spit out "Back=" once.

The problem is that when we say "Back" meaning "the view from the back," that's a very different meaning from "go back to the previous screen." As an English speaker, the difference may seem subtle, but here's what our German translator supplied us:

```
. . .
Left=Von links
Right=Von rechts
Front=Von vorne
Back=Von hinten
. . .
```

The reverse translation back to English looks fine to a casual scan---something like "From the left," "from the right," "from the front," and "from behind." But oh! "From behind" is *not* a good translation of our Back *button*!

It gets worse. In the context of different visualizations of something, the German "von hinten" is innocuous. On its own, though, it has... shall we say... *erotic* connotations.

Well, that's Hard Lesson #2:

> Supplying translations without context, you might accidentally label your back button "Doggy style." 🤦‍♂️

## Getting fancy: Numeric substitutions

Translation is at its easiest when you have a set of totally static strings (though as we've seen, even that is easy to screw up). By sheer volume, those static strings make up the majority of UIs, but there will also be cases where we need to substitute dynamically-generated values into the text. After all, we don't really want to pay to individually translate "1 result," "2 results," "3 results," "4 results," . . .

As a first pass<a href="#footnote-3" id="footnote-3-return">³</a>, we might build an API like this:

```js
label.text = results.length === 0 ?
  translate("No results") :
  translate("%d results", results.length);
```

That won't do even in English though---we need a singular version so we don't end up saying "1 results." Okay, we can do that!

```js
label.text = results.length === 0 ?
  translate("No results") :
  translatePlural("%d result",
                  "%d results",
                  results.length);
```

Now we have a special translation for the zero, singular, and plural forms. Great!

But oh, it turns out a number of languages (most notably Arabic) have special forms for exactly two of a thing (the [dual form](https://en.wikipedia.org/wiki/Dual_(grammatical_number))). I guess if we want to extend our API to support those, we could do something like:

```js
label.text = results.length === 0 ?
  translate("No results") :
  translatePlural("%d result",
                  "%d results",
                  "%d results",
                  results.length);
```

Oof. That's not awesome.

That's when you learn about the [paucal form](https://en.wikipedia.org/wiki/Grammatical_number#Paucal), most notably found in Russian and Polish. These languages have different grammatical rules for "a few" (2-4 things) versus "many" (5+).

Rather than extending the API of `translatePlural()` to support all these cases, I think the best thing you can do is make the _translator_ side of the system support creating these rules.

Thus, Hard Lesson #3:

> There are no universal rules about how plural forms operate. 🤷‍♂️

## Riding into the danger zone: Other substitutions

While numbers are perhaps the most common dynamic substitution to make, they're far from the only kind you'll need. Consider this sentence:

> We've sent a confirmation code to jdoe@hotmail.com.

If you've gotten this far, you'll surely appreciate that we can't just do this:

```js
message =
  translate("We've sent a confirmation code to ")
   + user.email;
```

We need to support string substitutions. As a first pass, we'll try something like the [`printf()` format specifiers](https://en.wikipedia.org/wiki/Printf_format_string), so we can write:

```js
translate(
  "We've sent a confirmation code to %s",
  user.email
);
```

That works fine, and it allows the translators the freedom to move the substitution around within the phrase. If the language's grammar would put the email somewhere in the middle, as in German, the translation might look like this:

> Wir haben einen Bestätigungscode an jdoe@hotmail.com gesendet.

What happens, though, when you have two or more substitutions to make? Suppose we want to translate the string:

> You sent payment 1 of 5

Naively, we might try:

```js
translate(
  "You sent payment %d of %d",
  payment,
  totalPayments
)
```

Now we have a problem. In some languages (I think Japanese is one of them, but don't quote me on that), the grammatical way to render this amounts to "You sent, of the 5 payments, payment number 1." The problem with the `printf()`-style substitution is that the order of the substitutions is fixed. If we can't move the word order around, the Japanese (?) version will incorrectly get rendered as "You sent, of the 1 payments, payment number 5."

Hard Lesson #4:

> You need to support reordering all substitutions anywhere in the phrase.

With that in mind, we could have a format specifier that requires "named" substitutions like this:

> You sent payment %payment number% of %total payments%.

Then translators can then reorder the substitutions as needed, and all is well.

## The rabbit hole goes deeper

This is an incomplete look at the challenges of translation (and the broader concerns of [localization](https://lingohub.com/academy/glossary/translation-vs-localization)). Problems I didn't cover include, but are surely not limited to:

- Right-to-left languages present a whole new category of UI framework challenges.
- There are different cases for different substitution forms (e.g., Mozilla's Fluent [allows](https://blog.mozilla.org/l10n/2019/04/11/implementing-fluent-in-a-localization-tool/#attachment_1377) you to translate substituted words like brand names differently for the nominative versus genitive case).
- Your sentence with substitutions might need to account for the grammatical gender of the words being substituted, or the substitutions might need to account for the gender of the surrounding words.

This stuff is complicated, *really* complicated at scale. But I hope this post has been useful in opening your eyes at how many mistaken ideas we have about the structure of language, and maybe it'll help you think twice before you assume other languages will work like English.

If you liked this, you might also like these articles and videos in a similar vein:

- Patrick McKenzie's [Falsehoods Programmers Believe About Names](https://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/)
- Noah Sussman's [Falsehoods Programmers Believe About Time](http://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)
- Tom Scott's [The Problem with Time and Timezones](https://www.youtube.com/watch?v=-5wpm-gesOY)
- A meta-survey of other [Falsehoods Programmers Believe](https://spaceninja.com/2015/12/08/falsehoods-programmers-believe/)

## Footnotes

<a href="#footnote-1-return" id="footnote-1">¹</a> Disclaimer: I'm not qualified to translate any of the languages discussed here. I mostly ran stuff through Google Translate. Don't use my bad translations in your app. 🙃

<a href="#footnote-2-return" id="footnote-2">²</a> W3C has a [great resource](https://www.w3.org/International/articles/article-text-size) on text length in internationalization. They note that the problem is exacerbated with short English strings, both because they tend to have a higher expansion factor than longer strings post-translation and because we tend to design for them being short.

<a href="#footnote-3-return" id="footnote-3">³</a> It's actually generous to call this a first pass. In reality, I've seen otherwise smart programmers start with code more like this:

```js
label = 
  items.length + 
  " " +
  translate("results") +
  translate(" of ")
  query.totalItems;
```

😱
