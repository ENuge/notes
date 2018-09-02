You can write inline style elements within the `<head>` element:
```html
<head>
	<style>
		p {
			color: red;
			font-size: 20px;
		}
	</style>
</head>
```

`<link>` element (self-closing) lets you link HMTL and CSS files together.
IDs override the styles of tags and classes.
Style CSS using the lowest degree of specificity, so if an element needs a new style, it's easy to override.
Be very sparing in using ID selectors because of that.
chaining - when you combine multiple selectors in CSS like: `p.foo` (p elements with class foo)
Can also select nested elements like: `.main-list li` ...
More specific rules override less specific rules.
`!important` will override any style no matter how specific it is. Almost never use it!
Can separate css selectors with a comma to apply the same style to multiple things.

A CSS declaration: the `"color: blue;" bit.
color is the property, blue the value.

Times New Roman is the default typeface for all HTML elements.

Limit the number of typefaces used on a webpage to 2 or 3.

`text-align` aligns text to the element that holds it (its parent).

`color` - styles foreground color, background-color does same for background.

`opacity` - lets you set transparentness. So elements can have an overlay effect.

Can also set the `background-image` of an element.
For instance:
`background-image: url("images/mountains.jpg");`

# Box Model
`margin` vs `padding` - what's the difference?
`padding` - the amount of space between the content area and the border.

`border` - the thickness and style of the border surround the content area and the padding

`margin` - the amount of space between the border and the outside edge of the element.

	Other elements cannot come in your margin. Frequently used for just one side of an element,
	like `margin-right: 50px;` for instance.

Example border: `border: 3px solid coral;`
The default border is `medium none color;`, where color is the current color of the element.

`border-radius` lets you modify the corner of an element's border box.

`border-radius: 100%;` will create a perfect circle. Otherwise, you can use pixel radii.

Can also specify padding at each side:
`padding-top`, `padding-right`, etc.
Can also specify all four at once, top right bottom left like: `padding: 10px 6px 4px 2px;`

OR use the one that specifies just top and bottom: `padding: 10px 5px;`
Can do the same thing with margin to get all four sides at once. Or two and two.

Any element is by default set to the full width of its containing element.
You can't center an element that takes up the entire width!

So you need to specify a width as well as:
```html
margin: 0 auto;
width: 200px;
```

Padding never collapses. Vertical margins do. Horizontal margins don't!
The larger of the two vertical margins sets the distance between adjacent elements.
content can spill outside of a box if the max-height property is too low.
All the components of the box model combine to give an element's size.

`overflow` determines what happens when content spills outside the box.

`hidden` - hidden from view

`scroll` - scrollbar will be added

`visible` - will overflow. The default.

Many external stylesheets will start with this rule:
```css
* { /* <- * is the universal selector, it selects everything */
	margin: 0;
	padding: 0;
}
```
That will override the default margin and padding values of all HTML elements.
Because each browser will have its own defaults, and they aren't always consistent.

visibility: hides or displays elements.
It hides the contents, but not the element itself. It'll display a blank space.
`display: none` will actually be completely removed from the page!
By default, border/padding/margin add to the size of the element. Which makes sizing awkward.

The default box-sizing property used by browsers is `content-box`, which means the total size of element is the sum of border+padding+margin.
border-box includes border and padding within the sizing of the box, so changing those does
not change your box.

# Flow of HTML
Browser renders elements left to right, top to bottom.

block-level elements create a block the full width of their parent elements,
and they prevent other elements from appearing in the same horizontal space.

The default `position` property is `static`.
`position: relative` - position it relative to its default static position

top/bottom/left/right are the four offset properties. top moves the element
down by the specified amount, bottom moves it up etc. beacuse you're specifying
spacing.

`position: absolute` - all other elements on the page will ignore the element and act like
	it is not present on the page. It will be positioned relative to its closest positioned
	parent element. It will scroll with the rest of the document when the user scrolls.

`position: fixed` - it'll stay still as you scroll on the rest of the page. This removes it
	from the flow of the HTML document.

`z-index` controls the depth of html elements.
`z-index` does not work on static elements.
The greatest `z-index` value is the most "forward" element.

display property - controls the horizontal space used by an element

`display: inline` - such elements have a box that wraps tightly around their content.
	No newline required after each element. Height/width cannot be specified in css.
	You can make any element `display: inline;`

`display: block` - fill the entire width of the page by default. Can also set width.
	Displayed on its own line.

`display: inline-block` - can display next to each other. Can specify dimensions using
	height and width. Consider: images!

`display: float` - left or right, moves elements as far left or right as possible.
	Floated elements must have a width specified!

`clear: left/right/both/none` lets a floated element ignore what it might be bumping into
	and go there anyways, on the specified side.

Can specify colors in CSS in one of three ways: named colors, RGB, and HSL (hue/saturation/lightness)
Hex colors can be specified like so:
```html
background-color: #9932cc;
background-color: rgb(23,45,23);
```
Choose hex or decimal and stick with it, aside from named colors.
```html
color: hsl(120, 60%, 70%);
```
hue between 0 and 360, then the other two are percentages.
Saturation - the intensity or purity of the color. The lower the saturation, the grayer the color.

50% lightness is the normal lightness.
HSL makes it easy to pick colors that work well together - take the same lightness and saturation,
then mess with the hue.

Opaque - non-transparent.
`color: hsla(34, 100%, 50%, 0.1);` you now also have the alpha, or opacity.

Same for rgba.
Can also do `color: transparent;`
Everything has a white background, so opacity on a base element will change that element's background.

`font-family` lets you specify the font you want on the webpage.
font-weight can also be specified numerically from 100 to 900.
	Valid values are multiples of 100 within that range.
	400 is the default font-weight of most text.
	700 is bold, 300 is light.
	But which ones are available depends on the font you're using.

`font-style: italic;` lets you italicize text.
Can also specify the word-spacing in a body of text via:
`word-spacing: 0.3em;`
The default is usually 0.25em. 
Pretty uncommon to change.
`letter-spacing: 0.3em;` lets you adjust the kerning between letters.
`text-transform: uppercase;` and lowercase exists too.

`text-align: right;` (or center/left) aligns where your text goes. left is default.
line-height - lets you modify the leading of text.
leading is the space between the top of your character and the bottom of the character
	above it. Line height is the total height of your largest character plus the leading.
Often modify line-height apparently.
line-height can be unitless, a la 1.2, in which case it'll be a ratio of the line height.

rem units are relative to the root html element, em units are relative to their own element
	(so a parent could change the font-size and it would cascade down and affect your em's)

To use fallback fonts, just specify them in a line:
```html
font-family: "Garamond", "Times", serif;
```
The last bit means "use any serif font on the user's computer"

`@font-face` property lets you import fonts directly into CSS
Using the link from the Google fonts URL, you'll notice that it's literally just downloading those CSS rules.
You can also just host the fonts yourself - use a relative filepath (via the url(filepath) thing), and download
	the font yourself.
Here's an example:
```html
@font-face {
  font-family: "Roboto";
  src: url(fonts/Roboto.woff2) format('woff2'),
       url(fonts/Roboto.woff) format('woff'),
       url(fonts/Roboto.tff) format('truetype');
}
```
Where we specify multiple file formats because yay browser incompatibilities!

To set something to be a grid container, use `display: grid` or `display:inline-grid`
By default, each new grid item will be placed on its own line in the grid - it's a single-column grid!
`grid-template-columns: <size_in_px_per_column>;` - as a series of pixel values to define the number of columns.
(Can also set them as a percentage of the overall width)
Or mix and match, or even allow overflow!

grid-template-rows sets the number and size of the rows.
Or you can just use grid-template to replace both:
```html
grid-template: 200px 300px / 20% 10% 70% ;
```
sets the rows in the first two, the columns in the second. (So we have two rows, three columns, sized as mentioned)

`fr` is a new unit specifically for CSS grid. Things get split in parts, for instance:
1fr 2fr 1fr means effectively 25% 50% 25% but you can add another element easily
You can use fr with the other units, in which case it means a fraction of the remaining available space.

grid-template can actually take a function! repeat(..) was created for this reason!
repeat(3, 100px); will repeat that spec a given number of times
repeat can take multiple size parameters that it will then...repeat...

minmax(<minsize>, <maxsize>) can resize a particular element as the browser size changes, within the set bounds.

grid-row-gap and grid-column-gap let us put spaces between the columns in our grid.
But they do not add space at the beginning or end of the grid!

`grid-gap: 20px 10px;` sets the gap between rows to 20px and between columns to 10px

Grid items can be made up of more than just a single grid square. Noice!
grid-row-start: and grid-row-end, as css properties on grid items, tell it to
take up two rows, starting at 1. the numbers are grid lines.
They start at 1 because fuck you.
Can also do:
`grid-row: 1 / 6;` to start at 1 and end at 6.
`span` can be used for grid items. If you know where it'll start and how long it'll be, use span to specify the length,
instead of specifying the end value and maybe having an off-by-one problem.

Can also specify them all at once:
`grid-area: <grid_row_start> / <grid_col_start> / <grid_row_end> / <grid_col_end>;`

# Advanced CSS Grid
Can use grid-template-areas to name particular grid-areas. Like:
```html
grid-template-areas: "foo foo"
"bar bar"
"baz diffthing";
```

And then in the other classes you say: grid-area: "foo"; and it'll put it there. Dope!
Can use z-indexes to overlap CSS grid elements.

Row axis - also known as inline - left-to-right.
Column axis - also known as block - top-to-bottom.
justify-items: positions elements along the inline axis.
Takes one of four values: start, end, center, stretch (stretch to fill the row)
If you specify justify-items, those grid containers will not stretch the grid they are in.

`justify-content`: positions the entire grid along the row axis of its parent element.

`align-items`: positions items on the block axis. Similar rules.
Without setting align-items, they will span the height of the block item they are in.

align-content: positions the rows along the column axis.
justify-self and align-self work similarly, but just for that particular element.

CSS implicit grid - if you create more items than fit in your grid constraints, it'll auto-add them
according to its own algorithm (basically, filling row first, by default with grids only as tall as needed
to fill the content)

`grid-auto-rows` : sets the height of implicitly added rows
`grid-auto-columns` : sets the width of implicitly added columns
`grid-auto-flow`: sets whether new elements should be implicitly added to rows or columns.
	Has values row, column, and/or dense, which combines the earlier, but will fill unused areas earlier in the grid as it
	comes across them.