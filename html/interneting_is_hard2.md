# Interneting is Hard Notes

Percentages in CSS are relative to the width of their parent element. `overflow: hidden` will also make sure your content doesn't wrap around other elements.

## Flexbox

Stands for flexible box. Doh!

Reserve floats for when you need content to flow _around_ a box, or when you need to support legacy web browsers.

Flexbox consists of "flex containers" and "flex items". Flex containers group a bunch of flex items together and define how they're positioned. Every HTML element that's a direct child of a flex container is an "item".

A flex item can itself be a container for other flex items.

`display: flex;` tells the browsers to use the flexbox model.

`justify-content: center;` centers the flex items in that container. You typically manipulate items via their containers with flexbox. Other options are `flex-start`, `flex-end`, `space-around` (distributes space evenly between and around elements), and `space-between` (distributes space evenly, only between elements).

Flex containers can only position elements one level deep.

Margins with flexbox work just like in the world of floats.

Flex containers can manipulate the horizontal alignment of their items, which floats just cannot do.

`align-items` is pretty similar to `justify-content`, it can be `center`, `flex-start` (top of page), `flex-end` (bottom of page), `stretch`, and `baseline`. `stretch` makes the flex items take up the full content of the flex container, even if the content does not take up that much space. This lets you easily create equal-sized columns that have variable-sized content. Nice!

Use the `flex-wrap` property to create a grid. It's much more flexible and powerful than float-based grids. It'll force items that don't fit on page to get bumped to the next section. `justify-content` etc will operate on the last line, or the thing that actually needs to be spaced. Pretty nice! Having that same behavior with floats is super difficult.

You can use `flex-direction` to, in one fell swoop, switch the flexbox to be column-aligned as opposed to row-aligned. You can use that to _really_ easily make responsive designs - below a certain size, simply make it column-based. That's amazing.

When you rotate the direction of a container, you also rotate the direction of `justify-content`. Whoah.

`flex-direction` also lets you reverse the direction of the items via `row-reverse` and `column-reverse`. Note it does that on a per-row basis. That's dope!

An `order` property on a flex item defines its order relative to everything else. The default is 0, increasing its value moves it to the right. (And decreasing, to the left). `order` works across rows.

You can align flex items with `align-self`.

Flex items are _flexible_ - they can stretch and shrink to match the width of their container. `flex` property defines how elements should stretch and shrink - they default to 1, if one is 2 it'll grow twice as fast as the others.

`justify-content` tells you how to distribute the space between elements, `flex` tells you how to distribute the space into your elements themselves.

`flex: initial;` lets you combine flexible and static elements. It'll fall back to the element's specified `width`. `flex: 1;` will by itself ignore the explicitly-specified width.

auto-margins are like a divider for flex items in the same container. Auto-margins eat up all the extra space in the container. So you can specify `margin-left: auto;` on one flex item, and it / all subsequent items will start from the rightmost bit of the container.

## Advanced Positioning

"static positioning" - the normal flow of the page, which the box model/floats/flexbox all operate in that.

`position` property lets you alter the positioning scheme of an element. Defaults to `static`. It's called a "positioned element" if it doesn't have a value of _static_.

Relative positioning moves elements around relative to where they would normally appear in the static flow - useful for nudging boxes that are slightly off. Neither the surrounding elements are affected by the relative positioning - it's as if it's applied after the browser finishes laying out the page. Can use `top` and `left` to nudge it, as well as `bottom`/`right`.

Absolute positioning is relative to the entire browser window. It completely removes an element from the normal flow of the page. Actually, it's always relative to the closest container that is a positioned element - so if you make that parent `relative` it will be relative to that element. That's how you combine absolute positioning with static positioning.

"Fixed" elements don't scroll with the rest of the page.

Navigation elements should almost always be marked as a `<ul>` list instead of a bunch of `<div>` elements.

The default `z-index` value is 0, can go negative or positive. Only positioned elements pay attention to their `z-index` property. So you _need_ to make the `position: relative;`.

Relative positioning is for tweaking the positioning of an element without affecting its surrounding boxes. Absolute positioning takes your elements out of the normal flow of the page, relatively absolute has the absolute positioning behavior without taking the elements out of the flow of the page.

## Responsive Design

Separating content from presentation makes it really easy to have responsive designs.

"mobile-first" responsive design is often a good idea because the desktop version is likely to be the most complex. This lets you maximize code reuse because you can then special-case the desktop version.

flexbox's `order` property lets you change the order of things. That's dope - very useful for responsive designs.

You need the meta viewport line in order to make the mobile device not zoom out and try to encompass the entire desktop content site:

```html
<meta name='viewport' content='width=device-width, initial-scale=1.0, maximum-scale=1.0' />
```

## Responsive Images

SVGs "just work" in a responsive world. They automatically scale up for retina displays.

You need to add `width: 100%;` to make an image always fill the width of its container - it'll then resize as you resize the viewport. It'll maintain the aspect ratio, so the height will adjust accordingly.

An inline style setting the `max-width` of an image is acceptable because it's an inherent property of that image. They're really more content than presentation.

The `srcset` attribute on an `img` lets us present the high-res image only to retina devices, falling back to low-res for standard screens. Older browsers that don't understand `srcset` fall back to the `src` attribute. Serve the `1x` version to standard devices and the `2x` version to retina devices. But you often don't need to give the full image for retina mobile devices!

You can use `2000w` - w units in general - to specify the width of the photo to the browser, in pixels.

`sizes` is a series of media queries that tells you that image's width with that media query. So the browser can use how wide it should be, plus the width of the source information, to know which image to load.

You can also serve entirely different types of photos - for instance, serving a tall version for mobile devices and a wide version for laptops. `<picture>` is an element that wraps an `<img>`, `<source>` conditionally loads images based on media queries. The `media` attribute in the `source` determines when to load the image, and the `srcset` determines which image to load.

General recommendation: use `1x` and `2x` version of `srcset` for images less than 600px wide, `srcset` plus `sizes` for anything bigger, and `picture` and `section` when your designer requires something fancy.

## Semantic HTML

Basically, use named elements instead of just divs, so all the different sections of your page have a semantic meaning like `footer` etc. They're `divs` with meaning.

The actual value of the heading doesn't matter, but its value relative to its parent.

The `<article>` element represents an independent article in a web page, that could in theory be plucked out of the page in its entirety. You can have multiple `<article>` elements on one page. A `<section>` is like an `<article>` except it doesn't need to make sense outside the context of a page.

Each `<section>` can have its own set of `h1`-`h6` elements independent of the rest of the page - they have that top-level section. But don't rely on that - rely on the h1s because some devices don't treat the sections as such. smh.

Also, you need some heading element at the top of your section, otherwise that heading will appear untitled. You want to avoid that! Use `<section>` when it's a more descriptive `<div>`, not when it's a self-contained `<article>` and not if the div is purely for formatting.

Stick your navigation sections in a `<nav>`.

Wrap your header shit in a `<header>`. They're only associated with the nearest sectioning element, so you can have headers inside of sections as well as at the top of page, etc.

`<aside>` elements allow you to say "this bit is not directly part of the element" - like an ad within an `<article>`, for instance. You can also use it for quotations inside articles, things like that.

If in doubt, use a `<div>` instead. Misplaced semantic elements are worse than no such elements at all.

Google does in fact make use of at least some semantic HTML - the `time` element at least definitely shows on the page!

The `<address>` tag is intended for contact info (read: emails+phone numbers), _not_ a physical address.

`<figure>` is for an illustration, code snippet, diagram, etc. `<figcaption>` is an optional caption for that figure (including an `img`!). It's _not_ a replacement for `alt` text on an `img`.

schema.org is all about microdata and how you can use that to improve the context search engines have on your content - we should probably look into this for the apiref docs.

## Forms

A `label`'s `for` attribute must match the `id` attribute of its associated `<input>` element.

`input[type='text']` is a CSS attribute selector - it only matches things that have that attribute set to that value.

`flex-direction: row;` vs `column` lets you stack or put horizontally elements based on media queries.

You can give an `email` `type` for your `input` element, and the browser will provide some things like validation for free.

You can style `:valid` and `:invalid` states separately. Pseudoclasses are amazing! There's also one called `:focus` for when someone has clicked into your element.

Every radio button group should be wrapped in a `<fieldset>`, which is labeled with a `<legend>`. Have a `<label>` element with each radio button. Same `name` attribute for each radio button, but different values.

The `checked` attribute on an element will show whether the element will default to that value being checked.

`width: auto;` makes a box match the size of its contents (can do this to force it to all to be on one line).

## Web Typography

The `@font-face` rule lets you embed fonts into your stylesheets. Web fonts must always be included at the _top_ of our stylesheets.

The `font-family` name is how it'll be referred for the rest of the stylesheet. File paths will be relative to the css document, _not_ the HTML document. Which makes sense.

The browser will synthesize bold text if you don't provide the proper weight family, same with italics, but it doesn't look super nice compared to the actual hand-designed italic/bolded version.

Don't manually specify the bold and italic versions of your font, and then style `em` and `strong` to use those. Do use `font-style` and `font-weight` for that! Instead, add `font-style` and `font-weight` rules to the underlying styles.

Use your own fonts - it can be more performant if you do it right. Be sparing in your choice of fonts.

_Never_ use both an indent and a margin. Use one or the other to denote separation between paragraphs. Don't indent the first paragraph after a heading!

`text-align` and `text-indent` are the attributes we're working with here.

Long runs of text should almost always be left-aligned. Text alignment should be consistent through a page.

`line-height` is referred to in typography as leading because old print typography used to use thin strips of print to add spacing between lines. Use consistent spacing, give things enough space to breathe.

Line length or "measure" refers to the horizontal length of your text. Determined by the expected, `width` + `margin-left` + `margin-right`. Fixed-width layouts are often used to set a maximum line width.

Use a `font-size` between 14 and 20px. Use curly quotes and apostrophes, not flat quotes. Don't use `text-decoration: underline` except for hover states.