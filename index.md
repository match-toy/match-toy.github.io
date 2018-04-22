---
---

## What is it?
Match-toy is a pattern matching library for JavaScript. [Pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) is a way to check a sequence of a given input against one or more specific patterns. Many languages like Elixir/Erlang, Rust, F#, Elm, Haskell or Scala have this as a built-in feature.

Pattern matching is a very powerful concept and Match-toy is an attempt to bring this to javascript with an elegant and familiar syntax (jQuery-like chains) plus a simple [domain specific language](https://github.com/match-toy/match-toy/wiki/DSL).

With Match-toy, you'll be able to do:
- [Literal Pattern](https://github.com/match-toy/match-toy/wiki/DSL#literal)
- [Bind Pattern](https://github.com/match-toy/match-toy/wiki/DSL#bind)
- [Typed Pattern](https://github.com/match-toy/match-toy/wiki/DSL#typed)
- [Wildcard Pattern](https://github.com/match-toy/match-toy/wiki/DSL#wildcard)
- [Range Pattern](https://github.com/match-toy/match-toy/wiki/DSL#range)
- [List splitting](https://github.com/match-toy/match-toy/wiki/DSL#rest)
- [Mapping Pattern](https://github.com/match-toy/match-toy/wiki/DSL#mapping)
- [RegExp Pattern](https://github.com/match-toy/match-toy/wiki/DSL#regexp)
- [Logical Pattern](https://github.com/match-toy/match-toy/wiki/DSL#logical-or)
- [As Pattern](https://github.com/match-toy/match-toy/wiki/DSL#as)

All this in about 7.5kb gzipped.

[Try it now](https://npm.runkit.com/match-toy), then check out how to [install](https://github.com/match-toy/match-toy#installation#install) and [use](https://github.com/match-toy/match-toy#usage) it.

#### Interesting but...
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
