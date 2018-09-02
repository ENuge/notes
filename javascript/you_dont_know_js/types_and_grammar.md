# You Don't Know JS: Types & Grammar

# Chapter 1: Types

`symbol` was added in ES6. There's a bug with the `typeof` for `null` that will probably never be fixed:
```javascript
typeof null === "object"; // true! (what a language...)
```

`typeof` for `function` returns "function", despite functions really just being callable objects.

The `.length` of a function is the number of formal parameters it is declared with.

Arrays are just objects.

Variables can hold any value at any time. **Only values have types.**

`undefined` means a variable has been declared but not yet set to a value.

`typeof b` even if you have not declared `b` at all still returns `undefined`. wat. You can use that to check if a variable has been loaded yet, by only trying to use it after first checking if its typeof is something other than `undefined`. Or you could just do `if (window.thing)` assuming `thing` is a global variable.

# Chapter 2: Values

Using `delete` on an array will remove that slot from the `array`, but not update its `length` property. Skipping slots in arrays (ie, defining the first and third index but not the second) has confusing behavior, so explicitly set every value, if only to `undefined`.

Save arrays for strictly numerically indexed values.

You can use `Array.from(..)` to get an array from an array-like thing.

JS `string`s are immutable, `array`s are quite mutable. So many `string` methods must return a new `string`, not modify in place.

You can borrow array functions and use them on strings:
```javascript
a.join;			// undefined
a.map;			// undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
  return v.toUpperCase() + ".";
} ).join( "" );

c;				// "f-o-o"
d;				// "F.O.O."
```

There is no true `integer` type, just `number`s. Very large or very small numbers are outputted in exponent form. You can't just do `42.<NumberOperation>`, because that dot could be a decimal. It has to be one of these:
```javascript
// invalid syntax:
42.toFixed( 3 );	// SyntaxError

// these are all valid:
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
```

Note that floating point numbers are inexact, so `0.1 + 0.2 !== 0.3`!!You can use `Number.EPSILON` to give yourself some tolerance for doing inexact comparisons like that.

ES6 adds `Number.MAX_SAFE_INTEGER` and `Number.MIN_SAFE_INTEGER` to specify the largest and smallest integers that are safely representable in code.

64 bit numbers cannot be represented accurately with the `number` type, so they should be stored in JS using `string` representation.

ES6 also adds `Number.isInteger(..)` and `Number.isSafeInteger(..)`.

Bitwise operators only work for 32-bit integer values. Fun.

`undefined` is an identifier and can be assigned to (don't ever do that! strict mode forbids it), `null` cannot. For both, their label is both its type and its value.

`void` voids out any value, so the result of the expression is always the `undefined` variable - without modifying the existing value:
```javascript
var a = 42;

console.log( void a, a ); // undefined 42
```

`NaN` is really "invalid number", not "failed number". Because it is after all a `number`. For instance:
```javascript
var a = 2 / "foo";		// NaN

typeof a === "number";	// true
```

`NaN` is never equal to another `NaN` value, it never equals itself. `NaN !== NaN`. The built-in `isNaN(..)` returns `true` for both `NaN` and anything that is not a `number`. JS is a disaster. Finally, ES6 added `Number.isNaN(..)` to do what you expect. `NaN` is the only value in the entire language that is not equal to itself.

`1/0` returns `Infinity`, `-1/0` returns `-Infinity`. Fun times. You can go from finite to infinite (numbers overflow to `Infinity` and `-Infinity`), but you can't go from infinite to finite.

`Infinity/Infinity // NaN`

JS has both a positive and negative 0. WAT:

```javascript
var b = 0 * -3; // -0
```

Addition and subtraction cannot result in a negative zero. FFS:

```javascript
var a = 0 / -3;
a.toString(); // "0"
JSON.stringify(a); // "0"
+"-0"; // -0
JSON.parse("-0"); // -0
```

But the comparison operators also lie to you!
```javascript
-0 == 0; // true
-0 === 0; // true
```

It's used so you can get magnitude and direction within the same `number` type. Because god forbid you just represented that with a more complex object that included that information...

You can use `Object.is(..)` in ES6-land to actually get sane behavior when comparing with `-0`, `NaN`, and everything else.

You cannot have a reference from one JS variable to another variable. A reference in JS points at a shared **value**, so all the references to that value are distinct - none of them are references to each other.

`null`, `undefined`, `string`, `number`, `boolean`, and `symbol` are all passed by value-copy. Compound values - `object`s and `function`s - _always_ create a copy of the _reference_ on assignment or passing. So any of the variables can change the underlying value:
```javascript
var a = 2;
var b = a; // `b` is always a copy of the value in `a`
b++;
a; // 2
b; // 3

var c = [1,2,3];
var d = c; // `d` is a reference to the shared `[1,2,3]` value
d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```

You cannot use one reference to change where another reference is pointed:
```javascript
var a = [1,2,3];
var b = a;
a; // [1,2,3]
b; // [1,2,3]

// later
b = [4,5,6];
a; // [1,2,3]
b; // [4,5,6]
```

So that can be confusing with function parameters, and it makes a _lot_ of sense why we try to stick to functional-based code that does not do much mutating objects:

```javascript
function foo(x) {
  x.push( 4 );
  x; // [1,2,3,4]

  // later
  x = [4,5,6];
  x.push( 7 );
  x; // [4,5,6,7]
}

var a = [1,2,3];

foo( a );

a; // [1,2,3,4]  not  [4,5,6,7]
```

To do that in the way you'd want, you'd have to do it like this:
```javascript
function foo(x) {
  x.push( 4 );
  x; // [1,2,3,4]

  // later
  x.length = 0; // empty existing array in-place
  x.push( 4, 5, 6, 7 );
  x; // [4,5,6,7]
}

var a = [1,2,3];

foo( a );

a; // [4,5,6,7]  not  [1,2,3,4]
```

So, stick to immutable constructs!! value-copy vs reference-copy is entirely controlled by the underlying value, there's nothing you can do about it.

`String`/`Boolean`/`Number` objects hold a scalar primitive that is immutable.

# Chapter 3: Natives

Values that are `typeof` `"object"` are tagged with an internal `[[Class]]` property, which can be revealed by borrowing `Object.prototype.toString(..)`:
```javascript
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"

Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"
Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

JS automatically _boxes_ (wraps) primitive values to give them the properties and methods you'd expect. Always prefer using the literal primitive values.:
```javascript
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

Just use the literal forms for everything.
`new Array(5)` will give you an array of length 5 (sorta). It's weird. It creates a "sparse array". Just use the primitve unless you _really_ need something like this (hint: you don't). Empty slots in these arrays are _not_ prepopulated with `undefined`.

Don't ever use empty slot arrays, do this instead:
```javascript
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

The one chill one to use is `new Regexp(..)` if you need to dynamically define a regular expression. But if you know it beforehand, use the `/.../g` form, so it can be precompiled and cached.

`Date(..)` and `Error(..)` are also very useful - they don't have native literal forms! `new Date(..)` with no arguments assumes current date/time. To get the current timestamp, do `Date.now()`.

The `Error(..)` object includes the current stack-trace in a read-only `.stack` property. You'd typically use with `throw`:
```javascript
function foo(x) {
  if (!x) {
    throw new Error( "x wasn't provided" );
  }
  // ..
}
```
There are several other specific-error-type natives builtin, but you probably shouldn't use them - leave those to the language itself.
Symbols are meant to be used like `_prefix`'ed methods or properties to denote internal-only, because they're more difficult to access from outside the module.

`Function.prototype` is a function, `RegExp.prototype` is a regex, and `Array.prototype` is an array. You probably don't want to modify what those prototype objects are though!! So you can use them as handy-dandy "default" values if the argument was otherwise not supplied:
```javascript
function isThisCool(vals,fn,rx) {
  vals = vals || Array.prototype;
  fn = fn || Function.prototype;
  rx = rx || RegExp.prototype;

  return rx.test(
    vals.map( fn ).join( "" )
  );
}

isThisCool();		// true

isThisCool(
  ["a","b","c"],
  function(v){ return v.toUpperCase(); },
  /D/
);					// false
```

# Appendix A 

JavaScript is basically the browser implementation of ECMAScript.

"Host objects" - objects that are autodefined or provided by your (browser, Node, etc.) environment.

You need to make sure not to collide with the HTML content on your page. Any `id` in an html element will auto-become a global variable:
```javascript
// in html you have:
// <div id="foo"></div>
if (typeof foo == "undefined") {
	foo = 42;		// will never run
}

console.log( foo );	// HTML element
```

**Never extend native prototypes** - you're asking for trouble. Unless you know your code will be the only code ever running in that environment.

_polyfill_ or _shim_ - Adding something to prototype that enables functionality added at a later point in time. Polyfills can't bridge the gap added by new syntax, which is why you need an ES6-to-old _transpiler_.
_prollyfill_ - a polyfill that anticipates a future change that will probably be of that form.
_shims_ check for conformity to existence tests of your added behavior, and uses the existed thing if it's fine, _polyfills_ check if it exists at all, and if not defines the new behavior.

Different `<script>..</script>` tags operate like independent programs, but they share a common global. Global variable scope _hoisting_ does not occur across those boundaries, though, meaning this won't work:
```javascript
<script>foo();</script>

<script>
  function foo() { .. }
</script>
```

If one script errors and fails, it will stop, but subsequent ones will continue executing using the same `global`.

You can dynamically create `<script>` elements and inject them into the DOM:
```javascript
var greeting = "Hello World";

var el = document.createElement( "script" );

el.text = "function foo(){ alert( greeting );\
 } setTimeout( foo, 1000 );";

document.body.appendChild( el );
```
If you do that, you cannot mention `"</script>"` anywhere in your program - even in a string - or it'll early-terminate your script. For example:
```javascript
<script>
  var code = "<script>alert( 'Hello World' )</script>"; // bad!
  var code = "<script>alert( 'Hello World' )</sc" + "ript>"; // good!
</script>
```
The code will be interpreted using the character set of the HTML page, _not_ of the character set of the JS file.

# Chapter 4: Coercion

"type casting" when done explicitly, "coercion" when done implicitly.
Coercion always operates on primitive values.

The `+` operator with either operand being a `string` will get the whole thing coerced to a `string`.

`toString(..)` will return what you expect for primitives (with exponent notation for very large or small `numbers`), and the `[[Class]]` property for complex objects. If an object has its own `toString(..)` defined, it'll use that instead. Arrays for instance override that:
```javascript
var a = [1,2,3];

a.toString(); // "1,2,3"
```

`toString(..)` can be called explicitly or implicitly.
Things that are not JSON-safe:
`undefined`, `function`, `symbol`, and `object`s with circular references. You can define a `toJSON` method that handles safe-serializing your otherwise non-serializable object or what have you. `toJSON()` should return a regular object, the subsequent `JSON.stringify(..)` handles the actual stringification.

You can also use `JSON.stringify(..)` and specify an excluder like so:
```javascript
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

`null` when coerced to a number becomes 0. But `undefined` becomes `NaN`.

Coercing numbers goes through the `toPrimitive` abstract operation, if the object has a `valueOf()` it uses that as the primitive representation, otherwise it uses its `toString()` and then coerces that string. If it doesn't define either, it's non-coercible. If the object has a `null` prototype (ie. created by `Object.create(null)`), it is non-coercible. One somewhat surprising coercion:
```javascript
Number( [ "abc" ] );	// NaN
```

`1` and `0` are not identical to `true` and `false`. You can coerce them to each other but they're not the same.

Everything except those specified in this list coerce to `true`, so this is the falsy values list:
- `undefined`
- `null`
- `false`
- `+0`, `-0`, `NaN`.
- `""`

That _includes_ the object versions of those primitives, which are `true`:
```javascript
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
var d = Boolean( a && b && c );

d; // true...WAT.
```

"Falsy objects" like `document.all` exist as things that should be deprecated and removed entirely, but there are too many legacy JS codebases relying on something like `if (document.all) {..}` to detect legacy IE that would break if you removed them entirely, so instead it returns an object that is falsey. The old codebases acted on `document.all` so it can't be removed, but it's falsey so you can detect around it and do other things.

```javascript
var a = "false";
var b = "0";
var c = "''";

var d = Boolean( a && b && c );

d; // true! Because those things are all strings, and the only falsey string is ""
```

To reiterate, these are all true:
```javascript
var a = [];				// empty array -- truthy or falsy?
var b = {};				// empty object -- truthy or falsy?
var c = function(){};	// empty function -- truthy or falsy?

var d = Boolean( a && b && c );

d;
```

Don't use `new` when explicitly converting, just make it an object of the type you want:
```javascript
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```

You could do other stuff to explicitly cast:
```javascript
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

The unary operator is fun: `1 + - + + + - + 1;	// 2`.
Coercing a date into a number gives the UNIX timestamp:
```javascript
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000
var timestamp = +new Date(); // the now timestamp
// or actually be explicit:
var timestamp = new Date().getTime();
// or in ES5:
var timestamp = Date.now();
```

`~` operator is bitwise NOT. Good times. That'll first do a `ToNumber` coercion. It only operates on 32 bit numbers, which is why these all 0 out:
```javascript
0 | -0;			// 0
0 | NaN;		// 0
0 | Infinity;	// 0
0 | -Infinity;	// 0
```

`~x` is roughly the same as `-(x+1)`. So, the only value that will become 0 by tilde'ing is -1, which is useful with C-style error value returning. You can use that as a (debatably) nicer way of handling things like `indexOf(..)`:
```javascript
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- truthy!

if (~a.indexOf( "lo" )) {	// true
	// found it!
}

~a.indexOf( "ol" );			// 0    <-- falsy!
```

`~~` operates like `Math.floor(..)` via the same mechanism as `!!` for boolean, but it only works on 32-bit numbers, and goes the wrong way with negative numbers:
```javascript
Math.floor( -49.6 );	// -50
~~-49.6;				// -49
```

Parsing a string is tolerant of non-negative numbers, coercion is not tolerant:
```javascript
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

`parseInt(..)` no longer guesses octal as of ES5, but you used to run into a lot of bugs where '09' would be parsed as 0, because 9 is outside of the range for octal numbers.

Woo, this is fun:
```javascript
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

Avoid this, because it encourages you to ignore the implicit `boolean` conversion going on with `a`:
```javascript
var b = a ? true : false;
```

Wut is this:
```javascript
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

It `valueOf()` for `Array` fails to find a simple primitive, so it calls `toString(..)` and concatenates those, hence our `1,23,4`.

You can use this sort of thing to coerce a number to a string:
```javascript
varv  a = 42;
var b = a + "";

b; // "42"
```

You can use `-` to coerce from `string` to `number`, because `-` is only defined on `number`:
```javascript
var a = "3.14";
var b = a - 0;

b; // 3.14
```

Another example:
```javascript
var a = [3];
var b = [1];

a - b; // 2
```

Both `a` and `b` are first coerced via `toString()` and then become `number`s, hence how we end up with the value 2.

The value of `||` and `&&` will always be the value of one of the two operands.

`symbol`s cannot be coerced to a `number`, and cannot be implicitly coerced to a `string`, but they can (both explicitly and implicitly) be coerced to a `boolean` - and are always `true`.

`==` allows coercion in the equality comparison and `===` does not! That's the most accurate way to think about it - not comparing types or things like that. When comparing two `object`s, `==` and `===` behave identically. Whoa! Neither coerces to anything.

With `==` comparison of a `string` and a `number`, the `string` is coerced to a `number`.

`==` comparison to `true` is surprising:
```javascript
var a = "42";
var b = true;

a == b;	// false
```

If either value is a `boolean`, it'll get coerced to a _number_ and it'll operate on that - then coercing the other value as needed. Hence how those are not equal! (You could compare against `false` and it'll still be `false` for the same reason.) Your brain wants it to be a `ToBoolean(..)` coercion, but it's actually a `ToNumber(..)` _first_. Just avoid using `== true` and `== false` with the loose equality.

`null == undefined` is `true`, by the spec. You can use that to see if you have `null` or `undefined`, but not actual falsy values, back from a function, by checking on `if (return_val == null)` for instance.

`object`s with `==` comparison get converted to primitives, then compared against the `string` or `number`. Some examples of gotchas with that, applying rules already discussed above in multiple contexts:
```javascript
var a = null;
var b = Object( a );	// same as `Object()`
a == b;					// false

var c = undefined;
var d = Object( c );	// same as `Object()`
c == d;					// false

var e = NaN;
var f = Object( e );	// same as `new Number( e )`
e == f;					// false
```

I think there are legitimate cases for when it makes sense for a language to protect the programmer from themselves - y'know, bounds checking for instance.

Lol at:
```javascript
[] == ![];		// true
```

The main beef you have is with `string` coercion of arrays, and how it implicitly coerces to the `toString()` contents of its internals. whitespace coerced via `ToNumber` becomes 0! Including `"\n"` for instance.

For `<`, if both values are `string`s as primitives, simple lexicographic comparison is done.

Another weird gotcha:
```javascript
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```
This is because the spec defines `a <=b` as `!(b < a)`. "less than or equal to" is really more like "not greater than" in JS.

# Chapter 5: Grammar

`var a = 3*6;` -> a declaration expression. Defined to return `undefined`.
`a = 3*6;` -> an assignment expression
A statement that is just an expression is an expression statement.

Statements all have completion values, even if it is just `undefined`.

Wut:
```javascript
var a = 42;
var b = a++;

a;	// 43
b;	// 42
```
This is because `++` first returns the original value, then does the incrementing.
With `++a`, the side effect happens before the return. 

You can use a `,` to string together multiple expression statements into a single statement:
```javascript
var a = 42, b;
b = ( a++, a );

a;	// 43
b;	// 43
```

An assignment expression results in the assigned value, which lets you chain assignments, like so:
```javascript
var a, b, c;

a = b = c = 42;
```

You can use assignment expression to assign things in conditional expressions:
```javascript
function vowels(str) {
	var matches;

	// pull out all the vowels
	if (str && (matches = str.match( /[aeiou]/g ))) {
		return matches;
	}
}

vowels( "Hello World" ); // ["e","o","o"]
```

This by itself is just a code block:
```javascript
// assume there's a `bar()` function defined

{
	foo: bar()
}

```

`foo` is actually a label - you can use it to jump to that location from `continue` and `break` statements ("continue the iteration from the labeled loop"). `break` would break out of the labeled loop. You can `break` out of non-loop labels, but you can't `continue` out of non-loop labels. You probably don't want to do these ever.

JSON is a subset of JS syntax, but it's not valid JS grammar by itself.

```javascript
{} + []; // 0
```
That is true because it's actually treating `{}` as a standalone block, and thus the addition is just coercing on `[]`.

You can use object destructuring in function parameters:
```javascript
function foo({ a, b, c }) {
	// no need for:
	// var a = obj.a, b = obj.b, c = obj.c
	console.log( a, b, c );
}

foo( {
	c: [1,2,3],
	a: 42,
	b: "foo"
} );	// 42 "foo" [1, 2, 3]
```

`else if` doesn't actually exist -> it's an `else` that's evaluating a single expression, the `if`, and thus doesn't require the wrapping braces. Whoah.

`,` has lower precedence than `=`. `,` has the lowest precedence. `&&` also has a higher precedence than `(..)`. `&&` is evaluated before `||`. `&&` is more precedent than `||`, and `||` is more precedent than `? :`.

Left-to-right processing is the default in JS. Also, being left-to-right associativity, which is slightly different, but effectively amounts to the same thing. `? :` is right-associative, which _does actually matter_:
```javascript
a ? b : (c ? d : e)
// that is different than:
(a ? b : c) ? d : e
```

With Simpson on the automatic semicolon insertion point - avoid it.

The temporal dead zone is where you are before a `let`/`const` variable declaration has occurred:
```javascript
{
	a = 2;		// ReferenceError!
	let a;
}
```

Another example of the temporal dead zone, the `b` reference in the default expression will error because it's in the TDZ, it will not pull in the global variable reference:
```javascript
var b = 3;

function foo( a = 42, b = a + b + 5 ) {
	// ..
}
```

If you pass `undefined` to a function whose params have a default arg, it'll use that default arg.

`try..catch/finally` - you don't actually need the `catch` block. `finally` clause code _always_ runs. That's neat - you can basically use the `finally` block like a callback:
```javascript
function foo() {
	try {
		return 42;
	}
	finally {
		console.log( "Hello" );
	}

	console.log( "never runs" );
}

console.log( foo() );
// Hello
// 42
```

A thrown exception in the `finally` block would override anything set in the `try` block. The `finally` in a `try`/`yield` only runs _after_ the generator is resumed. `finally` can override the `return` value of the `try` block, but only if it explicitly `return`s.

"I'd wager no amount of comments will redeem this code." -> lol.

`switch` defaults to `===` equality, to get coercion you need to hack the `switch` statement a bit:
```javascript
var a = "42";

switch (true) {
	case a == 10:
		console.log( "10 or '10'" );
		break;
	case a == 42:
		console.log( "42 or '42'" );
		break;
	default:
		// never gets here
}
// 42 or '42'
```

If you don't put your `default` at the end, it will get to it and just continue executing until it reaches a `break` or the end of all the `switch` statements:
```javascript
var a = 10;

switch (a) {
	case 1:
	case 2:
		// never gets here
	default:
		console.log( "default" );
	case 3:
		console.log( "3" );
		break;
	case 4:
		console.log( "4" );
}
// default
// 3
```