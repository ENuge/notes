# What The Flexbox

You start by saying your thing is `display: flex;`. It's a totally new way to lay out your website.

Flex containers go around the entire width of your page.

`inline-flex` is like an inline element, like a span. Just wraps around the content with the width it needs.

Being a child of a `display: flex` makes your things automatically flex items.

`100%` means by itself 100% _of the parent element_. `vw` and `vh` are of the viewport width and height! `vw` accounts for the sidebar.

The default direction is `flex-direction: row;` They stack side-by-side. `flex-direction: column;` is the opposite, they stack vertically (like you have a single column).

Main axis with `flex-direction: row;` is left to right. Main axis with column is top to bottom. There's also `row-reverse` and `column-reverse`.