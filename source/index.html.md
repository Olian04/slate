---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript
  - typescript

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Turing.js API! 

# Demos

## Super simple counter

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

This is 

# Language

## new Language()

```javascript
const myLanguage = new Language();
```

```typescript
interface MyData {
  [key: string]: any;
}

const myLanguage = new Language<MyData>();
```

> While not necessary _(as seen in the [Super simple counter demo](#super-simple-counter))_, it is recommended to store the new Language object in a variable or constant

When using typescript the Language object requires an interface to be passed to it. This interface will be used to define the `data` field inside of the `state` object. The `state` object will then be passed as an argument to every handler-function attached to the Language object.

<aside class="notice">
Providing a detailed interface will greatly improve the development experience, as it not only provides type safety, but also allows editors and IDE's to provide autocompletion and type checks on both the `data` filed and the various overloads of the `.data()` method.
</aside>

## .token(name, handler)

```javascript
```

```typescript
```
