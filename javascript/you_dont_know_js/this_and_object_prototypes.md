# You Don't Know JS: Book Two

# Chapter 1

You can use `functionName.call(context);` to call a function against different contexts.

You could always explicitly pass along a context object instead of using `this`.

`this` does not refer to the function itself.

**NOTE**: Much of these notes are what is interesting or novel to me, but very much so do not represent the majority of the content in the book itself. Read that! I'm often noting digressions.

functions are just objects, so you can attach arbitrary other crap to a function.
```javascript
function foo() {
  foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
  // anonymous function (no name), cannot
  // refer to itself
  // aside from the deprecated `arguments.callee`
}, 10 );
```

You could give a function its own state to keep `this` on itself (if you do the `foo.call(context, params)` syntax), but that's still sidestepping the issue.

`this` does not exist to bridge lexical scopes. It's an entirely different context.

`this` is not author-time but runtime. It has everything to do with the manner in which a function is called.

When a function is invoked, an activation record, or execution context, is created, which contains all the stuff you'd expect about how it's called.

The only thing that matters for `this` binding is the actual callsite.

variables declared in the global scope are synonymous with global-object (`window`) properties of the same name.

In strict mode, the global object is not eligible for the default binding, so `this` is set to `undefined`. (and it depends on the strict mode of the contents of the function, not its callee)

Can have an implicit binding to a particular object etc. if the function is called
like: `obj.foo();`, even if `foo` was originally declared elsewhere and then made a member of `obj`. These are called _context objects_.
If you did `obj1.obj2.foo();` the context object for `this` would just be `obj2`.

The context object does not carry over with assignment, etc. So:
```javascript
var bar = obj.foo;
bar();
```
Will not call in `obj`'s context, but instead in the implicit binding of `bar`'s (as written, global). So calls can lose its implicit binding.
That same thing will happen if you were to pass `obj.foo` to a function, because it'll get assigned to the parameter which only sees it as a function reference.
No matter whether you wrote the function that you're passing a callback to or if it's built-in to the language.
Frequently, libraries make use of that implicit binding by implicitly calling your function with the context that triggered their event, etc. So that approach can be very bug-prone.

You can use `call(..)` and `apply(..)`, which both take an object as their first parameter, to use that object as the context for the call.
For instance:
```javascript
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

If you pass a primitive, it'll be wrapped in its object form, `new String(..)` etc. This is called "boxing".
A _hard binding_ is an explicit binding that cannot be unchanged.

TIL you can do something like this:
```javascript
var bar = function() {
  return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
```
Note we passed an argument to a function expression that doesn't accept any arguments! Whoa.

`Function.prototype.bind` as of ES5 provides that for free! So you can do something like:
```javascript
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Some libraries accept a context parameter, which will explicitly do the binding for you.

Constructors are just functions that happen to be called with the `new` operator in front of them. There's no connection to class-oriented functionality.
When you say `new Number(..)`, you're making a constructor call of the Number function, not calling a constructor. A slight difference!

When you make a constructor call, a new object is created, set to `this` automatically, and automatically returned unless you return a different object.
So `new` is another way to bind `this`, known as _new binding_.

_explicit binding_ takes precedence over _implicit binding_. Which makes sense!
_new binding_ is more precedent than _explicit binding_. (Which I expected, despite the author presenting it as shocking...)

You can also use `bind(..)` to partially apply arguments to a function - anything after the context arg will be passed as defaults to the subsequent arguments (as a list for the arg, and then that list is spread across the args).

`foo.call(null);` causes the explicit context to be ignored, and it will instead rely on the default binding. Which can be dangerous if that function then calls a standard library which modifies or messes around with `this`.

Instead of `null`, if you want to avoid the default binding pitfall with standard libraries, you could pass it a safe, empty object you never intend to use - the author likes calling it `ø`. (Editor's note: I'm not sure how I feel about non-ASCII names.) like so:
```javascript
var ø = Object.create(null);
```

This minorly blows my mind:
```javascript
function foo() {
  console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2 (!!)
```
The result value of the assignment expression is just a reference to the `foo` function object, so calling something on it just uses the default binding.
You can't re-explicitly bind a function, it can only be bound once.

`=>` functions lexically bound `this` at call-time, hence this:
```javascript
function foo() {
  // return an arrow function
  return (a) => {
    // `this` here is lexically adopted from `foo()`
    console.log( this.a );
  };
}

var obj1 = {
  a: 2
};

var obj2 = {
  a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```
The lexical binding of an arrow function cannot be overridden, even with `new`.

Here's a more practical example of why that matters:
```javascript
function foo() {
  setTimeout(() => {
    // `this` here is lexically adopted from `foo()`
    console.log( this.a );
  },100);
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

Otherwise, to keep `this.a` referring to `foo`'s scope, you'd need to use `bind` or `var self = this;` type stuff. They essentially disable the flexibility of `this` (but probably avoid a whole class of bugs). 

Fat arrow functions adopt the `this` binding from its enclosing function call, effectively a syntactic replacement for `var self = this;`.
The default binding for functions is `undefined` unless not in strict mode, in which case it's the global.

# Chapter 3: Objects

Object literals and constructed objects (see below) are identical:
```javascript
# Constructed form
var myObj = new Object();
myObj.key = value;
```
There are 6 primary types in JS: `number`, `string`, `object`, `boolean`, `null`, `undefined`. So the simple primitives are not themselves `object`s. Everything in JS is **not** an object!!

_complex primitives_  - special object subtypes. `function` is a subtype of `object`. A "callable object".
Arrays are also forms of objects.
All the object subtypes are just functions, `String`, `Array`, etc. They're not types, or classes!

So string primitives and String objects for instance are entirely different things. And that explains why `flow` complains when you type something as a `String` as opposed to a `string`:
```javascript
var strPrimitive = "I am a string";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "I am a string" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// inspect the object sub-type
Object.prototype.toString.call( strObject );	// [object String]
```

The primitive string is a primitive, immutable value. Whenever you try to check its length, access individual characters, etc. it coerces that `string` primitive to a `String`. **Use the literal form for a value, not its constructed form.**

`null` and `undefined` only have their primitive values, no object wrappers. `Date` values can only be constructed from their object form. `Object`s, `Array`s, `Function`s, and `Regexp`s are all objects regardless of which form is used.

The member variables on an object are referred to as its properties.
You can use either the dot or bracket syntax to access a property:
```javascript
var myObject = {
  a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

In objects, property names are always strings - including if you used a number, which would be coerced to a string. Actually, anything would be coerced to a string using that `Object.prototype.toString` method - probably not what you intend frequently!

ES6 adds _computed property names_ - put it in brackets, and then you can dynamically set your key when making an object literal:
```javascript
var prefix = "foo";

var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

There aren't really methods at all, just functions that are properties of an object.

Arrays are objects, so you can add properties on an array (!!). Adding named properties does not change the length of the array.
But if you add a property to an array that looks like a number, it will treat it as setting that numeric index on the array. Oh JS.
_Shallow copy_ (at least with respect to JS) would just copy the primitive and otherwise leave the references to the other objects.
Defaulting to deep copies leave lots of issues with circular references causing infinite copying.

ES6 has builtin shallow copying:
```javascript
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```
That is purely `=` style assignment.

As of ES5, there are now **property descriptors**:
```javascript
var myObject = {
  a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

You can define it manually:
```javascript
var myObject = {};

Object.defineProperty( myObject, "a", {
  value: 2,
  writable: true, // false would make overwriting error in strict mode
  configurable: true, // whether you can modify a descriptor (by calling `defineProperty` again)
  // it will TypeError if you try it.
  // it also makes it so you can't `delete` that property.
  enumerable: true // whether it shows up when you iterate over the object (ex. in a `for..in` loop)
} );

myObject.a; // 2
```

`delete` is primarily just an object property removal tool, not for GC, unless the object happens to now not have any references to the remaining properties in the object.

Most immutability is referring to shallow immutability, which means you cannot modify the top-level references, but the underlying objects those refer to may still be mutable.

`writable:false` and `configurable:false` effectively defines a constant on an object - cannot be modified or deleted.
You can prevent an object from having new things added to it:
```javascript
var myObject = {
  a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```
`Object.seal(obj)` prevents everything except modifying existing properties on the object.
`Object.freeze(obj)` does that, plus make them unmodifiable.

If you reference a property on an object that does not exist, you get `undefined`, **not** `ReferenceError`, which you might expect given the functionality with RHS variable lookups.

You can override the default `[[Get]]` and `[[Put]]` operations (which to my naked eye appear to work as expected) using getters and setters on a per-property level.
getters actually call a hidden function to get a value.
Setters likewise for setting a value.
With those, we now have an "accessor descriptor" as opposed to a "data descriptor".

Here's an example:
```javascript
var myObject = {
  // define a getter for `a`
  get a() {
    return 2;
  }
};

Object.defineProperty(
  myObject,	// target
  "b",		// property name
  {			// descriptor
    // define a getter for `b`
    get: function(){ return this.a * 2 },

    // make sure `b` shows up as an object property
    enumerable: true
  }
);

myObject.a; // 2

myObject.b; // 4
```
If you don't define a setter, a subsequent call to assign `a` or `b` in that example would just get thrown away. You almost always want to have both a getter and a setter.

You can check to see if an object has a certain property in a couple of ways:
```javascript
var myObject = {
  a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

`hasOwnProperty` does not check up the prototype chain, whereas `in` will.
You can always borrow another function and call it on a particular object.
`in` operator checks for keys, not values! That's important with arrays, because `2 in [1,3,5]` will not work as you expect!!

Even if something is not enumerable, it will show up if you use the `in` operator (but not the `for..in` structure).
So, with arrays, `for..in` will also enumerate properties on that array in addition to the objects actually contained in the array. -_-
`propertyIsEnumerable(..)` tells you if a given property is enumerable,
`Object.keys(..)` returns all enumerable properties,
`Object.getOwnPropertyNames(..)` returns all property names regardles of enumerability.
^ both of those only inspect the direct object, they don't go up the prototype chain.
`for..in` iterates over a list of all enumerable properties, including its `[[Prototype]]` chain.
The order of iteration over an object's properties is not guaranteed and may vary!

You can use `for..of` to iterate over the values of an array, or the values of an object if it defines an iterator. (Regular objects do not have a built-in `@@iterator` object.)

Here's an example of manually defining an iterator:
```javascript
var myObject = {
  a: 2,
  b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
  enumerable: false,
  writable: false,
  configurable: true,
  value: function() {
    var o = this;
    var idx = 0;
    var ks = Object.keys( o );
    return {
      next: function() {
        return {
          value: o[ks[idx++]],
          done: (idx > ks.length)
        };
      }
    };
  }
} );

// iterate `myObject` manually
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// iterate `myObject` with `for..of`
for (var v of myObject) {
  console.log( v );
}
// 2
// 3
```

I can see the power in defining custom iterators. I feel like I've never really needed to do that. Might speak to the sorts of problems I'm trying to solve...🤔

You can use those iterators to write generators:
```javascript
var randoms = {
  [Symbol.iterator]: function() {
    return {
      next: function() {
        return { value: Math.random() };
      }
    };
  }
};

var randoms_pool = [];
for (var n of randoms) {
  randoms_pool.push( n );

  // don't proceed unbounded!
  if (randoms_pool.length === 100) break;
}
```

# Chapter 4: Mixing Up "Class" Objects

Relative polymorphism - "look one level up" to find the method or property we're accessing and calling and perhaps overriding.

Classes themselves can call `super` (relatively referencing their superclass), but not the object instances created from those classes.

**Class inheritance implies copies.** The child class gets a copy of everything from its parent class, and accesses those when you use polymorphism to get relative references. It does not maintain a link to that parent class. It works the same way with multiple inheritance, you just get each parent copied into the child class.

JS simply does not provide a means for multiple inheritance, which avoids things like the Diamond Problem entirely.

JS does not automatically perform copy operations like that. So the model is entirely different. They're all linked. You don't copy, you just get references to the already existing functions.

JS forces you to have explicit absolute pseudo-polymorphism in each function that shadows a function from the parent object, for instance like so (with `Vehicle.drive(..)`):
```javascript
// vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
  for (var key in sourceObj) {
    // only copy if not already present
    if (!(key in targetObj)) {
      targetObj[key] = sourceObj[key];
    }
  }

  return targetObj;
}

var Vehicle = {
  engines: 1,

  ignition: function() {
    console.log( "Turning on my engine." );
  },

  drive: function() {
    this.ignition();
    console.log( "Steering and moving forward!" );
  }
};

var Car = mixin( Vehicle, {
  wheels: 4,

  drive: function() {
    Vehicle.drive.call( this );
    console.log( "Rolling on all " + this.wheels + " wheels!" );
  }
} );
```

It's ugly and brittle and error-prone, so...probably avoid it if you can. Note the mixin copying will give a reference to objects, so if one property object gets manipulated, they would both be operated on. And they both just share references to their common functions, those aren't copied over **at all**.
So if you modify one of the shared function objects, they both get modified.

Here's an alternate model of mixins in JS, called parasitic inheritance:
```javascript
// "Traditional JS Class" `Vehicle`
function Vehicle() {
  this.engines = 1;
}
Vehicle.prototype.ignition = function() {
  console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
  this.ignition();
  console.log( "Steering and moving forward!" );
};

// "Parasitic Class" `Car`
function Car() {
  // first, `car` is a `Vehicle`
  var car = new Vehicle();

  // now, let's modify our `car` to specialize it
  car.wheels = 4;

  // save a privileged reference to `Vehicle::drive()`
  var vehDrive = car.drive;

  // override `Vehicle::drive()`
  car.drive = function() {
    vehDrive.call( this );
    console.log( "Rolling on all " + this.wheels + " wheels!" );
  };

  return car;
}

var myCar = new Car();

myCar.drive();
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```
Which I think is reasonably straightforward to understand from the code itself.

You could also do implicit mixins:
```javascript
var Something = {
  cool: function() {
    this.greeting = "Hello World";
    this.count = this.count ? this.count + 1 : 1;
  }
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
  cool: function() {
    // implicit mixin of `Something` to `Another`
    Something.cool.call( this );
  }
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (not shared state with `Something`)
```

But now you have a brittle reference to `Something` within `Another`, so you probably don't want to go with that approach.
All of this is to show that faking classes introduces more problems than it solves. Intrigued by how the appendix will deal with the new class syntax, given that's now super prevalent.

# Chapter 5: Prototypes

Objects in JS have an internal property referred to in the spec as `[[Prototype]]`, which is a reference to another object.

You can use `Object.create(anotherObject);`, which lets you create an object with a prototype link to `anotherObject`.
`[[Get]]` will work its way up the prototype chain until it finds what it's looking for.

The top-end of every _normal_ prototype chain is `Object.prototype`. That `Object.prototype` has a whole bunch of utilities that objects can use, like `.toString()` and `.valueOf()`.

A property can `fooObject` will shadow a property on an object farther up `fooObject`'s prototype chain.

No shadowing occurs if a property higher in the chain is set to **read-only**.
If a foo is found higher on the `[[Prototype]]` chain and it's a setter, then the setter will always be called. The `foo` setter won't be redefined and no `foo` will be added to that object in the prototype chain.
In those cases, you _can_ use `Object.defineProperty(..)`.

Shadowing with methods leads to _explicit pseudo-polymorphism_, you can't really avoid the design structures we've already laid out. Try to avoid shadowing, it's complicated.

You can accidentally implicitly shadow variables like so, because `++` is really `foo = foo + 1` which does an assignment:
```javascript
var anotherObject = {
  a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // oops, implicit shadowing!

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```

**There's just the object.** There's no such thing as classes. The object itself describes what to do.

Whenever you create a `function Foo() {..}`, it gets a `Foo.prototype;` which points at an arbitrary empty object. Each object created from calling `new Foo()` will end up `[[Prototype]]`-linked to this "Foo dot prototype" object.

```javascript
function Foo() {
  // ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```
In JS, creating multiple instances of a class does not result in copying. Instead, in JS you create multiple objects that `[[Prototype]]` link to a common object. By default, no copying occurs. All the objects created from that common object are thus **linked** by their `.prototype`.

You don't make copies from one object to another, you make links between them.

Think of JS's object-linking mechanism as being _delegation_, where you can delegate a property access up the prototype chain.

The underlying prototype object has a `constructor` property that tells you who created it:
```javascript
function Foo() {
  // ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```
That `a.constructor` is just being delegated up the prototype chain (as you might expect). It _doesn't_ mean the `a` object _was constructed by_ `Foo`.  (In fact, it would be incorrect for the object created by `Object.create(newObject, Foo)`). It also breaks if you modify `a1.constructor`.

Note it's the same as the object created from calling the function.
Functions themselves are not constructors - you just happened to make a constructor call.

"constructor does not mean constructed by", as demonstrated a few different ways! And the `.constructor` property happens to be writable and configurable, so you can't really trust it! Avoid relying on `.constructor`!!

`Object.create(Foo)` creates a new object and links that new object's prototype to `Foo`.

`Bar.prototype = Foo.prototype;` would mean that if you downstream modify `Foo`, `Bar` would also get modified - because it's just a reference to the same object. (And vice versa.)

`Object.setPrototypeOf(..)` prototype-links in the way you'd expect and want - it was introduced in ES6.

So, here are the two ways to set the prototype link:
```javascript
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

Basically, don't use `instanceof` to reason about class semantics. The tl;dr of this entire book I think is that the JS language has had a lot of awkward shit imposed upon it by people that understand class inheritance but not the prototype chain and delegation at all.

You could do:
```javascript
Foo.prototype.isPrototypeOf(a); // true
```
`isPrototypeOf` answers: in the entire `[[Prototype]]` chain of `a`, does `Foo.prototype` ever appear? Or just `b.isPrototypeOf(c)`.

You can also do this in ES6:
```javascript
a.__proto__ === Foo.prototype; // true
```

Note, `__proto__` is a property on the `Object.prototype` that every object has in its prototype chain.
Don't change the prototype of an existing object, that would be very confusing!
Double-underscore like around `proto` are called "dunder proto" - if you want to look cool.

If you want a dictionary, used purely for data storage and thus not having any of the built-in `Object` properties in its prototype chain, you can do `var dict = Object.create(null);`, which always returns `false` to `instanceOf`.

And here is the author's simple way for creating linkages between two objects:
```javascript
var foo = {
  something: function() {
    console.log( "Tell me something good..." );
  }
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

Delegation is easier to reason about when it's made explicit, so uh yeah, try to do that:
```javascript
var anotherObject = {
  cool: function() {
    console.log( "cool!" );
  }
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
  this.cool(); // internal delegation!
};

myObject.doCool(); // "cool!"
```

All normal objects have the `Object.prototype` as the top of the prototype chain.

The constructor call is the most common way to get two objects linked to each other. No copies are made - objects are linked via an internal `[[Prototype]]` chain.

You can use delegation effectively by setting the property on the prototype, then constructor calling your function, and relying on the delegation to access the property from the prototype.

The function has the `prototype`, _not_ the object created via a constructor call of the function. You can add things to the function's prototype and set the prototype object of one function to be another object, so objects constructed by calling the second one will be prototype-linked to objects constructed by calling the first function.

`Bar.prototype = Object.create( Foo.prototype )` -> That says make a new prototype object that is itself `[[Prototype]]`-linked to `Foo`. And thus you have your linkage. You can then add properties on `Bar.prototype` and still rely on the delegation of methods in `Foo.prototype`.

# Chapter 6: Behavior Delegation

Heh, I feel like advice from this chapter will clash strongly with how I'll end up doing things day-to-day at work.

It's all about objects being linked to other objects. 

Here's an example of how you do behavior delegation as opposed to class composition:
```javascript
var Task = {
  setID: function(ID) { this.id = ID; },
  outputID: function() { console.log( this.id ); }
};

// make `XYZ` delegate to `Task`
var XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
  this.setID( ID );
  this.label = Label;
};

XYZ.outputTaskDetails = function() {
  this.outputID();
  console.log( this.label );
};
```

Note you're not dealing with classes or functions, just **objects**.

lol, "**OLOO**" - objects linked to other objects. Put state on the delegators, not the delegate. **avoid if at all possible naming things the same in the prototype chain**. Use less general method names intended to be overriden, do more self-documenting descriptive methods. Think of objects side-by-side, not vertically.

You can't do mutual delegation, sadly.

Simpson is desperate to justify behavior delegation as nicer than class-based inheritance, but I really am not convinced that it scales nicer for deeply-inherited classes. Also, the ES6 class syntax does look really nice...

Composition - pass in another object that you then use within your own object. Despite claiming the behavior delegation is purely among peers, there's a direction to that delegation. They are only truly peers if you never shadow a property. But then the more linked objects, the more enumerated properties you don't care about. With that said, OLOO is a nicer design pattern within the Login/Authentication example given.

You can use concise method declarations in any object literal, courtesy of ES6 (note the lack of `function` keywords in this example):
```javascript
// use nicer object literal syntax w/ concise methods!
var AuthController = {
  errors: [],
  checkAuth() {
    // ...
  },
  server(url,data) {
    // ...
  }
  // ...
};

// NOW, link `AuthController` to delegate to `LoginController`
Object.setPrototypeOf( AuthController, LoginController );
```

Pretty nice for both OLOO and class-based.

If `Foo` and `Bar` are two functions that you intend to get constructor-called, you can't do `Foo instanceOf Bar`, what you'd actually want is `Foo.prototype instanceof Bar`. Yucky.

Duck typing (whether a given object has a `then()` property on it) is how ES6 checks if a thing is a **"then"** Promise, so be very careful with using `then()` arbitrarily.

If you do OLOO-style stuff, you can then check things more directly like:
```javascript
Foo.isPrototypeOf( Bar ); // true
Object.getPrototypeOf( Bar ) === Foo; // true
```

# Appendix A

ES6 class syntax does not allow for class-level variables, only methods. Everything else is instance time. `class` is mostly just syntactic sugar. `class` is not static - because it's a reference to that prototype under the hood, changing the class will affect already-created instances of that class.

```javascript
class C {
  constructor() {
    // make sure to modify the shared state,
    // not set a shadowed property on the
    // instances!
    C.prototype.count++;

    // here, `this.count` works as expected
    // via delegation
    console.log( "Hello: " + this.count );
  }
}

// add a property for shared state directly to
// prototype object
C.prototype.count = 0;

var c1 = new C();
// Hello: 1

var c2 = new C();
// Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```

`class`-syntax, as noted above, is awkward for class properties - you need to explicitly set them outside of the declaration with `C.prototype.count = 0;` here, and then be careful about how it's accessed within the class.

You can still have accidental shadowing:
```javascript
class C {
  constructor(id) {
    // oops, gotcha, we're shadowing `id()` method
    // with a property value on the instance
    this.id = id;
  }
  id() {
    console.log( "Id: " + this.id );
  }
}

var c1 = new C( "c1" );
c1.id(); // TypeError -- `c1.id` is now the string "c1"
```

(Though I imagine a linter could catch and disallow that..?)

`super` is not bound dynamically, so if you start assigning functions to different objects you will need to rebound `super` for it to work properly. (You probably shouldn't do that with classes...?)

So you won't until that binding occurs be able to call things in the implicit binding of a different object, like so:
```javascript
class P {
  foo() { console.log( "P.foo" ); }
}

class C extends P {
  foo() {
    super();
  }
}

var c1 = new C();
c1.foo(); // "P.foo"

var D = {
  foo: function() { console.log( "D.foo" ); }
};

var E = {
  foo: C.prototype.foo
};

// Link E to D for delegation
Object.setPrototypeOf( E, D );

E.foo(); // "P.foo"
```

I think this summarizes best Simpson's beef with `class` syntax, which is fair:
> In other words, it's as if class is telling you: "dynamic is too hard, so it's probably not a good idea. Here's a static-looking syntax, so code your stuff statically."
> 
> What a sad commentary on JavaScript: dynamic is too hard, let's pretend to be (but not actually be!) static.

But I do find static code easier to reason about, and this is despite me spending my entire professional life in dynamic languages!