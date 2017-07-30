---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - typescript: Typescript
  - javascript: Javascript

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:

search: true
---

# Turing.js

```javascript
const Language = require('turingjs').Language;

new Language()
    .token('+', state => { state.data.sum++ })
    .token('-', state => { state.data.sum-- })
    .data({ sum: 0 })
    .run('+--++-++++')
    .then(state => console.log(state.data.sum))
    .catch(err => { throw err });
```

```typescript
import { Language } from 'turingjs';

new Language<{ sum: number }>()
    .token('+', state => { state.data.sum++ })
    .token('-', state => { state.data.sum-- })
    .data({ sum: 0 })
    .run('+--++-++++')
    .then(state => console.log(state.data.sum))
    .catch(err => { throw err });
```

> The code above would print `4` to the console

Turing.js is an Event-Driven-Language-Design-API based on the definition of a turing machine.

_A Turing machine is an abstract machine that manipulates symbols on a strip of tape according to a table of rules. The machine operates on an infinite memory tape divided into discrete cells. The machine positions its head over a cell and "reads" the symbol there. Then, as per the symbol and its present place in a finite table of user-specified instructions, the machine (i) writes a symbol (e.g. a digit or a letter from a finite alphabet) in the cell, then (ii) either moves the tape one cell left or right, then (iii) (as determined by the observed symbol and the machine's place in the table) either proceeds to a subsequent instruction or halts the computation._


# Language()

```javascript
const myLanguage = new Language();
```

```typescript
interface MyData {
  [key: string]: any;
}

const myLanguage = new Language<MyData>();
```

> While not necessary _(as seen in the [introduction](#turing-js))_, it is recommended to store the new Language object in a variable or constant.

<aside class="notice">
Typescript - When using typescript the Language object requires an interface to be passed to it. This interface will be used to define the <code>data</code> field inside of the <code>state</code> object. The <code>state</code> object will then be passed as an argument to every handler-function attached to the Language object.
Providing a detailed interface will greatly improve the development experience, as it not only provides type safety, but also allows editors and IDE's to provide autocompletion and type checks on both the <code>data</code> field and the various overloads of the <code>.data()</code> method.
</aside>

## .token()

### .token(token, handler)

```javascript
const myLanguage = new Language()
  .token('+', function(state, token) { 
      // Handler body
    });
```

```typescript
const myLanguage = new Language<{}>()
  .token('+', (state, token) => { 
      // Handler body
    });
```

Every time the string 

#### Parameters

Name | Type 
-----|-----
token | string 
handler | [TokenHandler](#tokenhandler)

### .token({ token: handler })

```javascript
```

```typescript
```

### .token(handler)

```javascript
```

```typescript
```

## .data(key, value)

```javascript
```

```typescript
```

## .data({ key: value })

```javascript
```

```typescript
```

## .on(event, handler)

```javascript
```

```typescript
```

## .run(code)

```javascript
```

```typescript
```

## .run(code, onSuccess, onError)

```javascript
```

```typescript
```

<aside class="notice">
Providing the <code>onError</code> callback is optional.
</aside>

# Handler types

<aside class="notice">
All handler types are shown in Typescript but their signature applies the same to both Javascript and Typescript. 
</aside>

Most handlers take 2 parameters, a `state` object, and a `token` object.

### State object

```ts
interface IState<T> {
    tokenPointer: number,
    stack: number[],
    index: number,
    data: Partial<T>
}
```

The `state` object has 4 fields, `tokenPointer`, `stack`, `index`, and `data`.

Field | Description
------|------------
`tokenPointer` | The token pointer works much like an instruction pointer in assembly, it points to the token thats being executed.
`stack` | The stack will automatically grow into the positive indices.
`index` | The index that determine where on the stack the program should read or write from.
`data` | Reserved for user defined data.

### Token object

```ts
interface IToken {
    position: number,
    value: string
}
```
The `state` object has 2 fields, `position` and `value`.

Field | Description
------|------------
`position` | The position points to the number of tokens that came before this one. If the position is `0` then this is the first token.
`value` | The value is the exact string that matched in order to trigger this callback.

## TokenHandler

```ts
type TokenHandler<T> = (state: IState<T>, token?: IToken) => (void | SkipHandler<T>);
```

A TokenHandler is a method that takes a `state` object plus an optional `token` object, and optionally returns a [SkipHandler](#skiphandler).

If a SkipHandler is returned, normal execution will be put aside. As long as the SkipHandler returns true, the SkipHandler will be called instead of the regular TokenHandlers whenever a token is processed.

<aside class="warning">
Returning a <a href="#skiphandler">SkipHandler</a> will pause normal execution
</aside>

## SkipHandler

```ts
type SkipHandler<T> = (state: IState<T>, token?: IToken) => boolean;
```

A SkipHandler is a method that takes a `state` object plus an optional `token` object, and returns a boolean. 

The SkipHandler is a method where as long as it returns true, it will be called instead of the regular TokenHandlers whenever a token is processed.
The returned boolean will determine wether or not the SkipHandler should keep being used.

## EOFHandler

```ts
type EOFHandler<T> =  (state: IState<T>) => boolean;
```

A EOFHandler is a method that takes a `state` object and returns a boolean. 

The EOFHandler is a method thats called once the interpreter reaches the end of file. Returning `true` tells the interpreter that the EOF was valid and for the interpreter not to throw an error.

# Demos

## Brainfuck

```typescript
import { Language } from 'turingjs';

interface BFData {
  in: string[],
  out: string[],
  loops: number[]
}
let brainfuck = new Language<BFData>()
    .token({
        '+': state => { state.stack[state.index]++ },
        '-': state => { state.stack[state.index]-- },
        '>': state => { state.index++ },
        '<': state => { state.index-- },
        ',': state => { state.stack[state.index] = state.data.in.shift().charCodeAt(0) },
        '.': state => { state.data.out.push(String.fromCharCode(state.stack[state.index])) },
        '[': (state, token) => {
            state.data.loops.push(token.position)
            if (state.stack[state.index] === 0) {
                return (state, token) => {
                    if (token.value === '[') { state.data.loops.push(NaN) }
                    if (token.value === ']') { state.data.loops.pop() }
                    return state.data.loops.length === 0;
                };
            }
        },
        ']': (state, token) => { 
            state.tokenPointer = state.data.loops.pop() - 1; 
            // -1 in order to offset the auto increment done by turingjs
        },
    })
    .on('eof', state => state.data.loops.length === 0)
    .data({ in: [], out: [], loops: [] });

brainfuck
    .data({ in: 'ABC'.split('') })
    .run('+++[->,.+++.<]')
    .then(finalState => console.log(finalState.data.out.join('')))
    .catch(error => console.log(error));
```

```javascript
let Language = require('turingjs').Language;

let brainfuck = new Language()
    .token({
        '+': state => { state.stack[state.index]++ },
        '-': state => { state.stack[state.index]-- },
        '>': state => { state.index++ },
        '<': state => { state.index-- },
        ',': state => { state.stack[state.index] = state.data.in.shift().charCodeAt(0) },
        '.': state => { state.data.out.push(String.fromCharCode(state.stack[state.index])) },
        '[': (state, token) => {
            state.data.loops.push(token.position)
            if (state.stack[state.index] === 0) {
                return (state, token) => {
                    if (token.value === '[') { state.data.loops.push(NaN) }
                    if (token.value === ']') { state.data.loops.pop() }
                    return state.data.loops.length === 0;
                };
            }
        },
        ']': (state, token) => { 
            state.tokenPointer = state.data.loops.pop() - 1; 
            // -1 in order to offset the auto increment done by turingjs
        },
    })
    .on('eof', state => state.data.loops.length === 0)
    .data({ in: [], out: [], loops: [] });

brainfuck
    .data({ in: 'ABC'.split('') })
    .run('+++[->,.+++.<]')
    .then(finalState => console.log(finalState.data.out.join('')))
    .catch(error => console.log(error));
```

> The code above would print `ADBECF` to the console

And that right there is an interpreter for the entire Brainfuck language, as simple as that, in less than 30 lines of code.

<aside class="success">
I'm sure there are shorter ways of writing this, if you can come up with a shorter version using turingjs, open an issue on github and tell me all about it!
</aside>