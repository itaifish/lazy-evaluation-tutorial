
# Lazy Evaluation

In most programming languages we use today, expressions are evaluated as soon as they are bound to a variable. Thus, if we say
```js
let x = 5;
let y = 6 * Math.pow(5,4);
let z = new Date();
```
*x*, *y* and *z* are all stored in memory as 
* 5 
* 6 * 5<sup>4</sup>
* The current datetime at the instant line 3 is evaluated

However, some languages, like Haskell, have something called **Lazy Evaluation**. This means that the variables are not computed until their value is needed. If *x*, *y*, or *z* is never referenced again, its value will never be calculated. For *z* specifically, when the variable is referenced actually changes what its value is - the later it is referenced, the more milliseconds since epoch it will store. 

Lazy Evaluation lets the user do things that otherwise would be basically impossible to accomplish, namely infinite lists. Since variables are only calculated when referenced, the list would not be infinite in terms of memory allocated, since it only saves values the program needs/uses. For example, the Haskell code:
```haskell
 fibs = 0 : 1 : zipWith (+) fibs (tail fibs)
```
Creates a list of all Fibonacci numbers. 


In order to be efficient, implementations of Lazy Evaluation take advantage of memoization (this isn't a typo, the 'R' really is missing!), where the result of values are stored in O(1) lookup tables as they are computed. Thus, whenever the program needs to access a variable, it first looks up if it has previously calculated the variable's value. If so, it will return that pre-calculated value. Otherwise, it will calculate it, store the result in the memoization table, and then return it. 

### Pros and Cons
#### Pros
1. Loading large and infrequently accessed data is much faster
2. More efficient - The program never wastes CPU cycles computed information that is unused
3.  Allow for infinite data structures
#### Cons
1. Memory usage becomes hard to predict
2. Memory leaks from thunk<sup>*</sup> buildup become something to worry about 
3. Difficult to combine with exception handling and I/O


### In Racket

As is likely topical for our COMP 524 class, Racket has multiple lazy evaluation features. By default, function arguments are eager, and special form arguments are lazy<sup>1</sup>. However, we can set the entire language to default to lazy evaluation by setting the language to *lazy*. 
In order to see it in action, try compiling the following code:
```scheme
#lang lazy
(require racket/promise)
(write (current-seconds))
(define x (current-seconds))
```
Wait a couple of Mississippi's, then type into the console:
```scheme
(force x)
```
Racket will then force an evaluation for x, giving a value that is however many seconds you waited more than the first value printed to console.
![Picture of the above code run in DrRacket](https://cdn.discordapp.com/attachments/277950567559725059/813300249682509824/unknown.png)

Try this code on your own, and see if it operates how you'd expect it to.
```scheme
#lang lazy
(require racket/promise)
(define comparison (current-seconds))
(force comparison)
(define x (current-seconds))
(define y (current-seconds))
(define z (current-seconds))
(format "~a Seconds off from the comparison Value"  (- (force x) (force comparison)))
(sleep 5)
(format "~a Seconds off from the comparison Value"  (- (force y) (force comparison)))
(sleep 2)
(format "~a Seconds off from the comparison Value"  (- (force z) (force comparison)))
(format "Once a value is computed, it will stay that value until reassigned. Thus this value should still be 0: ~a"  (- (force x) (force comparison)))
```
See [The Official Docs](https://docs.racket-lang.org/lazy/index.html) for more information

### TL;DR

Lazy Evaluation allows the program to wait until a variable's value is needed before computing it. This matters if you're using a language that has lazy evaluation, as it allows for efficient loading of chunks of large data, infinite lists, and can mean that the order of when a variable is first accessed can have an effect on its result. 

### Footnotes

**\*** Thunks in Racket are zero-argument functions to delay evaluation. There are lots of tricks with the racket language that involve using thunks as a way to enable lazy evaluation. See [here](http://www.cs.rhodes.edu/~kirlinp/courses/proglang/s13/360-lect12-slides.pdf) for details.


### Sources

1. https://medium.com/background-thread/what-is-lazy-evaluation-programming-word-of-the-day-8a6f4410053f
2. https://wiki.haskell.org/Lazy_evaluation
3. http://www.cs.rhodes.edu/~kirlinp/courses/proglang/s13/360-lect12-slides.pdf 
4. https://www.tutorialspoint.com/functional_programming/functional_programming_lazy_evaluation.htm
5. https://docs.racket-lang.org/lazy/index.html#%28form._%28%28lib._lazy%2Fmain..rkt%29._begin%29%29
6. http://blog.ezyang.com/2011/05/space-leak-zoo/
7. Fibonacci Haskell Code: https://en.wikipedia.org/wiki/Lazy_evaluation#cite_note-MachineryLanguages2002-15 / https://books.google.com/books?id=hsBQAAAAMAAJ
