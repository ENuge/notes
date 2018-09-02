# You Don't Know JS: Async & Performance

# Chapter 1: Asynchrony: Now & Later

`console.log(..)` can occur asynchronously, if the browser engine decides the IO will be expensive and should be deferred - in which case the value it logs may be something that occurred later in a top-to-bottom reading of your code. If you think that's happening, you can use `debugger;` or force a "snapshot" by stringifying the object with something like `JSON.stringify(..)`.

Until recently, the asynchrony bits were handled entirely by the engine. JS had nothing built-in to do anything other than execute a chunk of your program when asked to.

The JS engine runs inside a _hosting environment_ - like your browser. The "event loop" in that _hosting environment_ handles executing multiple chunks of your program _over time_.

When you call `setTimeout(..)` it sets up a timer, when the timer expires, the environment places your callback into the event loop, so some future tick picks it up and executes it. If there's a bunch of stuff in the event loop, it waits - which explains why your thing won't exactly run when the timer ends.

_async_ is about the gap between _now_ and _later_. Parallel is about things occurring _simultaneously_.

Multiple threads can share the memory of a single process. JS runs in a single thread, so we don't have to worry about parallelism. Because of that single-threading, code in a given function will run-to-completion before another function runs at all:

```javascript
var a = 1;
var b = 2;

function foo() {
  a++;
  b = b * a;
  a = b + 3;
}

function bar() {
  b--;
  a = 8 + b;
  b = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo` or `bar` will run in its entirety first, depending on which ajax request gets back to us first. Which means there's still nondeterminism if they modify the same values. So you can have a race condition between `foo` and `bar`.

Concurrency is when two or more processes execute simultanouesly over the same period, regardless of whether they happen _in parallel_.

Nondeterminism is totally OK if the different processes don't interact at all. But if they do interact, you must make sure your callback coordinates properly. For instance, if the order in `res` matters, you could do something like:
```javascript
var res = [];

function response(data) {
  if (data.url == "http://some.url.1") {
    res[0] = data;
  }
  else if (data.url == "http://some.url.2") {
    res[1] = data;
  }
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

Another form of this uses a "gate" to make sure we have all the data we need:
```javascript
var a, b;

function foo(x) {
  a = x * 2;
  if (a && b) {
    baz();
  }
}

function bar(y) {
  b = y * 2;
  if (a && b) {
    baz();
  }
}

function baz() {
  console.log( a + b );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

You can also use a "latch" so only the first call through is allowed to call the function:
```javascript
var a;

function foo(x) {
  if (a == undefined) {
    a = x * 2;
    baz();
  }
}

function bar(x) {
  if (a == undefined) {
    a = x / 2;
    baz();
  }
}

function baz() {
  console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

You can be cooperatively concurrent by, if you know you have a huge amount of work, yielding back to the event loop so other things can occur in the meantime. Without using the `yield` keyword specifically, you could do something like:

```javascript
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
  // let's just do 1000 at a time
  var chunk = data.splice( 0, 1000 );

  // add onto existing `res` array
  res = res.concat(
    // make a new transformed array with all `chunk` values doubled
    chunk.map( function(val){
      return val * 2;
    } )
  );

  // anything left to process?
  if (data.length > 0) {
    // async schedule next batch
    setTimeout( function(){
      response( data );
    }, 0 );
  }
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

Note that `setTimeout(..0)` does not guarantee events will be put directly on the event loop - it's when the timer has its next opportunity to do so.

The Job queue basically inserts at the beginning of the event loop a Job from the event that happened in the last event loop tick. Jobs are like the `setTimeout(..0)` hack intended - operate **later, but as soon as possible**. Jobs happen at the end of the current event loop tick, so before other things are processed in the event loop.

# Chapter 2: Callbacks

The callback function encapsulates the _continuation_ of the program.

Inversion of control - when you use a callback with a function you didn't write, like a third party library, you entrust the execution of that function to that third party library. Protecting against everything that could go wrong would require a gigantic amount of code, _each and every time_.

Node tends to do error-object style callback handling, where the first argument is an `error` object if there is any error - and you just handle that in the one callback:
```javascript
function response(err,data) {
  // error?
  if (err) {
    console.error( err );
  }
  // otherwise, assume success
  else {
    console.log( data );
  }
}

ajax( "http://some.url.1", response );
```

Always invoke callbacks asynchronously, so they're all predictably async. Otherwise, you can introduce confusing bugs into your code (releasing Zalgo, as it were).

# Chapter 3: Promises

**Promises** resolve the inversion of control problems - it gives us the ability to know when another function has finished, and then we can run our code within our own context.

"Promises are one of those tools where it can be painfully obvious from how someone uses it whether they understand what it's for and about versus just learning and using the API." And this is where this book is so, tremendously, useful.

A **Promise** returns a future value, that can then be exchanged for the value itself when it is ready (or, when the future value has _resolved_). `.then(..)` itself returns a `Promise`, which the `return` inside of it immediately resolves.

A **Promise** can be fulfilled or rejected (called a "rejection reason"). The `.then(..)` call can take two functions, one for fulfillment, one for rejection:
```javascript
add( fetchX(), fetchY() )
.then(
  // fullfillment handler
  function(sum) {
    console.log( sum );
  },
  // rejection handler
  function(err) {
    console.error( err ); // bummer!
  }
);
```

Promises encapsulate the time-dependent state, so from the outside you can reason about different Promises (and chains of Promises) time-independently. Once a Promise is resolved, it stays that way forever - it becomes immutable. So multiple parties can act on a resolution of a Promise, knowing that none of the others can in any way modify that Promise.

An individual Promise can also behave as a flow-control mechanism - if you need this-then-that behavior, the interface is very natural.

You also get a nice _separation of concerns_ - where the asynchronous function itself does not need to know who cares about its completion (or who will be continuing after it).

Here's one sample implementation of an event listening capability (a prelude to how Promises would handle this...):
```javascript
function foo(x) {
  // start doing something that could take a while

  // make a `listener` event notification
  // capability to return

  return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){
  // now we can do the next step!
} );

evt.on( "failure", function(err){
  // oops, something went wrong in `foo(..)`
} );
```

That allows for a nicer _separation of concerns_, and things like multiple functions to listen on that returned listener.

Here's the "revealing constructor", which still seems a bit opaque to me - I'm hoping continuing to read more of this chapter will illuminate that (EDIT ON REREAD: Ah, it makes more sense now - just move the "Start doing something that could take a while" comment into the Promise itself, and it becomes clear. And, I'm pretty sure that long-running code *needs* to be inside the revealing constructor, otherwise the first slow bit will operate synchronously):
```javascript
function foo(x) {
  // start doing something that could take a while

  // construct and return a promise
  return new Promise( function(resolve,reject){
    // eventually, call `resolve(..)` or `reject(..)`,
    // which are the resolution callbacks for
    // the promise.
  } );
}

var p = foo( 42 );

bar( p );

baz( p );
```

Here are two different ways you can listen and perform a deferred action on a promise:
```javascript
function bar(fooPromise) {
  // listen for `foo(..)` to complete
  fooPromise.then(
    function(){
      // `foo(..)` has now finished, so
      // do `bar(..)`'s task
    },
    function(){
      // oops, something went wrong in `foo(..)`
    }
  );
}

// ditto for `baz(..)`
```
Above passes the promise directly into a function, and the function handles both `.then` cases. You can also directly attach the functions to the `.then(..)` like so:
```javascript
function bar(fooPromise) {
  // listen for `foo(..)` to complete
  fooPromise.then(
    function(){
      // `foo(..)` has now finished, so
      // do `bar(..)`'s task
    },
    function(){
      // oops, something went wrong in `foo(..)`
    }
  );
}

// ditto for `baz(..)`function bar() {
  // `foo(..)` has definitely finished, so
  // do `bar(..)`'s task
}

function oopsBar() {
  // oops, something went wrong in `foo(..)`,
  // so `bar(..)` didn't run
}

// ditto for `baz()` and `oopsBaz()`

var p = foo( 42 );

p.then( bar, oopsBar );

p.then( baz, oopsBaz );
```

Promises retain their same resolution forever, which is why we could do the multiple `.then(..)` calls in the above example.

A "thenable" is any function or object which has a `then(..)` on it, in which case you use duck typing to assume it's actually Promise-like. So, any existing library has to update itself or become incompatible with Promises, and anything accidentally adding a `.then(..)` can create hard-to-track bugs in your code. Nice!

Even an immediately-resolved `Promise` cannot be _observed_ synchronously: `new Promise(function(resolve) { resolve(42);})` is still put on the job queue. The callback you provide to `then(..)` will **always** be called asynchronously on the Job queue.

Don't rely on the ordering/scheduling of callbacks across Promises. Don't code where the ordering of multiple callbacks matters at all.

Nothing can prevent a Promise from notifying you of its resolution (if it's resolved).

You can use `Promise.race(..)` to set multiple promises to race against each other, using the first one that resolves (or rejects!):
```javascript
// a utility for timing out a Promise
function timeoutPromise(delay) {
  return new Promise( function(resolve,reject){
    setTimeout( function(){
      reject( "Timeout!" );
    }, delay );
  } );
}

// setup a timeout for `foo()`
Promise.race( [
  foo(),					// attempt `foo()`
  timeoutPromise( 3000 )	// give it 3 seconds
] )
.then(
  function(){
    // `foo(..)` fulfilled in time!
  },
  function(err){
    // either `foo()` rejected, or it just
    // didn't finish in time, so inspect
    // `err` to know which
  }
);
```

Promises are defined so they can only be resolved once. It will accept only the first resolution and silently ignore all others. If you don't call `resolve` or `reject` with any arguments, you'll just pass `undefined` to them. But if you pass it with multiple parameters, all but the first will be silently ignored!! You still have access to the closure in which the callback is defined, though.

The `rejected` callback will be called with any error on things like `TypeError`s. That makes JS exceptions effectively asynchronous, avoiding another Zalgo moment. Each chained `.then(..)` implicitly returns another Promise. If you don't call `resolve` or `reject`, subsequent `.then()`s will just never happen.

A value returned by a `.then()` handler is immediately passed to the next handler, unless you explicitly make it async by constructing your own `new Promise(..)`.

Instead of the two-function `.then(..)` that handles both success and failure, you can have separate `.then(..)` and `.catch(..)` that each just takes a single `function`. This can be nice because you can chain multiple `.then(..)`s and then add a `.catch(..)` at the end of it all.

You don't actually pass your callback to the function that returns the `Promise`, you pass your callback to the Promise itself which is returned by that function.

You can always wrap the function call in `Promise.resolve(func(..))` which will unwrap the function to its non-thenable value, and make a promise out of it, OR, if it's already a `Promise`, just return that `Promise`. So with that, you can trust any `thenable`, and also make sure it runs asynchronously (keeping Zalgo at its bay). So you can protect against the malicious behavior of this `Promise`, for instance:
```javascript
var p = {
  then: function(cb,errcb) {
    cb( 42 );
    errcb( "evil laugh" );
  }
};

p
.then(
  function fulfilled(val){
    console.log( val ); // 42
  },
  function rejected(err){
    // oops, shouldn't have run
    console.log( err ); // evil laugh
  }
);
```

Every time you call `.then(..)` on a Promise, it returns a new Promise that you can chain with. You can also chain multiple async steps:
```javascript
var p = Promise.resolve( 21 );

p.then( function(v){
  console.log( v );	// 21

  // create a promise to return
  return new Promise( function(resolve,reject){
    // introduce asynchrony!
    setTimeout( function(){
      // fulfill with value `42`
      resolve( v * 2 );
    }, 100 );
  } );
} )
.then( function(v){
  // runs after the 100ms delay in the previous step
  console.log( v );	// 42
} );
```

Wrapping some of this in a nice bow, here's a delay-Promise creation series of steps:
```javascript
function delay(time) {
  return new Promise( function(resolve,reject){
    setTimeout( resolve, time );
  } );
}

delay( 100 ) // step 1
.then( function STEP2(){
  console.log( "step 2 (after 100ms)" );
  return delay( 200 );
} )
.then( function STEP3(){
  console.log( "step 3 (after another 200ms)" );
} )
.then( function STEP4(){
  console.log( "step 4 (next Job)" );
  return delay( 50 );
} )
.then( function STEP5(){
  console.log( "step 5 (after another 50ms)" );
} )
...
```

Or a Promise-aware ajax request:
```javascript
// Promise-aware ajax
function request(url) {
  return new Promise( function(resolve,reject){
    // the `ajax(..)` callback should be our
    // promise's `resolve(..)` function
    ajax( url, resolve );
  } );
}
```

An intermediate rejection handler that just returns a value will "reset" the Promise chain back on the resolve chain:
```javascript
// step 1:
request( "http://some.url.1/" )

// step 2:
.then( function(response1){
  foo.bar(); // undefined, error!

  // never gets here
  return request( "http://some.url.2/?v=" + response1 );
} )

// step 3:
.then(
  function fulfilled(response2){
    // never gets here
  },
  // rejection handler to catch the error
  function rejected(err){
    console.log( err );	// `TypeError` from `foo.bar()` error
    return 42;
  }
)

// step 4:
.then( function(msg){
  console.log( msg );		// 42
} );
```

Step 4 still executes after `rejected(..)` in step 3 runs. If you don't throw an error handler, an assumed error handler is run that just re-throws the error it created. That is why the `.catch` chained bit way above works as expected. If you specify `null` as the `resolve` step, it'll just passthrough the value. (Which is the same as the `.catch(..)` bit mentioned earlier, again!).

Always use `reject(..)` as the second function to the `Promise`, because that is _always_ what it refers to. 

`Promise.resolve(..)` can result in a fulfilled or rejected state, so `resolve` as opposed to _fulfilled_ is a good choice of names:
```javascript
var rejectedTh = {
  then: function(resolved,rejected) {
    rejected( "Oops" );
  }
};

var rejectedPr = Promise.resolve( rejectedTh );
```

Because `resolve` will unwrap underlying Promises (and thus keep it `rejected` if the underlying Promise was `rejected`), the first callback function being called `resolved` makes more sense. (I'm not sure if this makes sense to me...->please return to this):
```javascript
var rejectedPr = new Promise( function(resolve,reject){
  // resolve this promise with a rejected promise
  resolve( Promise.reject( "Oops" ) );
} );

rejectedPr.then(
  function fulfilled(){
    // never gets here
  },
  function rejected(err){
    console.log( err );	// "Oops"
  }
);
```

`Promise.reject(..)` does not do the same sort of unwrapping that `Promise.resolve(..)` does. Ah!! You can `resolve` while still ending up in the `rejected` state. In your `.then(..)`, you only get into the first state if the previous Promise was fulfilled, and thus it makes more sense for that value to be called `fulfilled`, _not_ `resolved`. That makes a fair bit of sense to me!!

It's very easy to have errors swallowed in Promise-land if you expect something like this to get the error caught in its error handler:
```javascript
var p = Promise.resolve( 42 );

p.then(
  function fulfilled(msg){
    // numbers don't have string functions,
    // so will throw an error
    console.log( msg.toLowerCase() );
  },
  function rejected(err){
    // never gets here
  }
);
```

As explained a bit above, this would require chaining another `.then(..)` that handles the error, because the existing `.then(..)` only handles errors when the original Promise resolves. But even if you do that, any error in the final `.catch(..)` won't get caught by anybody and will just silently fail.

The author's proposed solution would be to make any errors in the final `.then(..)` raise and log in the console etc. unless you put a `.defer()` in the promise, in which case it wouldn't. I personally like the idea of a `.finally(..)` rejection handler.

`Promise.race([..])` and `Promise.all([..])` exist, I think I'll remember what they do. `Promise.race([..])` never resolves if you give it an empty array! There are also variations like `.none([..])`, `.first([..])`, `.last([..])`, `.any([..])` that work as you might expect.

This is the revealing constructor for Promises, because I guess it reveals how the Promise itself was created?
```javascript
var p = new Promise( function(resolve,reject){
  // `resolve(..)` to resolve/fulfill the promise
  // `reject(..)` to reject the promise
} );
```

```javascript
var fulfilledTh = {
  then: function(cb) { cb( 42 ); }
};
var rejectedTh = {
  then: function(cb,errCb) {
    errCb( "Oops" );
  }
};

var p1 = Promise.resolve( fulfilledTh );
var p2 = Promise.resolve( rejectedTh );

// `p1` will be a fulfilled promise
// `p2` will be a rejected promise
```

`then` and `catch` syntax:
```javascript
p.then( fulfilled );

p.then( fulfilled, rejected );

p.catch( rejected ); // or `p.then( null, rejected )`
```

This `p` will refer to the final Promise in the chain:
```javascript
var p = foo( 42 )
.then( STEP2 )
.then( STEP3 );
```

Interim `then(..)`s can catch and swallow errors, just like interim `try..catch`es.

A better way of composing multiple things within a single Promise:
```javascript
function foo(bar,baz) {
  var x = bar * baz;

  // return both promises
  return [
    Promise.resolve( x ),
    getY( x )
  ];
}

Promise.all(
  foo( 10, 20 )
)
.then( function(msgs){
  var x = msgs[0];
  var y = msgs[1];

  console.log( x, y );
} );
```

Worth reiterating - Promises can only ever be resolved once. This means you need to do gymnastics to use them with things like event handlers (instead, you'd have to define the entire Promise chain within the event handler callback itself).

If you ever needed a good example of how ugly JS code can be, consider this block, which adds a `Promise.wrap(..)` helper function:
```javascript
// polyfill-safe guard check
if (!Promise.wrap) {
  Promise.wrap = function(fn) {
    return function() {
      var args = [].slice.call( arguments );

      return new Promise( function(resolve,reject){
        fn.apply(
          null,
          args.concat( function(err,v){
            if (err) {
              reject( err );
            }
            else {
              resolve( v );
            }
          } )
        );
      } );
    };
  };
}
```

That lets you promisify the callback, so instead you get a function that returns a Promise like so:
```javascript
var request = Promise.wrap( ajax ); // a function that returns a Promise!

request( "http://some.url.1/" )
.then( .. )
..
```

Our author likes calling it a promisory, like a promisory note. I'm OK with that.

Promises are uncancelable by design - if one person were to cancel a Promise, it would affect all the other thenables chained on it, which is effectively like another case of callback hell!

The author does distinguish between canceling a single Promise and canceling a sequence of chained Promises, because the latter is _not_ a single immutable value. But it does eventually resolve to a single immutable value, just like an individual Promise would, so I'm not sure that I follow. But I need to check out the actual proposed _asynquence_ library first!

Promises don't get rid of callbacks, they just redirect how those callbacks are composed to a trustable intermediary mechanism that sits between us and another utility.

# Chapter 4: Generators

**generators** enable us to express async flow in a sequential, synchronous-looking fashion.

**preemptive** multitasking would mean another function or executing thing could preempt the current one and run in the middle of its process, making any changes to shared state.
**cooperative** multitasking requires the initial process to explicitly yield its control to the other so it can run.

And here's such a generator in action:
```javascript
var x = 1;

function *foo() {
  x++;
  yield; // pause!
  console.log( "x:", x );
}

function bar() {
  x++;
}
```

The *foo is purely syntactic, it could also be function* foo in which case you would call foo and not *foo.

And then you use an iterator to control the execution of the generator:
```javascript

// construct an iterator `it` to control the generator
var it = foo(); // doesn't actually execute the generator, just creates the iterator.

// start `foo()` here!
it.next();
x;						// 2
bar();
x;						// 3
it.next();				// x: 3
```

Generators still take inputs and can still return outputs:
```javascript
function *foo(x,y) {
  return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;		// 42
```

You can pass the generator arguments with each call to `.next()`:
```javascript
function *foo(x) {
  var y = x * (yield); // returns at yield, mid-expression, and pauses until next() is called with an argument to be what yield is here
  return y;
}

var it = foo( 6 );

// start `foo(..)`
it.next();

var res = it.next( 7 );

res.value;		// 42
```

Note there's one more `.next()` call than `.yield()`, because you need to start the thing!

And `yield` can also pass things in the other direction:
```javascript
function *foo(x) {
  var y = x * (yield "Hello");	// <-- yield a value!
  return y;
}

var it = foo( 6 );

var res = it.next();	// first `next()`, don't pass anything
res.value;				// "Hello"

res = it.next( 7 );		// pass `7` to waiting `yield`
res.value;				// 42
```

I guess you could use async generators to get a sort of message passing / actor model going in JS.

You can have multiple iterators to the same generator function, which will iterate and keep state separately:
```javascript
function *foo() {
  var x = yield 2;
  z++;
  var y = yield (x * z);
  console.log( x, y, z );
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;			// 2 <-- yield 2
var val2 = it2.next().value;			// 2 <-- yield 2

val1 = it1.next( val2 * 10 ).value;		// 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;		// 600 <-- x:200, z:3

it1.next( val2 / 2 );					// y:300
                    // 20 300 3
it2.next( val1 / 4 );	
```

You can interleave between generators (and produce confusing contrived code in the process!). Depending on how you iterate through these, you will observe different values:
```javascript
var a = 1;
var b = 2;

function *foo() {
  a++;
  yield;
  b = b * a;
  a = (yield b) + 3;
}

function *bar() {
  b--;
  yield;
  a = (yield 8) + b;
  b = a * (yield 2);
}
```

The generator itself is not technically an _iterable_ - when you execute the generator, you get an _iterator_ back:
```javascript
function *foo(){ .. }

var it = foo();
```

A generator's _iterator_ is also an _iterable_, it just uses the iterator itself for the iterating.

A `break`, `return`, or uncaught exception in the `for..of` loop sends a signal to the generator's _iterator_ for it to terminate.

A `try..finally` clause inside a generator will always run even when the generator is externally completed, which is handy for cleanup.

You can manually terminate a generator by calling `.return(..)` on its iterator:
```javascript
var it = something();
for (var v of it) {
  console.log( v );

  // don't let the loop run forever!
  if (v > 500) {
    console.log(
      // complete the generator's iterator
      it.return( "Hello World" ).value
    );
    // no `break` needed here
  }
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

A standard JS iterator can automatically be consumed in a `for..of` loop:
```javascript
for (var v of something) {
  console.log( v );

  // don't let the loop run forever!
  if (v > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

Here's the code used to create that iterator, using function closures. Obviously, the ES6 alternative is a nicer syntax:
```javascript
var something = (function(){
  var nextVal;

  return {
    // needed for `for..of` loops
    [Symbol.iterator]: function(){ return this; },

    // standard iterator interface method
    next: function(){
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }

      return { done:false, value:nextVal };
    }
  };
})();

something.next().value;		// 1
something.next().value;		// 9
something.next().value;		// 33
something.next().value;		// 105
```

`yield` allows you to have code that appears to be blocking and synchronous, but doesn't actually block the whole program. The reliance on `it` here is super janky, but nonetheless, I present unto thee:
```javascript
function foo(x,y) {
  ajax(
    "http://some.url.1/?x=" + x + "&y=" + y,
    function(err,data){
      if (err) {
        // throw an error into `*main()`
        it.throw( err );
      }
      else {
        // resume `*main()` with received `data`
        it.next( data );
      }
    }
  );
}

function *main() {
  try {
    var text = yield foo( 11, 31 );
    console.log( text );
  }
  catch (err) {
    console.error( err );
  }
}

var it = main();

// start it all up!
it.next();
```

Note that allows `*main` to have code that looks and operates perfectly synchronously, which is dope. Including with synchronous error handling.

Note in this example how the `yield` sends and receives messages (the receive bit is very implicit imo):
```javascript
function *main() {
  var x = yield "Hello World";

  yield x.toLowerCase();	// cause an exception!
}

var it = main();

it.next().value;			// Hello World

try {
  it.next( 42 );
}
catch (err) {
  console.error( err );	// TypeError
}
```

You could make your iterator yield a Promise, and then have that returned Promise's `then`'ing control the next level of the iteration - so the iterator function itself remains synchronous-looking:
```javascript
function foo(x,y) {
  return request(
    "http://some.url.1/?x=" + x + "&y=" + y
  );
}

function *main() {
  try {
    var text = yield foo( 11, 31 );
    console.log( text );
  }
  catch (err) {
    console.error( err );
  }
}

var it = main();

var p = it.next().value;

// wait for the `p` promise to resolve
p.then(
  function(text){
    it.next( text );
  },
  function(err){
    it.throw( err );
  }
);
```

All of this can be made simpler if JS adopts an `async`/`await` pattern:
```javascript
function foo(x,y) {
  return request(
    "http://some.url.1/?x=" + x + "&y=" + y
  );
}

async function main() {
  try {
    var text = await foo( 11, 31 );
    console.log( text );
  }
  catch (err) {
    console.error( err );
  }
}

main();
```

Such an `async` function would automatically return a Promise. Which is something we have now!! You can then use Promises on top of this to make async things still happen in async: 
```javascript
function *foo() {
  // make both requests "in parallel"
  var p1 = request( "http://some.url.1" );
  var p2 = request( "http://some.url.2" );

  // wait until both promises resolve
  var r1 = yield p1;
  var r2 = yield p2;

  var r3 = yield request(
    "http://some.url.3/?v=" + r1 + "," + r2
  );

  console.log( r3 );
}

// use previously defined `run(..)` utility
run( foo );
```

Author suggests trying to hide the Promise logic outside of the generator, so your sync-looking code is nice and clean.

You can have a generator that yields the result of another generator:
```javascript
function *foo() {
  var r2 = yield request( "http://some.url.2" );
  var r3 = yield request( "http://some.url.3/?v=" + r2 );

  return r3;
}

function *bar() {
  var r1 = yield request( "http://some.url.1" );

  // "delegating" to `*foo()` via `run(..)`
  var r3 = yield run( foo );

  console.log( r3 );
}

run( bar );
```

Or you can rely on `yield`-delegation, where if you call it with `yield *foo` it'll yield when the called generator yields. Fancy!
```javascript
function *foo() {
  console.log( "`*foo()` starting" );
  yield 3;
  yield 4;
  console.log( "`*foo()` finished" );
}

function *bar() {
  yield 1;
  yield 2;
  yield *foo();	// `yield`-delegation!
  yield 5;
}

var it = bar();

it.next().value;	// 1
it.next().value;	// 2
it.next().value;	// `*foo()` starting
          // 3
it.next().value;	// 4
it.next().value;	// `*foo()` finished
          // 5
```

You can `yield`-delegate to any iterable, `yield *[1,2,3]` would `yield`-delegate to the default array iterator. Nice! Non-generator iterators will ignore the value you pass to it when calling `next`, which is worth noting. Errors and exceptions also pass in both directions.

You could use `yield`-generation to delegate a function to itself, aka to recurse:
```javascript
function *foo(val) {
  if (val > 1) {
    // generator recursion
    val = yield *foo( val - 1 );
  }

  return yield request( "http://some.url/?v=" + val );
}

function *bar() {
  var r1 = yield *foo( 3 );
  console.log( r1 );
}

run( bar );
```

A _thunk_ is a param-less function that already passes the arguments and just returns another function, allowing you to defer execution of that second function:
```javascript
function foo(x,y) {
  return x + y;
}

function fooThunk() {
  return foo( 3, 4 );
}

// later

console.log( fooThunk() );	// 7
```

Here's a typical `thunkify` method, which produces a function that produces a thunk:
```javascript
var whatIsThis = thunkify( foo );

var fooThunk = whatIsThis( 3, 4 );

// later
fooThunk( function(sum) {
  console.log( sum );		// 7
} );
```

# Chapter 5: Program Performance

JavaScript has no features to support threaded execution - Web Workers all happen browser-side. Each environment provides its own instance of the JS engine that tasks run on. You can instantiate a worker by just pointing it at the thread you wish to run:
```javascript
var w1 = new Worker( "http://some.url.1/mycoolworker.js" );
```

You can listen and send messages to workers as follows:
```javascript
// "mycoolworker.js"

addEventListener( "message", function(evt){
  // evt.data
} );

postMessage( "a really cool reply" );
```
It's the exact same in the code that created the web worker. Workers can instantiate subworkers. The worker thread cannot access the DOM or _any_ global variables created in the main program. But you can perform network operations and set timers. You can sync load other scripts into the worker:
```javascript
// inside the Worker
importScripts( "foo.js", "bar.js" );
```

You can create a `SharedWorker` between multiple open tabs.

SIMD stands for Single Instruction, Multiple Data. TIL! It's data parallelism - do multiple things on data at once. CPUs provide the SIMD capability.

asm.js - a highly optimizable subset of the JS language. Basically, you limit yourself to a subset of JS features and provide some hints (operations that force a thing to be an integer, for instance) so the compiler can then more easily optimize your code. You create and manage your own heap, for instance, to avoid the penalties of garbage collection. It's not intended to be a general compile target, but rather for specific things like mathematically intensive work. It's meant to be like assembly language, hence the name.

# Chapter 6: Benchmarking & Tuning

```javascript
var start = (new Date()).getTime();	// or `Date.now()`

// do some operation

var end = (new Date()).getTime();

console.log( "Duration:", (end - start) );
```

The above is not the best way to benchmark your performance! It has at best ms precision, and `15ms` precision on old versions of IE.

How long you run your benchmark should be measured not by how many times you might expect to iterate over something (or some multiple of that), but rather by basing it on how inaccurate your timer is. The more inaccurate, the longer you should run it to minimize the inaccuracy. A 15ms timer requires 750ms to get an error rate less than 1%, a 1 ms timer requires just 50 ms. (Shouldn't those ms numbers be doubled?? Or where did they come from? 🤔)

You'll want to think of the spread of results if you run that same code multiple times, the difference between fastest and slowest times, any skew, etc.

"Just use benchmark.js". It allows you to easily compare X to Y head-to-head. Using stuff like that to automate performance regression tests and show the performance over time in a controlled environment is pretty neat.

Don't overfocus on microoptimizations that only matter if you do things ten million times, because you're very infrequently doing things ten million times. The engine may also handle your benchmarked code in isolation extremely differently than it would handle it in an actual production environment.

Unrolling recursion - a compiler optimization that turns it into a loop.

Don't try to outsmart your JS engine, you'll probably lose when it comes to performance optimizations.

"paving the cowpath" - standardizing or optimizing some particular approach based on its existing wide mainstream usage.

_tail-call optimization_ - reusing the existing stack frame if all you are doing is returning the recursed function. Allows tail-call-optimized functions to recurse indefinitely without hitting a max stack frame. ES6 guarantees that you can rely on this - it's fundamental to some of its own language constructs. Interesting!

I basically skipped over the asynquence library, knowing full well Stripe does not use it (or, I haven't seen it in use).

TODO: Read up on goroutines and all this CSP stuff in a more thorough way.
TODO2: Read this: https://gist.github.com/staltz/868e7e9bc2a7b8c1f754 . Bills itself as an easy-to-read intro to reactive programming.