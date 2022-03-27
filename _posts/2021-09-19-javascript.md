---
key: post-codes
---


### method_missing 实现
```js
class Base {
  constructor () {
    return new Proxy(this, {
      get (target, property) {
        if (Reflect.has(target, property)) {
          return Reflect.get(target, property)
        } else {
          return function () {
            if (target.method_missing) {
              return target.method_missing(property, ...arguments)
            } else {
              throw new ReferenceError('Property "' + property + '" does not exist.')
            }
          }
        }
      }
    })
  }
}
```

