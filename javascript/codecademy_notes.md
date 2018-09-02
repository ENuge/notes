# Codecademy JS Notes

**NOTE:** This was meant to be a quick re-intro to JS syntax, plus an intro to ES6 stuff. The YDKJS notes are _far_ more complete, and also that book is quite worth the time investment. Go read it.

Can use ref forwarding to pass refs on from a higher-order component 
through to the actual underlying component that wants it.

Render prop - share code between components using a prop whose value is a function.
Just pass the function (that returns a component!) to the render prop, and call that function which will return its component.

# JS NOTES

Declare const variables with const
`let` variables can be reassigned
You can declare a variable without assigning it - it'll just be undefined. 
`let` creates block scope local variables, var global or function-scoped.
`let` does not create a property on the global object.
Can't redeclare `let` variables in the same function or block scope.
Only one block in switch statements.
let variables are not initialized until their definition evaluated - vars are actually hoisted to the top of the scope.

Arrow function - shorter syntax than function expression, no this, args, super, or new.target.
They replace this:
```javascript
function funcName(params) {
	return params + 2;
}
// with:
var funcName = (params) => params + 2;

(parameters) => {statements}
```

Parentheses optional if only one parenthesis. Same for statements if it is only a single expression.
`this` is bound lexically - ie it keeps its meaning from its original context. 
Regular functions bind `this` - so for those, `this` is local to that function.
Can always test what this you're bound to by `console.log`'ing it.

# THIS IN JS
`this` may be different each time a function is called. Especially for arrow functions.
Every variable you declare in the global scope is attached to the window object.

If `this` is not inside of a declared object, it defaults to the global (`window`) object.
`this` inside a declared object sets it to the closest parent object the method is called on.
This is bound to the new object being created when you use the new keyword. So you can use that to bound functions to not globals:

```javascript
function Car(make, model) {
	this.make = make;
	this.model = model;
}
var myCar = new Car('Ford', 'Escape');
console.log(myCar);
// logs => Car {make: "Ford", model: "Escape"}
```

`call`/`bind`/`apply` can all explicitly set the value of this. Like `func.call`, `func.bind` etc.
```javascript
func.call(whatYouWantThisToBe, other_args)
func.apply(whatYouWantThisToBe, [other, args])
```
`bind()` is not invoked immediately, but same idea. It returns a function that already has this bound. And you can partially apply other args.

# BASIC JAVASCRIPT SYNTAX
Can `+=`, `*=`, `++`, `--`, etc. number variables.

Can use ES6-style string interpolation with backticks and then the variable, like: `my variable is ${variable}.`

In JS, all values have a truthy or falsy value.
The falsy values are:
`false`
`0`, `-0`, `""`, `''`, `null`, `undefined`, `NaN`

Use threequals, not `==`. And inequality, use `!==`.
Use `&&` and `||` for and and or.

switches are like:
```javascript
switch (variable) {
	case 'foo': 
		blah;
		break;
	...
	default:
		...break;
}
```

Without break, it'll execute all matching statements plus the default!

# FUNCTIONS

`const var () => { };` is arrow function syntax.
Any code between curly braces is called a block.
If you don't pass a required argument, JS will just run OK with undefined - which lets you do funky currying.

Function declaration - function bound to an identifier or name, like:
```javascript
function square (number) {
	return number * number;
}
```

Function expressions are often stored in variables, and they end in a semicolon. Arrow functions are a shorter syntax for function expressions.
Functions that take a single parameter generally should not use parentheses. Otherwise, the parentheses are required.
Can have an implicit return if the function is a single-line block. No return keyword needed!
A sole single-line block function does not need brackets! Nice :D.

So you could transform:
```javascript
const foo = (bar) => {
	return baz;
};
// into:
const foo = bar => baz;
```

# SCOPE
Anything inside a set of curly braces is a block!
Global scope - anything defined outside a set of curly braces.
If you don't declare your variable at a more local scope and a globally scoped one exists, it will use (and modify) that variable at the global scope.
If you define a variable at the block scope level, you'll get a ReferenceError if you try to refer to it outside of that block.

Javascript's hoisting - no matter where variables are declared in a particular scope, all variable declarations are moved to the top of their scope (but the assignment still happens at the assigned spot).

No error is thrown if a let declaration creates a variable with the same name as that of a variable in its outer scope. Same with const. But if a variable has already been defined in that scope with a var, declaring a same-named one with let will error.
A const object's properties can be mutated, however! But you can't redefine the entire object.

All of `const`/`let`/`var`/etc are hoisted to the top of the scope they're used in - which means that if a variable is declared in a scope, in that scope the identifier will always reference that particular variable. (not any outside scoped variables of the same name).
Everything in the temporal dead zone stays uninitialized
The temporal dead zone is purely time-based, not location-based, so you can declare code above it that *will* work as long as you don't execute it until after the declaration.

# ARRAYS
Can get the length of an array with `x.length`.
`.push()` let us add items (however many we want) to the end of an array. Also there's `.pop();`

# LOOPS
break breaks out of a switch or loop (but not a set of nested loops!)

# ITERATORS
`Array.forEach()` -> operates function on each element. Returns undefined. Does not modify original array.
`.filter()` -> returns the elements in the original array (in a new array) that evaluate to truthy in the condition.
`.some()` -> returns true if any of the elements evaluate to truthy in the condition
`.every()` -> like that but for every value

# OBJECTS
JS objects are NOT ordered!
You can define methods within an object easier now:
```javascript
const restaurant = {
	openRestaurant() {
		return 'Foo';
	}
}
```
Whoa! Note that will allow to access `this`, which the arrow syntax model *wont*, because of lexical scoping (it doesn't know what `this` is.)
Use `this` to access properties within the same object.
getters and setters are the preferred way (in mutable-JS land) to change object properties.
Use the _-prefix property thing like Python to denote (in an unenforced way) a private property.
Setter and getter methods are prefixed with the term `set` and `get`...for some reason. So you can call them like restaurant.seatingCapacity = 150; (assuming there is a setter with that name). Neato! The getter method is called as if it were just a property object, not a function.

JS calls the `constructor()` method every time it creates a new instance of a class. 
`this` within a class refers to an instance of that class.
You create an instance of a class with `new Dog('foo')`.
The new keyword calls the constructor.
Class method and getter syntax is the same as for objects.

You just say `Class extends OtherClass` in the definition.
The constructor of the subclass can take more arguments.
Call `super` on the first line of the constructor, otherwise you cannot access this within the constructor.
`static` methods are called on the class, as opposed to any particular instance of the class.
You can use `npm build` to handle your ES5 transpilation - it takes a key-value of bash commands.

# TRANSPILATION etc.
`npm install <PACKAGE> -D` , -D specifies that something is a dev dependency.

# MODULES
You could do `module.exports = {...}` and define the object in-line like that.
But in ES6-land, just do `export default foo;`
Now you can just import stuff like:
`import <THING> from <FILE_NO_EXTENSION>`

`export default` - if you're exporting one thing and don't care about its name.
Use named exports if you want the importing file to also use that name.
Named exports get imported in curly braces, default exports don't (and can be named in the importing file whatever you want - the braces ones require aliasing via {x as y}).
You can also export things as soon as they're declared:
`export let foo = 'bar';` for instance.
You can both `import {x as y}` and `export {x as y}`, neat!
You can also import the entire module as an alias, `import * as ModuleName`.
Can have multiple import statements for the same file, or group them together.

# REQUESTS etc.
JS is async by default, it uses an event loop. Functions get called in a stack, functions that make blocking requests get put onto a queue. When the stack is exhausted, things are read from the queue.
JS - double equals will do type conversion, triple equals will not and return false if they are different types.

# PROMISES/OTHER NICE ES6 THINGS
`Promise` - an object that handles async data. Has three states:
`pending` - it's created or waiting for data
`fulfilled` - request handled successfully
`rejected` - operation was unsuccessful

You can chain an additional method to the promise to run after being fulfilled or rejected.
fetch creates the URL and sends the request, then returns a promise.
`.then()` will only fire once the promise status of fetch has been resolved
`.then` takes two parameters, a callback function for success and another for failure.
Chained `.then()`s will only call when the previous has completed, and they have the same success/failure callback functions you can specify.
You need a second `.then()` on success because `response.json()` itself returns a Promise (because getting all of the data may take some time).
`async`/`await` - await can only be in async functions. It just makes the function non-blocking - the line won't continue to operate until that previous line finishes.
So you can write code after it without having to include it in an ugly series of chained then expressions.
You can use a `try`/`catch` to handle the failure case of the `then` call.
`const foo = async () => {};` is the syntax you want.
`async` makes sure your function returns a promise.
`await` says to keep moving through the message queue while the promise resolves.

# FUNCTION HOISTING
Function declarations (a la function foo() {bar}) gets hoisted in its entirety - you can execute foo before it has been declared.
Function expressions in JS are NOT hoisted.