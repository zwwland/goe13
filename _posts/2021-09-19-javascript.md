---
key: post-codes
---

### yum 安装nodejs
```sh
curl -sL https://rpm.nodesource.com/setup_14.x | bash -
```

### apt 安装nodejs
```sh
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```

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

