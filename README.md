# Module sync assert

## Problem to solve

Some code must be synchronous.
If the module accidentally becomes async (by having a [top-level await](https://github.com/tc39/proposal-top-level-await) or the [old semantics of WebAssembly ESM integration](https://github.com/WebAssembly/esm-integration/tree/26e6faa9762b604e8eea399be1e8a1c3bda256ab/proposals/esm-integration#why-does-this-proposal-depend-on-top-level-await) in the subgraph) the code might break in a way that hard to debug.

### Example: Service Worker

```js
import './some-module.js'
addEventListener('fetch', () => {})
```

> [!NOTE]
> If you try to use the native implementation of the ES Module in Service Worker,
> it will throw a TypeError "**Top-level await is disallowed in service workers.**".
> 
> Let's assume the service worker is bundled via a bundler and transformed into a non-ES module format.

If `some-module.js` becomes async,
then the `addEventListener` no longer works:

> [!WARNING]
> Event handler of 'fetch' event must be added on the initial evaluation of worker script.

### Example: Polyfill

The polyfill code must run synchronously, otherwise,
the application might be broken in old browsers.

### Example: API stability

Adding TLA is a breaking change,
the library author may want to add a test that their library won't accentually become async due to dependency changes.

## Solution

Adding a hint to the engine,
if the module evaluation is async or contains an async subgraph,
the engine should fail early (syntax error),
so the developer can fix it early.

### Directive

```js
"assert sync"
```

Adding a new directive to hint the engine.

## Other solutions: Linter/bundler level bans

It is possible, but each tool needs to invent its convention to do this.
It also does not apply to developers that don't use a bundler/linter.
