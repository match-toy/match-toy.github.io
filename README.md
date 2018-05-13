## Match-Toy

> The pattern matching library for javascript.

[![Build Status](https://travis-ci.org/match-toy/match-toy.svg?branch=master)](https://travis-ci.org/match-toy/match-toy)
[![Test Coverage](https://api.codeclimate.com/v1/badges/b4ef038eea888a8a8cfb/test_coverage)](https://codeclimate.com/github/match-toy/match-toy/test_coverage)
[![Greenkeeper badge](https://badges.greenkeeper.io/match-toy/match-toy.svg)](https://greenkeeper.io/)
[![npm version](https://badge.fury.io/js/match-toy.svg)](https://badge.fury.io/js/match-toy)
[![Try match-toy on RunKit](https://badge.runkitcdn.com/match-toy.svg)](https://npm.runkit.com/match-toy)

<iframe allowtransparency="true" scrolling="no" style="width: 63px; height: 28px; border: medium none;" src="https://buttons.github.io/buttons.html#href=https%3A%2F%2Fgithub.com%2Fmatch-toy%2Fmatch-toy&amp;title=&amp;aria-label=Star%20match-toy%2Fmatch-toy%20on%20GitHub&amp;data-icon=octicon-star&amp;data-text=Star&amp;data-size=large" frameborder="0"></iframe>

<iframe allowtransparency="true" scrolling="no" style="width: 150px; height: 28px; border: medium none;" src="https://buttons.github.io/buttons.html#href=https%3A%2F%2Fgithub.com%2Fmatch-toy&amp;title=&amp;aria-label=Follow%20%40match-toy%20on%20GitHub&amp;data-text=Follow%20%40match-toy&amp;data-size=large" frameborder="0"></iframe>

## What is it?
Match-toy is a pattern matching library for JavaScript. [Pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) is a way to check a sequence of a given input against one or more specific patterns. Many languages like Elixir/Erlang, Rust, F#, Elm, Haskell or Scala have this as a built-in feature.

Pattern matching is a very powerful concept and Match-toy is an attempt to bring this to javascript with an elegant and familiar syntax (jQuery-like chains) plus a simple [domain specific language](#dsl).

With Match-toy, you'll be able to do:
- [Literal Pattern](#literal)
- [Bind Pattern](#bind)
- [Typed Pattern](#typed)
- [Wildcard Pattern](#wildcard)
- [Range Pattern](#range)
- [List splitting](#rest)
- [Mapping Pattern](#mapping)
- [RegExp Pattern](#regexp)
- [Logical Pattern](#logical-or)
- [As Pattern](#as)

All this in about 7.5kb gzipped.

[Try it now](https://npm.runkit.com/match-toy), then check out how to [install](#install) and [use](#usage) it.

## Interesting but...
If you think this looks like an overengineering `switch..case`, let's compare some code then you make your mind.

Let's say, hypothetically, we have to create a function that extracts the name and the address of a user from an object then return a string with a human-friendly message. A kind of task very common on daily work.

So, on a plain javascript version, this function could be something like this:

```javascript
const getUserResponse = (response) => {
  try {
    if(response && response.status === 200) {
      if(response.user) {
        
        let name;
        let address;
        
        if(response.user.name && typeof response.user.name === 'string') {
          name = response.user.name;  
        }

        address = Object.keys(response.user)
              .filter((key) => key !== 'name')
              .reduce((acc, it) => {
                acc[it] = response.user[it];
                return acc;
              }, {});
        if(name && address) {
          return `User name is ${name} and lives on ${formatAddress(address)}` 
        }
        
      } else {
        return 'No user found';
      }
    } else {
      return 'No user found';
    }
  } catch(e) {
    console.log('Error on getUserResponse', e)
  }
}

console.log(getUserResponse(/* user data from server maybe... */))

// About 33 lines of code
```
What if we do the same with Match-toy:
```javascript
// Using match-toy
import { match } from 'match-toy'

const getUserResponse = match()
  .case('{ status: 200, user: { name:String, ...address } }', ({name, address}) => 
    `User name is ${name} and lives on ${formatAddress(address)}`)
  .else(() => 'No user found')
  .catch((e) => console.log('Error on getUserResponse', e))
  .end()
  
console.log(getUserResponse(/* user data from server maybe... */))

// About 9 lines of code
```

Did you notice the difference in terms of readability and maintainability? That's why pattern matching is awesome!

## Getting started
- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [DSL](#dsl)

A good way to learn is by example. So the best kick off is [read the tests](./src), there are tons of them covering all the cases. Another source is the [examples](./examples) folder, especially the [tic-tac-toe](./examples/tic-tac-toe).

### Install

#### From NPM
```sh
$ npm install match-toy --save
```
Or yarn:
```sh
$ yarn add match-toy
```
Then import/require the module.
```javascript
const { match } = require('match-toy');
// or
import { match } from 'match-toy';
```

#### From CDN
Place the snippet into your html:
```html
<script src="https://cdn.jsdelivr.net/npm/match-toy/dist/bundle/index.min.js"></script>
```
For specific version append the desired version (on the format `@x.x.x`) before the word `match-toy` just like this:   `https://cdn.jsdelivr.net/npm/match-toy@2.0.0/dist/bundle/index.min.js`.

This file is a bundle in the [UMD](https://github.com/umdjs/umd) format. In browser's environments, the module name is in camelcase and available on `window` scope.
```javascript
var myFunc = matchToy.match()
                      .case('1', () => 'one')
                      .end()
```

See more in [examples](./examples).

### Usage
Most basic usage:
```javascript
import { match } from 'match-toy';

// Create a new pattern matching function
const convertOneToString = match()
  .case('1', () => 'one')
  .end();

convertOneToString(1); // return 'one'
convertOneToString(2); // return undefined

// Create another one, but now we only need
// the value returned by the match
const one = match()
  .case('1', () => 'one')
  .return(1); // using `return()` match runs immediately

one === 'one'; // true
```
Let's dig around the library features and see what else we can do.

#### Declaring patterns
The `case()` functions are used to define patterns and callbacks:
```javascript
const myFunc = match()
  .case('1', () => 'one')
  
  // every bound variable is passed into an object 
  .case('x, 1', (boundVars) => 'X is ' + boundVars.x) 
  
  // using es6 destructuring, things looks much better
  .case('2, y', ({ y }) => `Y is ${y}`) 

  // you can use values as well
  .case('0, 0', 'zero') 
  .end()

myFunc(2, 1); // === 'X is 2'
myFunc(1); // === 'one'
myFunc(2, 3); // === 'Y is 3'
myFunc(0, 0); // === 'zero'

```
The first argument is a string under javascript point of view. But it is not a simple string. Actually, this is a [DSL](#dsl) with which we can express the pattern easier than using a complicated data or function structures.

You can define patterns as much as you want. As you can notice in the example above when the first pattern matches the related callback is invoked. Then every variable is passed as an object member. So keep in mind that order is important.


#### Guards
Guards are a way to declare a condition. It is defined with `when()`:
```javascript
const myFunc = match()
  .case('x', ({x}) => `${x} is even`).when(({x}) => x % 2 === 0)
  .case('x', ({x}) => `${x} is odd`)
  .end()

myFunc(2); // === '2 is even'
myFunc(3); // === '3 is odd'

```
In this case, the pattern is exactly the same. But since there is guard defining a condition, `myFunc` returns different values. The guard function will receive the matched values, just like the `case()` callbacks.

Only one guard per pattern is allowed:
```javascript
const myFunc = match()
  .case('x, y', () => 'are equal').when(({x, y}) => x === y)
  .case('x, y', () => 'x > y').when(({x, y}) => x > y)
  .case('x, y', () => 'x < y').when(({x, y}) => x < y)

  .case('x, y', () => 'wrong')
    .when(({x, y}) => !!x)
    .when(({x, y}) => !!y)  // more than one when() throws an error

  .case('x, y', (() => `${x}, ${y}`))
  .end()

```

#### Do
Alternatively, you can define the pattern callback using the `do()`:
```javascript
const myFunc = match()
  // Common (and recommended) syntax
  // case(<pattern>, <callback>).when(condition)
  .case('x, y', () => 'x < y').when(({x, y}) => x < y)
  
  // `do` syntax
  // case(<pattern>).do(<callback>).when(condition)
  .case('x, y').do(() => 'x > y').when(({x, y}) => x > y)
  
  // `when` and `do` inverted
  // case(<pattern>).when(condition).do(<callback>)
  .case('x, y').when(({x, y}) => x > y).do(() => 'x > y')
  
  // Or even you can use only `case`
  // case(<pattern>, <callback>, <condition>)
  .case('x, y', () => 'are equal', (x, y) => x === y)
  .end()

```

#### Else
If there is no match, the default return value is `undefined`:
```javascript
const myFunc = match()
  .case('1', () => 'one')
  .end()

myFunc(3) // === undefined

```
But you can use an `else()` in order to handle no matching cases.
```javascript
const myFunc = match()
  .case('1', () => 'one')
  .case('2', () => 'two')
  .else(() => 'I give up gracefully')
  .end()

myFunc(1) // === 'one'
myFunc(3) // === 'I give up'

```

#### Catch
Sometimes our callbacks may throw an error for some reason. Because of that, they are wrapped into a try..catch.
```javascript
const myFunc = match()
  .case('true', () => JSON.parse('wrong json syntax'))
  .end();

myFunc(true) // === Match error: SyntaxError: JSON.parse...
```
With `catch()` you can define a function to handle this cases for a better response or some kind of recovery strategy.
```javascript
const myFunc = match()
  .case('true', () => JSON.parse('wrong json syntax'))
  .catch((e) => 'Oh! No! Not again.')
  .end();

myFunc(true) // === 'Oh! No! Not again.'
```

#### Nesting
You can achieve nesting matching by simply creating a new pattern match inside the callback:
```javascript
const myFunc = match()
  .case('0', () => 'is zero')
  .case('_, x', ({x}) => match()
    .case('y', () => 'Y is even').when(({y}) => y % 2 === 0)
    .case('y', () => 'Y is odd')
    .return(x))
  .end();

myFunc(2, 2) // Y is even
```

### DSL
So far, we covered how to use the library functions. Now, let talk about what makes Match-toy shines. As discussed above, the first argument of the `case()` function is not really a string. It is [Domain Specific Language](https://en.wikipedia.org/wiki/Domain-specific_language), designed to make patterns definitions easier.  [Other libraries](#other-nice-projects-and-initiatives) have different approaches like object schemas or extending the language with macros _(macros are great BTW)_. But for the sake of expressiveness and simplicity Match-toy use a really powerful and straightforward declarative language. Right below, you'll find out everything you need to start using it.

#### Sequence of values
All patterns expect a defined order and length. If a given input doesn't match either, the evaluation fails and go to the next pattern.
```javascript
.case('1')
// Given 1 matches
// Given 1, 2 not matches

.case('1, 2')
// Given 1, 2 matches
// Given 1 not matches

.case('2, 1')
// Given 2, 1 matches
// Given 1, 2 not matches

```

#### Literal
The literal pattern is used when an equal value match is expected.
```javascript
                  // Supported types
.case('1')        // Numbers
.case('"string"') // String
.case('true')     // Boolean
.case('{ a: 1 }') // Objects
.case('[ 1, 2 ]') // Arrays
.case('{}')       // empty Objects
.case('[]')       // empty Arrays

.case('1, 2, 3')  // sequence of values  
.case('true, "text", { a: 1 }, [true, true]') // sequence of mixed values
```
#### Bind
The bind pattern assigns the matched value to a variable.
```javascript
// Bind any value to variable 'x'
.case('x')

// Bind the second value to variable 'x'
.case('1, x')

// Bind any values from the sequence respectively 
// to the variables 'a', 'b' and 'c'
.case('a, b, c')

// Bind the second item of an array to the variable 'second'
.case('[1, second]')

// Bind any value of the property b from an object
// to the variable 'myvar'
.case('{ a: 1, b: myvar')
```
#### Wildcard
This pattern will match anything, despite the value, then it is ignored. This is very useful for ignore irrelevant values inside a sequence or list. Another case is use wildcards as a final clause if none of the previous patterns matches.
```javascript
.case('_')      
.case('_, 2')   
.case('[2, _]') 
.case('{ a: 1, b: _ }') 

// Example of wildcard as final match
.case('0', () => 'zero')
.case('1', () => 'one')
.case('_', () => 'not binary')

// But remember, order is important
.case('_', () => 'not binary') // every value would match
.case('0', () => 'zero')       // unreachable
.case('1', () => 'one')        // unreachable


```
#### Typed
It is possible to use types in order to qualify the values.
```javascript
// Bind only string type value to the variable 'x'
.case('x:String')        

// Match any value that is a Number type
.case('Number')

// Bind the value of the property 'b' to
// the variable 'myvar', if it is a string
.case('{ a: Number, b: myvar:String }')

// Match an array where every item is a string
.case('[...]:String') 

```

Supported types: 
```javascript
.case('String')    // "what's the meaning of life"
.case('Number')    // 42
.case('Boolean')   // true
.case('Undefined') // undefined
.case('Null')      // null
.case('Array')     // []
.case('Object')    // {}
.case('Function')  // () => {/*code*/}
.case('RegExp')    // /a-z/
.case('NaN')       // Nan
.case('Date')      // new Date()

// Nullable is a special type. Any empty object,
// empty array, empty string, zero, undefined or null.
.case('Nullable')  // {}, [], '', 0, undefined, null
```

In can use 'custom types' by checking if the input is an instance of an Class/Function/Object:
```javascript

const red = new Color();
const result = match(red)
  .case('Color', () => 'it\'s a color') // true
  .else(() => 'it\'s not a color')
  .end()
```

#### Range
This pattern can match a range of values.
```javascript
.case('1..5')        // A range of numbers from 1 to 5
.case('1..5, 10')    // Numbers
.case('a..z')        // A range of chars from a to z
.case('A..Z')        // A range of chars from A to Z
.case('A..z')        // A range of chars from A to z, so something like (A, B, C ... Z, a, b, c ... z)
```

#### Rest
The rest pattern, is very similar to the rest operator of Javascript. 
```javascript
// Match an object with the property 'a' equal to 1 then
// capture the rest of the properties and assign to the variable 'others'.
// e.g.: given { a: 1, b: 2: c: 3 } then others === { b: 2, c: 3 }
.case('{ a: 1, ...others}')     

// Split an array by assigning the first item to the variable 'head'
// and the rest of the list to the variable 'tail'
.case('[ head, ...tail]')       

// Again, split an array binding the first two values
// but now, the rest of the list is ignored
.case('[ first, second, ...]')

// In this case, we capture the first the items and the last one.
// Every thing else in the middle is binding to dtoy variable.
.case('[ a, b, c, ...dtoy, z]')

// Match an array with one or more items
.case('[ ... ]')

// Match an object with one or more items
.case('{ ... }')
```

### Mapping
Mapping pattern allow us to define a pattern to be used over every item of an array or object. The is perfect when we need to filter and/or deconstruct a dataset. This is like an extension of the Rest pattern.
```javascript
const marvelCharacters = [
  { name: 'Spiderman', alterEgo: 'Peter Parker', type: 'hero' },
  { name: 'IronMan', alterEgo: 'Tony Stark', type: 'hero' },
  { name: 'Doctor Doom', alterEgo: 'Victor Von Doom', type: 'villain' },
  { name: 'Venom', alterEgo: 'Eddie Brock', type: 'villain' }
 ];

const getHeroes = match()
  .case('[...characters({ name: hero, type: "hero", ... })]')
    .do(({characters}) => characters)
  .end();

getHeroes(marvelCharacters);
/*
 the result will be:
 [
   { hero: 'Spiderman' },
   { hero: 'IronMan' }
 ]      
*/
```

#### RegExp
You can define regular expressions.
```javascript
.case('/^(http|https)/')
```

#### Logical Or
With the operator `|` you can combine multiple patterns in order to match one of them.
```javascript
.case('2 | 4') // will match either 2 or 4
.case('1, _ | 2, _ ')
// given (1, 2) then match
// given (2, 8) then not match
// given (3, 1) then not match
// given (3) then not match
```
#### Logical And
With the operator `&` you can combine multiple patterns in order to match all of them.
```javascript
// should match the input 2, 4 and bind 2 to the variable `x`
.case('2, x & _, 4')
```
#### As
The As pattern is tool to define alias for literals and groups.
```javascript

// Using with literals
.case('1, x@2, 3', ({ x }) => x === 2)
.case('200, resp@true', ({ resp }) => resp === true)

// Using with groups
.case('firstTwo@(1, 2), 3, 4', ({ firstTwo }) => firstTwo === [ 1, 2 ])

.case('all@(x, y, z)', ({ x, y, z, all }) => { 
  assert(x === 1)
  assert(y === 'two')
  assert(z === true)
  assert(all === [ 1, 'two', true ])
})  

.case('all@("one", x@"two")', ({ all, x }) => { 
  assert(all === ['one', 'two']);
  assert(x === 'two');
 })

// Using with lists
.case('list@[1, 2, 3]', ({ list }) => assert(list === [1, 2, 3]))

.case('list@[1, 2, last@3]', ({list, last}) => {
  assert(list === [1, 2, 3]);
  assert(last === 3);
})

// Using with objects
.case('hash@{ one: 1, two: 2 }', ({ hash }) => {
  assert(hash === { one: 1, two: 2 });
})

.case('hash@{ a@one: 1, b@two: 2 }', ({ hash, a, b }) => {
  assert(hash === { one: 1, two: 2 })
  assert(a === 1)
  assert(b === 2)
})

// You can reuse the key name for the alias with
// this syntax variation
.case('hash@{ one@: 1, two@: 2 }', ({ hash, one, two }) => {
  assert(hash === { one: 1, two: 2 })
  assert(one === 1)
  assert(two === 2)
})

// Using with mapping patterns 
.case('[...heroes{ name, type@: "hero", publisher@: String }]')
/* given ([ 
  { id: 0, name: 'Clark Kent', type: 'hero', publisher: 'DC' }
  { id: 1, name: 'Lex Luthor', type: 'villain', publisher: 'DC' }
  { id: 2, name: 'Luke Cage', type: 'hero', publisher: 'Marvel' }
 ]) then
   heroes: [
    { name: 'Clark Kent', type: 'hero', publisher: 'DC' }
    { name: 'Lex Luthor', type: 'villain', publisher: 'DC' }
    { name: 'Luke Cage', type: 'hero', publisher: 'Marvel' }
   ]
 }
*/

```

### API
#### `match(...value?: any): any | function`
Start pattern match definition. If one or more arguments are passed `match()` will return the result of the matching. Otherwise, the return will be a function.
#### `case(pattern: string, callback?: function | any, guard?: function)`
Define a patten. Optionaly can set the callback and a guard. If suits better, you can pass a value to be returned instead of a function for the callback parameter.
#### `with(pattern: string, callback?: function | any, guard?: function)`
Alias for `case()`.
#### `when((...input?: any) => boolean)`
Define a guard. Receive a function as an argument. This function will receive the sequence of input, then should return a boolean value. For example, if the input is `match(1, 2, 3)`, a valid guard must be `(a, b, c) => a > b > c`.
#### `do(callback: function)`
Define a callback
#### `else(callback: function)`
Define a callback if no pattern maches
#### `catch(errorHandler: function)`
Define a callback if exception is throwed from callbacks
#### `end()`
Finish pattern match declaration chain.
#### `return(...value: any)`
Finish pattern match declaration and return the match value.

## Development

### Run build
```sh
$ npm run build
```

### Running tests
Run test with coverage:
```sh
$ npm test
```
Run test and watch:
```sh
$ npm run test:watch
```

### Linter
```sh
$ npm run lint  
```

### Built with
- [PEG.js](https://pegjs.org/)
- [Typescript](https://www.typescriptlang.org/)
- [Jest](https://facebook.github.io/jest/)
- [Rollup](https://rollupjs.org/)
- [Yarn](https://yarnpkg.com/en/)
- [Testcheck-js](https://github.com/leebyron/testcheck-js)

## References about Pattern Matching and DSL
- https://en.wikipedia.org/wiki/Domain-specific_language
- https://en.wikipedia.org/wiki/Pattern_matching
- https://fsharpforfunandprofit.com/posts/match-expression/
- https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/pattern-matching
- https://en.wikibooks.org/wiki/F_Sharp_Programming/Pattern_Matching_Basics#Using_Guards_within_Patterns
- https://en.wikibooks.org/wiki/Haskell/Pattern_matching
- http://learnyousomeerlang.com/syntax-in-functions

## Other nice projects and initiatives
Syntax proposals:
- https://github.com/tc39/proposal-pattern-matching
- https://github.com/eborden/JS-Pattern-Matching
- https://gist.github.com/bterlson/da8f02b95b484cd4f8d9

Other JavaScript libraries:
- https://codemix.github.io/flow-runtime/#/docs/pattern-matching
- https://github.com/HerringtonDarkholme/Pat-Mat
- https://github.com/natefaubion/sparkler
- https://github.com/bramstein/funcy
- https://github.com/FGRibreau/match-when
- https://github.com/z-pattern-matching/z
- https://github.com/dherman/pattern-match
- https://github.com/mcollina/bloomrun

## Contributing
- Improving or correcting the documentation.
- Translating.
- Finding bugs
- Sharing this project.
- PR are very welcome.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog
See [CHANGELOG](CHANGELOG.md) file for details.
