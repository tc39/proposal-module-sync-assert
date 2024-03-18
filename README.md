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

## Possible solutions

Adding a hint to the engine,
if the module evaluation is async or contains an async subgraph,
the engine should fail early (syntax error),
so the developer can fix it early.

### Directive

```js
"assert sync"
```

Adding a new directive to hint the engine.

### Other candidates

#### [Import assertions](https://github.com/tc39/proposal-import-attributes/)

> [!IMPORTANT]  
> Import assertions proposal has been renamed to import attributes, and their semantics are changed accordingly. Now it no longer fits the needs of this proposal.

```js
import 'x' assert { sync: true }
// or
import 'x' with { assertSync: true }
```

This solution is not as good as the directive one.
When a developer uses this feature,
they intend to assert the current module itself is synchronous,
but if we take this approach,
they will have to add the assertion to every `import` and `export` in the current module.

```js
import './a.js' assert { sync: true }
import './b.js' assert { sync: true }
export { x } from './a.js'
// oh! I forgot to add assert here, and it can turn async with no error!
```

This usually does not make sense and is error-prone.

Note there _is_ a valid use case to let some of the imports have sync assert and others do not,
this is when using [defer import](https://github.com/tc39/proposal-defer-import-eval).
A developer may be okay with an async module in the initial graph
but wants to ensure there is no async subgraph of the deferred module that gets evaluated initially.

```js
import './async-module.js'
import defer * as mod from './mod.js' assert { sync: true }
import './mod-2.js'

// No subgraph of mod.js should execute before this line.
mod.start()
```

#### Linter/bundler level bans

It is possible, but each tool needs to invent its convention to do this.
It also does not apply to developers that don't use a bundler/linter.
