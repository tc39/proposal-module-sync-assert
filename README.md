# Module sync assert

## Problem to solve

One of the module in the dependency graph becomes async and introduce bugs or breaking change to the current module.

### Chrome extension, a real world example

```js
import './some-module.js'
chrome.runtime.onInstalled.addListener(function () {
    // open welcome page
})
```

The listener is only triggered at the end of the first event loop, so if the `./some-module.js` becomes async, it will
miss the event and introduce a new bug.

## Possible solutions

It should be a link error to fail early so the developer can notice it immediately when a new async module comes in.

### Linter/bundler level bans

It's possible, but each tool need to invent their own convension.

### Import assertions

```js
import 'x' assert { sync: true }
```

Import assertions has already renamed to import attributes and the keyword becomes `with` so it might becomes this:

```js
import 'x' with { assertSync: true }
```

This solution is not good because when a developer uses this feature, their intension is to let the whole module sync.
The following example does not make sense:

```js
// a must sync but b don't have to
import 'a' assert { sync: true }
import 'b'
```

If we take this approach, developers have to do this on all import and export statements.

### Directive

```js
"assert sync"
```

The problem of this approach is it looks like the committee may not want to add new directives.
