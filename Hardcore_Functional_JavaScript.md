Hardcore Functional Programming in JavaScript
---------------------------------------------

Functional programming is all about separation of concerns and being able to recognize where the separation should occur.

It helps going into this understanding `map`, `reduce`, JavaScript's notion of `this` and context, and `call`.

## The Silence

To write functional code, you must exercise restraint, since a lot of times, "Why couldn't I just do ___, like use `go to`" leads to "Why is the code breaking all the time".

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

## The Demo
