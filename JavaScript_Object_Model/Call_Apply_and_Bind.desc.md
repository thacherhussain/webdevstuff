# Functions: Call Apply and Bind

## Objectives

By the end of this lesson you will be able to:

- Call a function 3 different ways (with parens, using `.call` and using `.apply`)
- Create new functions with `.bind`

### Definitions

* **Execution Context** - This is a way of talking about whatever object is enclosing a function. All functions have an object that they're called from inside. The default calling context is `window` or `global`.
* **Invoked** - to "call" a function- to make the code inside run. `()` is sometimes called the "invocation operator"


One of the more conceptually challenging concepts in JavaScript is the idea of `this`, and setting the value of the keyword `this`.

Call, Apply and Bind are three methods on `Function.prototype` that enable you to invoke a function (or store the result of that function) and set the context of the keyword `this`

### A quick overview of the keyword `this`, and `arguments`

In JavaScript, every time that a function is invoked, two special keywords are created that live in the scope of that function.

1. `arguments` - the keyword arguments is an array-like object (does not have native array methods like push/pop/forEach/map) which represents each argument passed to the function.

```js
    function logArgs(a,b,c){
        return arguments;
    }

    logArgs(1,2,3) // returns [1,2,3]
```

2. `this` - the keyword `this` refers to the parent object in which the function has been called. What does that mean? Simply think whatever the execution context is of that function. Is it a function that is attached to an object? Is it a function that is attached to the `window`? Let's see some examples:

```js

    var person = {
        name: "Elie",
        sayHi: function(){
            return `${this.name} says hi!`
        }
    };

    // the keyword `this` has a parent object called `person`
        // so we are going to access the parent object's property of name
    person.sayHi(); // returns "Elie says hi!"


```

In this example, the keyword `this` refers to the `person` object (sometimes it's easier to just think, what is the function I am calling attached to?). Let's see what happens when we use the keyword `this` without explicitly defining a parent object.

```js
    function hello(){
        return this;
    }

    hello(); // what does this function return?
```

So what happens when we call a function without explicitly defining its parent object? It returns the `window` object! This actually makes a lot of sense, because everything that we define in the global scope (in the browser) is attached to the `window` object. We could actually invoke this function by using `window.hello()`.

## Call, Apply and Bind

One of the more conceptually challenging concepts in JavaScript is the idea of setting the context of the keyword `this`.

Call, Apply and Bind are three methods on `Function.prototype` that enable you to invoke a function (or store the result of that function) and set the context of the keyword `this`

### A quick review of the keyword `this`

In JavaScript, every time that a function is invoked, two special keywords are created (that live in the scope of that function).

1. `arguments` - the keyword arguments is an array-like object (does not have native array methods like push/pop/forEach/map) which represents each argument passed to the function.

```js
    function logArgs(a,b,c){
        return arguments;
    }

    logArgs(1,2,3) // returns [1,2,3]
```

2. `this` - the keyword `this` refers to the parent object in which the function has been called. What does that mean? Simply think whatever the execution context is of that function. Is it a function that is attached to an object? Is it a function that is attached to the `window`? Let's see some examples:

```js

    var person = {
        name: "Elie",
        sayHi: function(){
            return `${this.name} says hi!`
        }
    };

    // the keyword `this` has a parent object called `person`
        // so we are going to access the parent object's property of name
    person.sayHi(); // returns "Elie says hi!"


```

In this example, the keyword `this` refers to the `person` object (sometimes it's easier to just think, what is the function I am calling attached to?). Let's see what happens when we use the keyword `this` without explicitly defining a parent object.

```js
    function hello(){
        return this;
    }

    hello(); // what does this function return?
```

So what happens when we call a function without explicitly defining its parent object? It returns the window object! This actually makes a lot of sense, because everything that we define in the global scope (in the browser) is attached to the window object (we could invoke this function by using `window.hello()`)

### Changing the value of `this` using call or apply

So in these past examples we know with 100% certainty what the value of the keyword this is - so why would we ever want to change it? Well, in these examples we wouldn't, but what about something like this:

```js
     var person = {
        name: "Elie",
        sayHi: function(){
            return `${this.name} says hi!`
        }
    };

    var person2 = {
        name: "Janey",
        sayHi: function(){
            return `${this.name} says hi!`
        }
    }
```

There's a lot of duplication going on here! We just repeated the entire definition of sayHi allover again! So how can we borrow the sayHi function from person but instead of `this.name` referring to "Elie", we want it to refer to "Janey". Call/Apply/Bind to the rescue!


```js
     var person = {
        name: "Elie",
        sayHi: function(){
            return `${this.name} says hi!`
        }
    };

    var person2 = {
        name: "Janey",
    }

    // what if we want to borrow the sayHi method from person to be used on person?

    person.sayHi.call(person2);
```

### The second parameter to call/bind and apply

The key difference between call and apply is the second parameter. Call/Apply/Bind as their first parameter take in what is called the `thisArg` or whatever you want the context of `this` to be. For call, the second to n-th parameters are the arguments to the function you are invoking. With apply there are only 2 parameters. The first is the same as call/bind, but the second is an array of arguments. Here is an example

```js
    function addNumbers(a,b,c,d){
        return a+b+c+d
    }

```

### Common use cases for call, apply and bind

#### Call

- Converting an array-like object (like the keyword `arguments`) into an array (although this can be accomplished with the spread operator in es2015) `Array.prototype.slice.call(arguments)`

#### Apply

- finding the maximum value for multiple numbers (although this can be also accomplished with the spread operator in es2015) `Math.max.apply(this,[1,2,3,4,5])`

- For calling other functions when the arguments are not known. `Apply` works very well with the arguments array. An example would be - write a function that console.logs all of the arguments passed to it

```js
function log(){
  console.log.apply(console,arguments)
}
```

#### Bind

- ensuring that you get the correct value of the keyword `this` when dealing with async code (although this can be accomplished using the arrow syntax with es2015)

```js
    var person = {
        name: "Elie",
        sayHi: function(){
            setTimeout(function(){
                console.log(`${this.name} says hi!`)
            }.bind(this),1000)
        }
    }
```

As we move to more complicated code, you will see `bind` being used quite heavily with events in JavaScript as well as AJAX.

## Exercises

[Call, Apply, Bind](https://github.com/gSchool/cab_closure_exercises)

## Slides

[this, call, apply and bind](https://slides.com/lizh/this-call-apply-bind)

## Resources

- [this article on Call, Apply and Bind](http://dailyjs.com/2012/06/24/this-binding/)
- [Working with `this`](http://dailyjs.com/2012/06/17/js101-this/)
- [Call, Apply, Bind](http://dailyjs.com/2012/06/24/this-binding/)
- [Scope](http://dailyjs.com/2012/07/22/js101-scope/)
- [Functions](http://dailyjs.com/2012/07/01/function)
- [The Function Constructor](http://dailyjs.com/2012/07/08/function-2)
