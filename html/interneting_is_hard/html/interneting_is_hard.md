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

# Hello, CSS

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

# CSS Box Model

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

# CSS Selectors

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

# Floats
Floats let you put block-level elements side-by-side instead of on top of each other. Floats have mostly been replaced with flexbox in modern web dev.

Each block-level element fills 100% of its parent element's width. The `float` property gives us control over the horizontal position of an element.

The element underneath the `float:left`'ed element will flow around it. Floated boxes always align to the left or right of their parent element.

The floated thing is inside the other element. If you float multiple elements in the same direction, they'll stack horizontally.

Floated elements are removed from the normal flow of the page.

"Clearing" a float is when we tell a block to ignore any floats that appear before it. A cleared element will appear after any floats. `clear: both` clears both left and right floats. It'll make floated elements count towards the height of the `.page` container.

Clearing floats only fixes the height issue when there's an element _inside_ the container element that we can add a `clear` property to.

`overflow: hidden` can make sure the parent element itself contains any overflowing, floated elements.