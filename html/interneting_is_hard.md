# Interneting Is Hard

Most of this should be review. But this site is beautiful, so I don't mind reading duplicative content. Plus, the sections on CSS are probably still worth reading.

Search engines care about your headings, so it's important you use them wisely.

`<ul>` elements should _only_ contain `<li>` elements - don't ever put a `<p>` tag at the top level within them, for instance.

"block-level elements" are also called "flow content". Block-level elements are always drawn on a new line.
The other type is "inline elements", or "phrasing content".

For instance, `<p>` is a block-level element, `<em>` is inline.

HTML should define the structure of your document, leaving the appearance to CSS. `<b>` and `<i>` tags were made obsolete in HTML5 for this reason.

Each `<br/>` tag you use should still convey meaning. It shouldn't be used to add a bunch of space between paragraphs...which I've definitely done in the past.

`<hr/>` carries less significance than the separation by a new heading element, but _more_ significance than a new paragraph.

The `<a>` element by itself doesn't even get styled to _look_ like a link.

You can have absolute, relative, or root-relative links.

Only use absolute links when referring users to external sites.

Relative links are best for referring to resources in the same folder or in a standalone section of your website.

Root-relative is relative to the root of the page...as one might expect. Aka start with a slash. So what you'd call absolute for file systems!

`target` attributes on `<a>` elements define where to display the page when a user clicks a link, `_blank` specifies a new tab or window.

JPG - use for photos and images with a lot of gradients. But they don't allow for transparent pixels.

Never use gifs for photos - they have a limited color palette. You also can't have semi-opaque pixels (but you can have fully transparent pixels). Use them for animation, otherwise use png.

PNGs are great for anything that's not a photo or animated.

By default, the `<img/>` element uses the inherited dimensions of its image file. Pixel-based image formats need to be twice as big as you want them to appear on retina displays.

Defining only one of `width` or `height` will cause the image to scale proportionally. Defining both will stretch the image. Never include a unit with `width`. It's usually better to set image dimensions with CSS so you can alter them with media queries.

Add descriptive `alt` attributes to your images for both vision-impaired users _and_ search engines.

Always define the `lang` attribute on the top-level `<html>` element of the page.

Always include a `<meta charset='UTF-8'/>` in the `<head>` element of your page.

`<`, `>`, and `&` have to be HTML-encoded because they mean something in the syntax of HTML itself. There's a ton of HTML entities like `&amp;` etc.

One vote in favor of curly quotes!

## Hello, CSS

A CSS _rule_ starts with a _selector_ that defines which HTML elements it applies to. The _declarations block_ is where you declare your properties on the selected elements.

CSS is _all_ about presentational information.

The `<link/>` element links your HTML to your CSS - it's a child of `<head>`. HTML glues together all your images, CSS, Javascript, and any other assets via similar things.

Use shades of grey, not black and white - it's just so much more readable.

`em` - the current font size of the element in question.
`px` - a pixel, regardless of pixel doubling etc.

Don't create redundant styles - just select multiple HTML elements at once (by separating them with commas).

`list-style-type` lets you alter the bullet icon, you probably want to define it on the parent `<ul>` or `<ol>` element. You can use `none` to have no such icon, which is useful for a vertical navigation item, for instance.

You can reuse stylesheets as much as you want - just have each html file link to it.

You can set `text-decoration` to `none` to remove the underlining from your links.
`text-align: justify` stretches the lines so that each line has equal width.
`font-weight` determines boldness, `font-style` determines whether italicized or not.

You can add page-specific styles with the `<style>` tag as a child of the `<head>` element.

inline styles trump all others in terms of cascading preference. You should avoid them at all costs!!

The order of the `<link>` elements matters - stylesheets that come later will override styles in earlier ones.

## CSS Box Model

Block boxes always appear below the previous block element. The width of block boxes is set based on the width of its parent container. The default height of block boxes is based on the content it contains.

Inline boxes don't affect vertical spacing - they're _not_ for determining layout, they're for setting stuff _inside_ of a block. The width of the inline element is determined by its content, _not_ its parent container.

You might set `display: block;` to format images, or links that are actually buttons.

If you add padding, the `background-color` will fill to include that padding. Everything inside the border gets the background!

Borders are very valuable for debugging - when you're not sure how a box is being rendered, add a `border: 1px solid red;` and fiddle, see what happens.

Margins are always transparent. Padding is included in the click area of an element, margins are not. Margins collapse vertically, padding does not (??).

Inline boxes _completely ignore_ the top and bottom margins of an element. The horizontal margins _do_ show though! But if you use padding, it _will_ use the full width and height - which makes sense, the inline element is the size of its content. It will not break the flow of the parent block, though, so it may overflow.

If you want to work with the vertical space of a page, you _must_ work with block-level elements.

Vertical margins collapse - so if you have two boxes next to each other, each with margins, though you might expect them to add up, what you'll get instead is _only_ the larger of the two margins. So in some way, you should think of the margin as the minimum amount of spacing between elements.

Only consecutive elements can collapse into each other - so you can avoid the behavior by adding an extra `<div>` that's like 1px of padding, and then the one above and below it won't collapse into each other.

If you only ever define top or bottom margins, you never have to worry about margin collapsing.

Flexbox doesn't have collapsing margins. Must learn!!

`<div>` is for block level content, `<span>` is for inline content.

`width` and `height` properties take precedence over the default size of a box's _content_. Note that content _does not_ include padding, border, or margin! So those will be additive on top of height+width. You can change that with the `box-sizing` property: `border-box` will force the width of the box to include padding and border. So then content width is determined automatically, if you were setting `width`. Using `border-box` is _a lot_ more intuitive and is considered a best practice.

`white-space` and `overflow` properties control how it'll handle text wrapping.

`text-align` only aligns the contents of block boxes - any inline boxes or text contained within it.

Three ways to horizontally align block-level elements:
  "auto-margins" for center alignment,
  "floats" for right/left alignment,
  "flexbox" for complete control over alignment

Setting the left and right margin of a block element to `auto` will center the element in its parent container. That will work as long as the block has an explicit width defined on it.

An example margin would be like:

```css
margin: 20px auto; /* Vertical Horizontal */
```

A block will, by default, be the width of its parent container.

Each browser will set some default css - you can override those by applying sane defaults on *, which matches every element.

## CSS Selectors

Use all hyphens and lowercase for class names. Use semantic class names, not declarative ones that describe what it does, because the latter limits your ability to change the style to something totally different.

`<div>` doesn't alter the semantic structure of a page, so it's great for defining the _presentational_ structure of a page - it won't impact what a crawler thinks of our page.

When there's two conflicting properties in a CSS file, the last one is always the one that gets applied.

"Descendant selectors" are when you select a block by describing one of its parents and itself, for instance `.synopsis em` will match on the `<em>` element within the element of class `synopsis`. You can be more strict and use a "child selector" with `.synopsis > em`, which will only match an `<em>` block whose immediate parent has the `.synopsis` class.

Try to use descendant and child selectors sparingly, don't use it to match super-deeply-nested items.

CSS "pseudo-classes" are classes built-in to each element that handle different states of that class, for instance `a:hover` on a link. The most common link pseudo-classes are:

- `:link` - A link the user has never visited.
- `:visited` - A link the user has visited before.
- `:hover` - A link with the user's mouse over it.
- `:active` - A link that's being pressed down by a mouse (or finger)

You can string together CSS pseudo-classes like `a:link:hover`.
If you don't define the pseudo-classes like `a:link`, it won't override the browser's default settings, and thus your more generic configuration won't apply.

You can get quite a lot of interactivity by just specifying the pseudo-classes.

There are lots of other pseudo-classes. `:last-of-type` selects the final element of a particular type in its parent element. You could do that to add more spacing to the last such element, or something like that.

Or you could use `:first-of-type`.

ID selectors are generally frowned upon for selecting within your CSS. Use class-selectors, so you get something maintainable. `id` attributes are the target of URL fragments, use them for that purpose only.

CSS specificity: all else equal, rules are applied top-to-bottom. But some CSS selectors are more specific than others.

ID selectors have higher specificity than class selectors.

Specifity: ID, pseudo-selectors, class selectors, element selectors...sort of:

- `#button-2`
- `.button:link`
- `a:link` and `.synopsis em` (they’re equal)
- `.button`
- `a`

Try to just use class names so you can keep everything within the same specificity.

## Floats

Floats let you put block-level elements side-by-side instead of on top of each other. Floats have mostly been replaced with flexbox in modern web dev.

Each block-level element fills 100% of its parent element's width. The `float` property gives us control over the horizontal position of an element.

The element underneath the `float:left`'ed element will flow around it. Floated boxes always align to the left or right of their parent element.

The floated thing is inside the other element. If you float multiple elements in the same direction, they'll stack horizontally.

Floated elements are removed from the normal flow of the page.

"Clearing" a float is when we tell a block to ignore any floats that appear before it. A cleared element will appear after any floats. `clear: both` clears both left and right floats. It'll make floated elements count towards the height of the `.page` container.

Clearing floats only fixes the height issue when there's an element _inside_ the container element that we can add a `clear` property to.

`overflow: hidden` can make sure the parent element itself contains any overflowing, floated elements.

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