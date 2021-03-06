# event-target-shim

[![npm version](https://img.shields.io/npm/v/event-target-shim.svg)](https://www.npmjs.com/package/event-target-shim)
[![Downloads/month](https://img.shields.io/npm/dm/event-target-shim.svg)](http://www.npmtrends.com/event-target-shim)
[![Build Status](https://travis-ci.org/mysticatea/event-target-shim.svg?branch=master)](https://travis-ci.org/mysticatea/event-target-shim)
[![Coverage Status](https://codecov.io/gh/mysticatea/event-target-shim/branch/master/graph/badge.svg)](https://codecov.io/gh/mysticatea/event-target-shim)
[![Dependency Status](https://david-dm.org/mysticatea/event-target-shim.svg)](https://david-dm.org/mysticatea/event-target-shim)

An implementation of [WHATWG EventTarget interface](https://dom.spec.whatwg.org/#interface-eventtarget), plus few extensions.

- This provides `EventTarget` constructor that can inherit for your custom object.
- This provides an utility that defines properties of attribute listeners (e.g. `obj.onclick`).

```js
import {EventTarget, defineEventAttribute} from "event-target-shim"

class Foo extends EventTarget {
    // ...
}

// Define `foo.onhello` property.
defineEventAttribute(Foo.prototype, "hello")

// Use
const foo = new Foo()
foo.addEventListener("hello", e => console.log("hello", e))
foo.onhello = e => console.log("onhello:", e)
foo.dispatchEvent(new CustomEvent("hello"))
```

## 💿 Installation

Use [npm](https://www.npmjs.com/) to install then use a bundler.

```
npm install event-target-shim
```

Or download from [`dist` directory](./dist).

- [dist/event-target-shim.mjs](dist/event-target-shim.mjs) ... ES modules version.
- [dist/event-target-shim.js](dist/event-target-shim.js) ... Common JS version.
- [dist/event-target-shim.umd.js](dist/event-target-shim.umd.js) ... UMD (Universal Module Definition) version. This is transpiled by [Babel](https://babeljs.io/) for IE 11.

## 📖 Usage

```js
import {EventTarget, defineEventAttribute} from "event-target-shim"
// or
const {EventTarget, defineEventAttribute} = require("event-target-shim")

// or UMD version defines a global variable:
const {EventTarget, defineEventAttribute} = window.EventTargetShim
```

### EventTarget

> https://dom.spec.whatwg.org/#interface-eventtarget

#### eventTarget.addEventListener(type, callback, options)

Register an event listener.

- `type` is a string. This is the event name to register.
- `callback` is a function. This is the event listener to register.
- `options` is a boolean or an object `{ capture?: boolean, passive?: boolean, once?: boolean }`. If this is a boolean, it's same meaning as `{ capture: options }`.
    - `capture` is the flag to register the event listener for capture phase.
    - `passive` is the flag to ignore `event.preventDefault()` method in the event listener.
    - `once` is the flag to remove the event listener automatically after the first call.

#### eventTarget.removeEventListener(type, callback, options)

Unregister an event listener.

- `type` is a string. This is the event name to unregister.
- `callback` is a function. This is the event listener to unregister.
- `options` is a boolean or an object `{ capture?: boolean }`. If this is a boolean, it's same meaning as `{ capture: options }`.
    - `capture` is the flag to register the event listener for capture phase.

#### eventTarget.dispatchEvent(event)

Dispatch an event.

- `event` is a [Event](https://dom.spec.whatwg.org/#event) object or an object `{ type: string, [key: string]: any }`. The latter is non-standard but useful. In both cases, listeners receive the event as implementing [Event](https://dom.spec.whatwg.org/#event) interface.

### defineEventAttribute(proto, type)

Define an event attribute (e.g. `onclick`) to `proto`. This is non-standard.

- `proto` is an object (assuming it's a prototype object). This function defines a getter/setter pair for the event attribute.
- `type` is a string. This is the event name to define.

For example:

```js
class AbortSignal extends EventTarget {
    constructor() {
        this.aborted = false
    }
}
// Define `onabort` property.
defineEventAttribute(AbortSignal.prototype, "abort")
```

### EventTarget(types)

Define a custom `EventTarget` class with event attributes. This is non-standard.

- `types` is a string or an array of strings. This is the event name to define.

For example:

```js
// This has `onabort` property.
class AbortSignal extends EventTarget("abort") {
    constructor() {
        this.aborted = false
    }
}
```

## 📚 Examples

### ES2015 and later

> https://jsfiddle.net/636vea92/

```js
const {EventTarget, defineEventAttribute} = EventTargetShim

// Define a derived class.
class Foo extends EventTarget {
    // ...
}

// Define `foo.onhello` property.
defineEventAttribute(Foo.prototype, "hello")

// Register event listeners.
const foo = new Foo()
foo.addEventListener("hello", (e) => {
    console.log("hello", e)
})
foo.onhello = (e) => {
    console.log("onhello", e)
}

// Dispatching events
foo.dispatchEvent(new CustomEvent("hello", { detail: "detail" }))
```

### Typescript

Currently typescript does not support type mutation by method, therefore the previous example will **not work** without the following modifications:

#### Working example #1
```ts
import { EventTarget, defineEventAttribute } from "event-target-shim";

// Define a derived class.
class Foo extends EventTarget<"onhello"> {
    // ...
}

// Define `foo.onhello` property.
defineEventAttribute(Foo.prototype, "hello")

// Register event listeners.
const foo = new Foo()
foo.addEventListener("hello", (e) => {
    console.log("hello", e)
})
foo.onhello = (e) => {
    console.log("onhello", e)
}

// Dispatching events
foo.dispatchEvent(new CustomEvent("hello", { detail: "detail" }))
```

In the future, if typescript adds support to string literal mutation (joining, slicing and etc.), it should be possible to automatically infer `onhello` type from `hello`. However, until then you are stuck with this:

#### Working example #2
```ts
import { EventTarget, defineEventAttribute } from "event-target-shim";

// Define a derived class.
class Foo extends EventTarget<"onhello">("hello") {
    // ...
}

// Register event listeners.
const foo = new Foo()
foo.addEventListener("hello", (e) => {
    console.log("hello", e)
})
foo.onhello = (e) => {
    console.log("onhello", e)
}

// Dispatching events
foo.dispatchEvent(new CustomEvent("hello", { detail: "detail" }))
```

### ES5

> https://jsfiddle.net/522zc9de/

```js
// Define a derived class.
function Foo() {
    EventTarget.call(this)
}
Foo.prototype = Object.create(EventTarget.prototype, {
    constructor: { value: Foo, configurable: true, writable: true }
    // ...
})

// Define `foo.onhello` property.
defineEventAttribute(Foo.prototype, "hello")

// Register event listeners.
var foo = new Foo()
foo.addEventListener("hello", function(e) {
    console.log("hello", e)
})
foo.onhello = function(e) {
    console.log("onhello", e)
}

// Dispatching events
function isSupportEventConstrucor() { // IE does not support.
    try {
        new CusomEvent("hello")
        return true
    } catch (_err) {
        return false
    }
}
if (isSupportEventConstrucor()) {
    foo.dispatchEvent(new CustomEvent("hello", { detail: "detail" }))
} else {
    var e = document.createEvent("CustomEvent")
    e.initCustomEvent("hello", false, false, "detail")
    foo.dispatchEvent(e)
}
```

## 📰 Changelog

- See [GitHub releases](https://github.com/mysticatea/event-target-shim/releases).

## 🍻 Contributing

Contributing is welcome ❤️

Please use GitHub issues/PRs.

### Development tools

- `npm install` installs dependencies for development.
- `npm test` runs tests and measures code coverage.
- `npm run clean` removes temporary files of tests.
- `npm run coverage` opens code coverage of the previous test with your default browser.
- `npm run lint` runs ESLint.
- `npm run build` generates `dist` codes.
- `npm run watch` runs tests on each file change.
