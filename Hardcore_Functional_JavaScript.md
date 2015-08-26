Hardcore Functional Programming in JavaScript
---------------------------------------------

Functional programming is all about separation of concerns and being able to recognize where the separation should occur.

It helps going into this understanding `map`, `reduce`, JavaScript's notion of `this` and context, and `call`.

## The Silence

To write functional code, you must exercise restraint, since a lot of times, "Why couldn't I just do blank, like use `go to`" leads to "Why is the code breaking all the time".

Symptoms of code that can be improved with functional techniques
* custom names - loops with names
* looping patterns - seeing the same kind of loop over and over again
* glue code - code you have to tack on just to make things work with your paradigm 
* side effects - stateful code

#### Omitting needless names

Separating inputs from environment - By omitting useless 'names', creating variables, you reduce chances you introduce state into your functions. Functions should always produce the same output for the same input. 

#### Separating mutation from calculation

Mutation being changing existing values. Good functions should just calculate or just mutate, not both, and not when it's clear that it what's going on. Example: teaser function.

#### Recognizing Pure Function

Reacts the same to the same inputs. Purity make functions testable, portable, memoizable, and parallelizable. Exercises for recognizing pure vs impure functions.

#### Separate functions from rules

Treat functions like nouns. Remember that mathematically a function has only one output set from one input set. Separating arity from functions: removing the number of arguments or operands they may need. Lends itself to currying. Order of arguments may matter

#### Currying example, hints, and solutions

Presenter endorses Ramda.js for its currying function where lodash and underscore are less so.

Try to find the source and work through the example. https://github.com/DrBoolean/hardcorejs.

Hint: recognize libs that do currying for you

#### Compose, hints, and solutions

Like mathematical composion of functions. Again, check the above github repo for the code to try and to solve.

#### Point-Free code

Points refer to arguments being passed from one function call to another within a function. The idea behind Point-Free is in being able to not reference arguments to functions directly, writing it where you use function composition and currying instead of declaring additional variables.

## The Voyage

*Category Theory* - To use Categories, you need to have a composition identity and an id identity, meaning a value with id is equal to itself, and two functions composed must take the same input and generate the same output if they were not composed. There are Category Laws that can be followed if you use Category Theory.

So what about null checks, callbacks, error handling, etc? Let\'s think of Objects as Containers/Wrappers for values, that have no methods, are not nouns, and that you\'ll not be creating your own very often. Containers hold a value, that's it, like an object with a property called `val` and you pass a value into a `new Container()` constructor.

*Drop in the code defining Container, _Container, Container.prototype.map, and compose using Containers*

You can then have functions that operate on Containers. For example, `map()` on a Container takes a function, and `map` passes the Container's val into the argument function, executes it, throws the output into a new Container, and returns the new Container. Basically, with Containers, you can chain `map`s and chain output from functions. We also introduce `map(f, obj)` as the same as `Container.map(f)`.

### Functors

Hah, you've been learning about Functors all along! Functors are objects or data structures you can map over. But what about null checks? Meet the **"Maybe" functor**. "Maybe" functors are a Container that when calling `map`, if the val is truthy, it executes the mapped function and returns a Maybe functor, else it will return a Maybe functor with "null" as the returned Maybe functors value.

*Drop in the code defining Maybes, _Maybes, Maybe.prototype.map, and compose using Maybes*

Also, Containers are actually referred to as the Identity functor.

## The Demo

Try finding the demos/exercises for this part of the course. Try looking at jsbin.com/kamak.

## Either

Like Maybe, but for error handling. The author uses `Right()` and `Left()` where it is a part of the functor? If the value is bad, `Left()` with an error message runs.

## IO

You set up a series of computation with an IO object, and then execute with `runIO`. Think of this as promises or awaits.
