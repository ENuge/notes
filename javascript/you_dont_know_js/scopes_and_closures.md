# You Don't Know JS

"You might see it in your code but homie you don't know JS" - someone, once.

_lexing_ vs _tokenizing_ - _lexing_ is stateful, tokenizing is not. So if a given thing should be considered a distinct token or just part of another token depends on state, it's _lexing_.

You then take that array of tokens and put it in an AST, an _Abstract Syntax Tree_.

JS compiles code before it executes it - but it happens to compile it just before that code is then executed. So it's a JIT, basically.

### Quiz
LHS lookups: set a equal to 2, set b equal to a, set all that equal to c
RHS lookups: invoking foo, find a, find a, find b,

If an RHS lookup occurs on something not found in any scopes, it's an undeclared variable, hence a `ReferenceError`.

If an LHS lookup does not exist in any scopes, unless you're in strict mode, it will create a variable of that name *in the global scope*.
In strict mode, it also throws a `ReferenceError`.

`ReferenceError` means a problem with Scope lookup. `TypeError` means the Scope lookup succeeded, but there was an illegal/impossible action attempted against the result.

Assignment = LHS, retrieval = RHS.

JavaScript modules are automatically in strict mode, nice!

TIL of JS's with semantics...wut. lol.

## Chapter 2

Two ways of handling scoping - lexical scope and dynamic scope. Almost everything, aside from like Bash, uses lexical scope.

lexical scope - defined at lexing time, and thus by you at write time, and thus mostly set in stone.

The scoping bubbles are strictly nested.

Scope lookup stops once it finds the first match. You can shadow an identifier by specifying it at multiple levels of scope.

Global variables are also automatically properties of the global object (`window` in browsers), so you can reference a global as a property reference of the global object.

For a function, the lexical scope is **only** defined by where the function was declared.

### Cheating Lexical Scope

Cheating lexical scope leads to poorer performance.
`eval`, when not in strict mode, is one way to cheat.
Whoa, `setTimeout(..)` and `setInterval(..)` can technically take a string and dynamically execute it, that's wild (and super-deprecated).
TIL there's a function constructor, `new Function(..)` that lets you dynamically generate functions.

`with(obj)` will not create a property on an object, it'll just use that object as the innermost scope. But a `var` declaration will still be scoped to the containing function's scope. So in non-strict mode, saying `a=2` if `obj` does not define `a`, and nothing in the outer scopes does, will end up with you declaring a global variable `a`, equal to 2.

Using them _even once_ will globally affect your performance because the JS engine can't make performance optimizations around getting/setting things in scope.

# Chapter 3 : Function vs Block Scope

You can hide variables and functions by putting them within the scope of another function.

Libraries will typically expose an object that has all the functions and property accesses on that object to prevent namespace collisions.

You can create a function expression like:
```javascript
var a = 2;

(function foo(){
  var a=3;
  console.log(a); // 3
})(); // this executes it

console.log(a); // 2
```

If `function` is the very first thing in the line, it's a declaration, otherwise it's an expression.

`foo` in the function expression does not pollute the global scope - it hides its name within itself (so you could still recursively call itself from within itself).

function expressions can be anonymous - just omit the name. Frequently used in callbacks like:
```javascript
setTimeout(function() {
  console.log("I waited 1 second!");
}, 1000);
```

Note they have no name in stack traces, it cannot easily reference itself, and you lose a bit of self-documentation frequently. So, always name them!

IIFE => Immediately Invoked Function Expression

You can take a parameter and give an argument with IIFEs.

Block-scoping inreases the possible scope bubbles to include any sort of wrapped indentation, like a conditional or for loop. JS does not support that with `var`.

One of the few examples of block-scoping is within the `catch` block of a try-catch.

`let` gives you another way of block-scoping stuff. `let` attaches the variable to whatever block it's been defined within, where a block is wrapped in `{..}`.

You can create an arbitrary block for `let` to bind to by including a `{..}` pair anywhere a statement is valid.
`let` variables are not _hoisted_ to the top of their block - interesting!

Declaring explicit blocks also makes it easier for garbage collection to do its thing:
```javascript
function process(data) {
  // do something interesting
}

// anything declared inside this block can go away after!
{
  let someReallyBigData = { .. };

  process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase=*/false );
```

Our `click` event listener no longer has a closure over everything including `someReallyBigData`, so the GC can collect that big data after the block has run.

`let` loops actually re-bind with each and every for loop, so this:
```javascript
for (let i=0; i<10; i++) {
  console.log( i );
}
```
is really, under the hood, this:
```javascript
{
  let j;
  for (j=0; j<10; j++) {
    let i = j; // re-bound for each iteration!
    console.log( i );
  }
}
```

`const` also creates a block-scoped variable.
_dynamic scope_ cares where a function was _called from_, not where it was declared.
`this` cares _how a function was called_.

Whoa, ES6 to ES5 transpilation occurs via stuff like using the fact that `try`/`catch` allows for block-scoped variables.

# Chapter 4: Hoisting

Declarations, like `var a;`, are processed during the compilation phase. The assignment is left in place for the execution phase.
This includes `function` declarations, but **not** expressions!
Hoisting is **per-scope**.

Re expressions not being hoisted:
```javascript
foo(); // this TypeErrors

var foo = function bar() {
  // ...
};
```

Functions are hoisted first, and then variables. So if a function and variable share the same name, the variable will be ignored (because it would have already been defined).

But subsequent function declarations _do_ override previous ones. Basically, don't duplicate definitions within the same scope - you're in for a world of pain.

# Chapter 5: Scope Closure

**Closure** is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

You say function x has a closure over function y when function x includes in its outer lexical scope function y's scope. Or you can say x closes over the scope of y, because x appears nested inside of y.

_closure_ - having a reference to a scope (only meaningful outside of that declared scope). The garbage collector won't collect the inner scope of a `foo` if you still have access to the function `bar`, declared inside of `foo`, that accesses something in `foo`.

However you transport an inner function (multiple assignments/layers of indirection, etc.), it will always have access to that lexical scope.

When you pass a function to `setTimeout` that has some variable inside it referencing an outer scope, that's a closure.
Basically, any time you're dealing with a callback function.

You're observing closure when something executes outside of its lexical scope but still has access to that lexical scope.

```javascript
for (var i=1; i<=5; i++) {
  setTimeout( function timer(){
    console.log( i );
  }, i*1000 );
}
```

This will print out 6 every time! Because while you have a closure on the outer scope, the reference you have is to `i` in that outer scope, which changes at each step of the loop. All 5 `function timer()` declarations are closed over the same shared global scope, which has only the one `i` in it.

So, to have it work as expected, you need to put something in a scope unique to each call to `setTimeout`, that saves the value we care about. For instance, this will do the trick:

```javascript
for (var i=1; i<=5; i++) {
  (function(){
    var j = i;
    setTimeout( function timer(){
      console.log( j );
    }, j*1000 );
  })();
}
```

Or you can do that more elegantly in today's land by making use of block-scoping via `let`:
```javascript
for (var i=1; i<=5; i++) {
  let j = i; // yay, block-scope for closure!
  setTimeout( function timer(){
    console.log( j );
  }, j*1000 );
}
```
Each `j` is local to that particular block, and thus you get your values 1 through 5.
And `let` has special semantics in for loops!
```javascript
for (let i=1; i<=5; i++) {
  setTimeout( function timer(){
    console.log( i );
  }, i*1000 );
}
```
That also works.

You can use closures to create modules that only expose what you want to, despite the functions having access to their private variables described in the original outer function's scope that it closes over.

A _module_ is a function that returns an object containing reference to at least one function, where that function has a closure over the rest of the module.

If you refer to the object you're returning from within the module, you can change that object from within the module. Pretty neat!

ES6's Module APIs are fully static, unlike the function-based modules we're talking about here, which are dynamic (the module can modify itself after all). So the compiler can check for the existence of the method you're calling at compile time, which is pretty great.

Closures are when a function can remember and access its lexical scope even when called from outside its lexical scope. - figured it would be worth repeating that again...

`this` changes its binding, so if you're relying on the scope referring to the lexical scope at author-time, you need to operate on your original `this` by doing something like `var self = this;` and then use `self.foo`.

```javascript
var obj = {
  count: 0,
  cool: function coolFn() {
    var self = this;

    if (self.count < 1) {
      setTimeout( function timer(){
        self.count++;
        console.log( "awesome?" );
      }, 100 );
    }
  }
};

obj.cool(); // awesome?
```

=> functions have a "lexical this". They use the `this` value of their immediate lexical enclosing scope, no matter what. You could get the same thing by attaching a `.bind(this)` to the end of the function, to keep `this` attached to its lexical scope.

Boom! And that's all (s)he wrote about scope and lexical closures. :D