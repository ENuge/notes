# You Don't Know JS: ES6 & Beyond

# Chapter 1: ES? Now & Future

(Was more or less a review of things stated in previous books, or that I'm already familiar with.)

# Chapter 2: Syntax

Totally undeclared variables are left as `undefined` if you try to access them. But `let`-defined variables, prior to that definition, are not initialized, and thus trying to access them in that temporal dead zone will get you a `ReferenceError`.

`let` used in a `for`-loop get re-initialized with each iteration of the loop - which means any closure occurring within that loop will close around the value of that iteration _at that time_, and not get overwritten with the next round of the loop.

A `const`-declared value will not be garbage-collected until you leave that lexical scope - because that value cannot ever be unset.

Function declarations inside of a block are now specified to be scoped to that block. The function _is_ hoisted within that block, though.

One example of the spread operator:
```javascript
function foo(x,y,z) {
  console.log( x, y, z );
}

foo( ...[1,2,3] );				// 1 2 3
```

You can also spread within another context:
```javascript
var a = [2,3,4];
var b = [ 1, ...a, 5 ];

console.log( b );					// [1,2,3,4,5]
```

You can also use it for more or less the opposite use case, as a rest operator:
```javascript
function foo(x, y, ...z) {
  console.log( x, y, z );
}

foo( 1, 2, 3, 4, 5 );			// 1 2 [3,4,5]
```

And you can use default values for arguments like so:
```javascript
function foo(x = 11, y = 31) {
  console.log( x + y );
}
```

You can't combine that with the spread operator, though. You can even use a function call as the default value. Variables mentioned in the parameter itself are parameter-scoped, which can be tricky if you're referencing previous values for your default.

You could use this fact to provide a default, no-op callback function:
```javascript
function ajax(url, cb = function(){}) {
  // ..
}

ajax( "http://some.url.1" );
```

Object destructuring destructures on _values_, not _properties_, which you can notice in the `ReferenceError`s here:
```javascript
function bar() {
  return {
    x: 4,
    y: 5,
    z: 6
  };
}
var { x: bam, y: baz, z: bap } = bar();

console.log( bam, baz, bap );		// 4 5 6
console.log( x, y, z );				// ReferenceError
```

You can use destructuring with anything that handles the assignment operator, it doesn't just need to be a variable. Here we set object properties:
```javascript
var o = {};

[o.a, o.b, o.c] = foo();
( { x: o.x, y: o.y, z: o.z } = bar() );

console.log( o.a, o.b, o.c );		// 1 2 3
console.log( o.x, o.y, o.z );		// 4 5 6
```

You could use this to map an object to an array and vice versa:
```javascript
var o1 = { a: 1, b: 2, c: 3 },
  a2 = [];

( { a: a2[0], b: a2[1], c: a2[2] } = o1 );

console.log( a2 );					// [1,2,3]
```

You could even use this to destructure objects with multiple arguments:
```javascript
var { a: X, a: Y } = { a: 1 };

X;	// 1
Y;	// 1
```

With destructuring, you don't have to assign all the values that are present:
```javascript
var [,b] = foo();
var { x, z } = bar();

console.log( b, x, z );				// 2 4 6
```

Object spread does not yet exist, like it does for arrays, but it's a level four proposal so _soon_.

Array and object destructuring works for parameters:
```javascript
function foo( [ x, y ] ) {
  console.log( x, y );
}

foo( [ 1, 2 ] );					// 1 2
foo( [ 1 ] );						// 1 undefined
foo( [] );							// undefined undefined

function foo( { x, y } ) {
  console.log( x, y );
}

foo( { y: 1, x: 2 } );				// 2 1
foo( { y: 42 } );					// undefined 42
foo( {} );							// undefined undefined
```

Note that that _basically_ gets you named arguments, and lets you not specify the first argument, for instance.

I skipped over some of the craziest bits of the object destructuring, I can always come back to them if I find it's relevant.

If your object is the same name as a lexical identifier, you can shorten it from `x: x` to `x`:
```javascript
var x = 2, y = 3,
  o = {
    x,
    y
  };

// Also with functions:
var o = {
  x() {
    // ..
  },
  y() {
    // ..
  }
}
```

That function shorthand can come back to bite you if you want to recurse, because now you need to make reference to the object property (which is nice and easy if you use => functions, not as easy if you want a dynamic `this`). Err, that doesn't work. The actual function is treated as an anonymous function expression, so that won't work. You need to access it on the object itself. Yucky.

You can assign the `Prototype` of an object within the object literal itself now:
```javascript
var o1 = {
  // ..
};

var o2 = {
  __proto__: o1,
  // ..
};
```

Though, people don't seem super enthusiastic about it - it's listed as something done for compatibility reasons. So, maybe avoid that and continue doing it explicitly after the fact with:
```javascript
Object.setPrototypeOf( o2, o1 );
```

If you use concise methods, ie the function shorthand, then you can call `super` to access the prototype-linked method:
```javascript
var o1 = {
  foo() {
    console.log( "o1:foo" );
  }
};

var o2 = {
  foo() {
    super.foo();
    console.log( "o2:foo" );
  }
};

Object.setPrototypeOf( o2, o1 );

o2.foo();		// o1:foo
        // o2:foo
```

Interpolated string literals can nicely be broken across multiple lines:
```javascript
var text =
`Now is the time for all good men
to come to the aid of their
country!`;

console.log( text );
// Now is the time for all good men
// to come to the aid of their
// country!
```

You can even nest interpolated string literals...if you ever find a good reason to do so. An interpolated string is lexically scoped where it appears, _not_ at all dynamically scoped.

You can have tagged string literals which are, uh, interesting:
```javascript
function foo(strings, ...values) {
  console.log( strings );
  console.log( values );
}

var desc = "awesome";

foo`Everything is ${desc}!`;
// [ "Everything is ", "!"]
// [ "awesome" ]
```

`foo` in that string literal tag can be any expression that returns a function, for instance:
```javascript
function bar() {
  return function foo(strings, ...values) {
    console.log( strings );
    console.log( values );
  }
}

var desc = "awesome";

bar()`Everything is ${desc}!`;
// [ "Everything is ", "!"]
// [ "awesome" ]
```

The first argument is an array of all the unevaluated bits of the string, the subsequent are all the already-evaluated interpolated expressions within the string. This seems like it would make handling translations _really nice_. Ayy, and that's basically the example! Nice. :D

You can use `String.raw` as a built-in tag to pass-through a raw string:
```javascript
console.log( `Hello\nWorld` );
// Hello
// World

console.log( String.raw`Hello\nWorld` );
// Hello\nWorld

String.raw`Hello\nWorld`.length;
// 12
```

Prior to ES6, regexes could only match on BMP characters due to JS using UTF-16!! For non-BMP characters, it would treat them as two separate characters, which has this unfortunate behavior:
```javascript
/^.-clef/ .test( "𝄞-clef" );		// false
/^.-clef/u.test( "𝄞-clef" );		// true
```

So, if you think there may be unicode, use the `u` form.

There's now "sticky mode" for regex, which can be shown by way of example (it is the one with a `y` appended):
```javascript
var re1 = /foo/,
  str = "++foo++";

re1.lastIndex;			// 0
re1.test( str );		// true
re1.lastIndex;			// 0 -- not updated

re1.lastIndex = 4;
re1.test( str );		// true -- ignored `lastIndex`
re1.lastIndex;			// 4 -- not updated

var re2 = /foo/y,		// <-- notice the `y` sticky flag
  str = "++foo++";

re2.lastIndex;			// 0
re2.test( str );		// false -- "foo" not found at `0`
re2.lastIndex;			// 0

re2.lastIndex = 2;
re2.test( str );		// true
re2.lastIndex;			// 5 -- updated to after previous match

re2.test( str );		// false
re2.lastIndex;			// 0 -- reset after previous match failure
```

`y` implies a virtual anchor at the beginning of the pattern that is relative to exactly the `lastIndex` position.

`^` is an anchor that _always_ refers to the beginning of the input, and is not in any way related to `y`. 

You can now get the flags from a regex with `.flags`:
```javascript
var re = /foo/ig;

re.flags;				// "gi"
```

You can use `toString()` to represent a string in any base from 2 to 32!:
```javascript
var a = 42;

a.toString();			// "42" -- also `a.toString( 10 )`
a.toString( 8 );		// "52"
a.toString( 16 );		// "2a"
a.toString( 2 );		// "101010"
```

Characters outside of the BMP plane are often referred to as _astral_ symbols, because that's the name given to the set of 16 planes of characters beyond the BMP.

You can now use Unicode code point escaping:
```javascript
var gclef = "\u{1D11E}";
console.log( gclef );			// "𝄞"
```

Previously, non-BMP characters had to be represented including the surrogate pairs, because you could only use the four-hexadecimal character version, which only covered the BMP: `var glef = "\uD834\uDD1E";`.

You can Unicode-normalize strings to make sure identical strings constructed in different ways show the same (for instance, using the diacritic surrogate pair to get é).
```javascript
var s1 = "o\u0302\u0300",
  s2 = s1.normalize(),
  s3 = "ồ";

s1.length;						// 3
s2.length;						// 1
s3.length;						// 1

s2 === s3;						// true
```

A _grapheme_ is a more precise description for a single character.

Symbols don't have a literal form - only the boxed form. You create them like so:
```javascript
var sym = Symbol("Some optional description");
typeof sym;
```

The description is solely used for the stringification representation of the symbol. The main point of a symbol is to create a string-like value that can't collide with any other value. Its value is hidden and unobtainable, and automatically generated. It could be a constant representing an event name:
```javascript
const EVT_LOGIN = Symbol( "event.login" );
```

Implicit string coercion of symbols is not allowed.

Or you can use it to create a singleton object (that only allows itself to be created once):
```javascript
const INSTANCE = Symbol( "instance" );

function HappyFace() {
  if (HappyFace[INSTANCE]) return HappyFace[INSTANCE];

  function smile() { .. }

  return HappyFace[INSTANCE] = {
    smile: smile
  };
}

var me = HappyFace(),
  you = HappyFace();

me === you;			// true
```

# Chapter 3: Organization

The API of an ES6 module is static. They are also singletons. 

The properties and methods you expose are actual bindings - if someone else changes those properties and methods, they are changed for you too.

`import` and `export` statements must always appear at the top-level scope of their usage, they cannot be put inside an `if` conditional for instance.

You can `export` things as an operator as well as the keyword form:
```javascript
export function foo() {
  // ..
}

export var awesome = 42;

var bar = [1,2,3];
export { bar };
export {bar as baz}; // can also rename a module member during named export
```

There is no global scope in modules.

The imported module will refer to the last-updated value for the export, for instance:
```javascript
var awesome = 42;
export { awesome };

// later
awesome = 100;
```

When `awesome` is imported, it will get the 100 value, _not_ 42. 

ES6 prefers modules having a single _default export_. `export default ..` takes an expression, _not_ an identifier, so if the variable pointing to the function changes, you keep the original function.

You can both have a `default` export and other named exports (but you probably _don't_ want to do it that way - in that case, do all named arguments and have people use your module via `import *`):
```javascript
export default function foo() { .. }

export function bar() { .. }
export function baz() { .. }
```

You cannot change the value of an imported module. 

You can re-export another module's exports:
```javascript
export { foo, bar } from "baz";
export { foo as FOO, bar as BAR } from "baz";
export * from "baz";
```

And you import like so:
```javascript
import { foo, bar, baz } from "foo";
```

The _module specifier_, "foo" in this case, must be a string literal - it cannot be a variable - because the whole point is for the whole thing to be statically analyzable.

You can rename imported functions, of course:
```javascript
import { foo as theFooFunc } from "foo";

theFooFunc();
```

The default export gets to not require the braces:
```javascript
import foo from "foo"; // not `import {foo} from "foo";`
```

Importing default and named functions would look like so, though you don't want to do it:
```javascript
import FOOFN, { bar, baz as BAZ } from "foo";
```

Only import what you need. But you can import everything as a single namespace if you really prefer that approach:
```javascript
import * as foo from "foo";

foo.bar();
foo.x;			// 42
foo.baz();
```

That * is required - you can't bring in just a couple of things to the `foo` namespace.

You can't change imported modules from that imported code:
```javascript
import foofn, * as hello from "world";

foofn = 42;			// (runtime) TypeError!
hello.default = 42;	// (runtime) TypeError!
hello.bar = 42;		// (runtime) TypeError!
hello.baz = 42;		// (runtime) TypeError!
```

Which is _good_, because otherwise that would enable extremely confusing code. ES6 modules in general are _intended_ to be static.

Declarations that occur as the result of an `import` are hoisted.

You could also just `import` like:
```javascript
import "foo";
```

That brings nothing into your own code. But it'll execute the module, which may be useful if it'll have side effects etc.

You can circularly reference two functions that call each other. All the static analysis will happen before either is executed, so all is groovy - it will act like they were in the same file and both were hoisted and available to the other.

`class` + `super` requires that you avoid all the expressiveness of using `this` (and doing things like borrowing functions).

The default `constructor` for a subclass will call `super(..)` on its parent, like C++ but unlike Java:
```javascript
constructor(...args) {
  super(...args);
}
```

`extends` actually fully works for extending built-in objects, like Arrays, which was pretty broken in subtle ways with Prototype-based inheritance (some values like `length` wouldn't autoupdate properly). You can also `extends` Error now.

`new.target` is a new property that tells you what class was actually called with `new`, or `undefined` if it wasn't `new`-gotten:
```javascript
class Foo {
  constructor() {
    console.log( "Foo: ", new.target.name );
  }
}

class Bar extends Foo {
  constructor() {
    super();
    console.log( "Bar: ", new.target.name );
  }
  baz() {
    console.log( "baz: ", new.target );
  }
}

var a = new Foo();
// Foo: Foo

var b = new Bar();
// Foo: Bar   <-- respects the `new` call-site
// Bar: Bar

b.baz();
// baz: undefined
```

There's also a `static` keyword, that's only on the class itself:
```javascript
class Foo {
  static cool() { console.log( "cool" ); }
  wow() { console.log( "wow" ); }
}

class Bar extends Foo {
  static awesome() {
    super.cool();
    console.log( "awesome" );
  }
  neat() {
    super.wow();
    console.log( "neat" );
  }
}

Foo.cool();					// "cool"
Bar.cool();					// "cool"
Bar.awesome();				// "cool"
              // "awesome"

var b = new Bar();
b.neat();					// "wow"
              // "neat"

b.awesome;					// undefined
b.cool;						// undefined
```

# Chapter 4: Async Flow Control

Really was just a shorter version of the book on Async & Promises.

# Chapter 5: Collections

A `TypedArray` is a way to provide structured access to binary data.

Endianness - whether the smallest byte starts or ends the multi-byte number. Nice review!

Maps - allow you to use non-string as keys, preventing this problem with using Objects for that purpose:
```javascript
var m = {};

var x = { id: 1 },
  y = { id: 2 };

m[x] = "foo";
m[y] = "bar";

m[x];							// "bar"
m[y];							// "bar"
```

This happens because `x` and `y` stringify to the same value. But Maps get around this:
```javascript
var m = new Map();

var x = { id: 1 },
  y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

m.get( x );						// "foo"
m.get( y );						// "bar"

m.delete(y);
```

Note: You need to use the `.get(..)` and `.set(..)` methods, not `[..]`. Use `size` to get its length, not `length`.
Also, the key will keep its reference to the object even if nothing else references that object, preventing the garbage collector from ever collecting that object until the map itself can be collected. With `WeakMaps`, if the object is garbage collected, the property on the map is removed. `WeakMaps` have a more limited interface. Also, note the values are held strongly, it's just the keys that are held weakly.

A Set is a collection of unique values, with duplicates ignored:
```javascript
var s = new Set();

var x = { id: 1 },
  y = { id: 2 };

s.add( x );
s.add( y );
s.add( x );

s.size;							// 2

s.delete( y );
s.size;							// 1

s.clear();
s.size;							// 0
```

`Set` doesn't have a `get` - just check if it has it and use it if so. Set uniqueness does not allow coercion.

# Chapter 6: API Additions

Use `Array.of(..)`, not `Array(..)`, as the way to construct an array. It avoids the issue of `Array(5)` creating a five-element pseudo-empty array, by instead creating an array of just that one item. `.of(..)` also makes it easier to create and initialize elements in a subclass:
```javascript
class MyCoolArray extends Array {
  sum() {
    return this.reduce( function reducer(acc,curr){
      return acc + curr;
    }, 0 );
  }
}

var x = new MyCoolArray( 3 );
x.length;						// 3 -- oops!
x.sum();						// 0 -- oops!

var y = [3];					// Array, not MyCoolArray
y.length;						// 1
y.sum();						// `sum` is not a function

var z = MyCoolArray.of( 3 );
z.length;						// 1
z.sum();						// 3
```

You can use `Array.from(..)` to copy an Array or get an Array from an Array-like Object:
```javascript
var arr = Array.from( arrLike );

var arrCopy = Array.from( arr );
```

`Array.from(..)` never produces empty slots. Nice!

You can also pass `Array.from(..)` a mapper like so:
```javascript
Array.from( arrLike, function mapper(val,idx){
  if (typeof val == "string") {
    return val.toUpperCase();
  }
  else {
    return idx;
  }
} );
```

`Number.isFinite` lets you check if a number is not Infinite, is a number, and isn't `NaN`:
```javascript
var a = NaN, b = Infinity, c = 42;

Number.isFinite( a );				// false
Number.isFinite( b );				// false

Number.isFinite( c );				// true
```

It doesn't do coercion:
```javascript
var a = "42";

isFinite( a );						// true
Number.isFinite( a );				// false
```

Also:
```javascript
Number.isInteger( 4 );				// true
Number.isInteger( 4.2 );			// false
```

You can repeat a `string` a specified number of times with:
```javascript
"foo".repeat( 3 );					// "foofoofoo"
```

There's also `startsWith`, `endsWith`, and `includes`:
```javascript
var palindrome = "step on no pets";

palindrome.startsWith( "step on" );	// true
palindrome.startsWith( "on", 5 );	// true

palindrome.endsWith( "no pets" );	// true
palindrome.endsWith( "no", 10 );	// true

palindrome.includes( "on" );		// true
palindrome.includes( "on", 6 );		// false
```

# Chapter 7: Metaprogramming

Metaprogramming focuses on: code inspecting itself, code modifying itself, or code modifying default language behavior so other code is affected.

The `name` property of a function is its lexical name, unless there is none (ie anonymous functions). Then it can infer it from its variable name.

A Proxy is an object that wraps or sits in front of another Object. You can register special handlers, called _traps_, on the proxy object when different operations are performed against the proxy.

Here's an example that sits in front of an object:
```javascript
var obj = { a: 1 },
  handlers = {
    get(target,key,context) {
      // note: target === obj,
      // context === pobj
      console.log( "accessing: ", key );
      return Reflect.get(
        target, key, context
      );
    }
  },
  pobj = new Proxy( obj, handlers );

obj.a;
// 1

pobj.a;
// accessing: a
// 1
```

You can create revokable Proxies, which means you can't access the object through the object after it's been revoked:
```javascript
var obj = { a: 1 },
  handlers = {
    get(target,key,context) {
      // note: target === obj,
      // context === pobj
      console.log( "accessing: ", key );
      return target[key];
    }
  },
  { proxy: pobj, revoke: prevoke } =
    Proxy.revocable( obj, handlers );

pobj.a;
// accessing: a
// 1

// later:
prevoke();

pobj.a;
// TypeError
```

You can use those to control and restrict access to your objects from other parts of your code - you give them a proxy, and when they should no longer be able to access it, you revoke it.

Here's an example of _proxy last_, which first interacts with the object, but sets the proxy in its prototype chain, so if the method etc is missing it can interact with the proxy (or the object can call up its prototype chain directly to interact with its proxy): 
```javascript
var handlers = {
    get(target,key,context) {
      return function() {
        context.speak(key + "!");
      };
    }
  },
  catchall = new Proxy( {}, handlers ),
  greeter = {
    speak(who = "someone") {
      console.log( "hello", who );
    }
  };

// setup `greeter` to fall back to `catchall`
Object.setPrototypeOf( greeter, catchall );

greeter.speak();				// hello someone
greeter.speak( "world" );		// hello world

greeter.everyone();				// hello everyone!
```

I can imagine that getting incredibly confusing incredibly quickly!!

You can use metaprogramming to test for the existence of certain syntax:
```javascript
try {
  new Function( "( () => {} )" );
  ARROW_FUNCS_ENABLED = true;
}
catch (err) {
  ARROW_FUNCS_ENABLED = false;
}
```

That, for instance, could be in the bootstrapping logic for your JS. It then loads an ES6 version of your code for the former and a transpiled version of the code for the latter. Which is called _split delivery_. It'll mean your ES6 code isn't suboptimal in browsers that can natively support ES6.

Well Known Symbols let you override intrinsic behaviors, such as coercion of an object to a primitive value.

# Chapter 8: Beyond ES6

(Lifting the curtain...)

The plan from here on out is to have particular features released when they're ready, not as the end result of a full vote and yearly update.

It looks likely they'll add an `Object.observe(..)` to the language, so you can can listen on any changes to a particular object like so:
```javascript
var obj = { a: 1, b: 2 };

Object.observe(
	obj,
	function(changes){
		for (var change of changes) {
			console.log( change );
		}
	},
	[ "add", "update", "delete" ]
);

obj.c = 3;
// { name: "c", object: obj, type: "add" }

obj.a = 42;
// { name: "a", object: obj, type: "update", oldValue: 1 }

delete obj.b;
// { name: "b", object: obj, type: "delete", oldValue: 2 }
```

Woo! Hé terminado 🎉

Bonus: Monads
Monads are immutable - all monad methods create new monad instances instead of mutating the monad's value itself. All monads will have `.map(..)`, `.chain(..)`, and `.ap(..)` methods similar to these:
```javascript
function Just(val) {
    return { map, chain, ap, inspect };

    // *********************

    function map(fn) { return Just( fn( val ) ); }

    // aka: bind, flatMap
    function chain(fn) { return fn( val ); }

    function ap(anotherMonad) { return anotherMonad.map( val ); }

    function inspect() {
        return `Just(${ val })`;
    }
}
```

When you call `.map(..)`, you get back another monad:
```javascript
var A = Just( 10 );
var B = A.map( v => v * 2 );

B.inspect();                // Just(20)
```

`chain(..)` (or `flatMap(..)`) operates similarly, but flattens the resulting monad:
```javascript
var A = Just( 10 );
var eleven = A.chain( v => v + 1 );

eleven;                     // 11
typeof eleven;              // "number"
```

`ap(..)` takes the value in a monad and applies to another monad's `map(..)`. `map(..)` always expects a function, so `ap(..)` must be a monad with a function as a value.

So given values, you could create a monad containing a function that contains of the values via currying:
```javascript
var A = Just( 10 );
var B = Just( 3 );
function sum(x,y) { return x + y; }

var C = A.map( curry( sum ) );

C.inspect();
// Just(function curried...)
```

And then you call `ap(..)` on `B`, passing it `C`:
```javascript
var D = C.ap( B );

D.inspect();                // Just(13)
```

A Monad by definition must be valid for all values, and cannot do any inspection of the value at all. A monad is how you define behavior around a value in a more declarative way. Don't use them because "it's the functional way", but use them when they're actually valuable.