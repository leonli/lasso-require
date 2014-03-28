raptor-optimizer-require
========================

Plugin for the [RaptorJS Optimizer](https://github.com/raptorjs3/raptor-optimizer) that adds support for transporting Node.js-style modules to the browser.

# Installation

This plugin is included as part of the `raptor-optimizer` module so it is not necessary to use `npm install` to add the module to your project. However, if you want to use a specific version of the `raptor-optimizer-require` plugin then you can install it using the following command:

```
npm install raptor-optimizer-require --save
```

# Usage

This plugin is enabled by default, but if you want to provide your own configuration then you can do that using code similar to the following:

```javascript
require('raptor-optimizer').configure({
    plugins: {
        'raptor-optimizer-require': {
            transforms: [ // Browserify compatible transforms
                'deamdify'
            ] // See https://github.com/substack/node-browserify/wiki/list-of-transforms
        }
    }
})
```

The `raptor-optimizer-require` plugin introduces two new dependency types that you can use to target Node.js modules for the browser. There usage is shown in the following `optimizer.json` file:

```json
{
    "dependencies": [
        "require: jquery",
        "require-run: ./main"
    ]
}
```


These new dependency types are described in more detail below.

# Dependency Types

## require

The `require` dependency type will wrap a Node.js module for delivery to the browser and allow it to be required from another module. For example:

__Input modules:__

_foo.js:_
```javascript
exports.add = function(a, b) {
    return a + b;
}
```

_bar.js:_
```javascript
var foo = require('./foo'); 

exports.sayHello = function() {
    console.log('Hello World! 2+2=' + foo.add(2, 2));
};
```

__Output Bundles:__

After running the following command:

```bash
raptor-optimizer require:./foo require:./bar --name test
```

The output written to `static/test.js` will be the following:

```javascript
$rmod.def("/foo", function(require, exports, module, __filename, __dirname) { exports.add = function(a, b) {
    return a + b;
} });
$rmod.def("/bar", function(require, exports, module, __filename, __dirname) { var foo = require('./foo'); 

exports.sayHello = function() {
    console.log('Hello World! 2+2=' + foo.add(2, 2));
}; });
```

__NOTE:__ `$rmod` is a global introduced by the [client-side Node.js module loader](https://github.com/raptorjs3/raptor-modules/blob/master/client/lib/raptor-modules-client.js). It should never be used directly!. The code that declares `$rmod` is not shown in the output above for brevity.

## require-run

In the previous examples, neither the `foo.js` or `bar.js` module will actually run. The `require-run` dependency type should be used to make a module self-executing. This is the equivalent of the entry point for your application when loaded in the browser.

Continuing with the previous example:

__Input modules:__

_foo.js_
(see above)

_bar.js_
(see above)

_main.js:_
```javascript
require('./bar').sayHello();
```

__Output Bundles:__

After running the following command:

```bash
raptor-optimizer require-run:./main --name test
```

Alternatively:
```bash
raptor-optimizer --main main.js --name test
```

The output written to `static/test.js` will be the following:

```javascript
$rmod.def("/foo", function(require, exports, module, __filename, __dirname) { exports.add = function(a, b) {
    return a + b;
} });

$rmod.def("/bar", function(require, exports, module, __filename, __dirname) { var foo = require('./foo'); 

exports.sayHello = function() {
    console.log('Hello World! 2+2=' + foo.add(2, 2));
}; });

$rmod.run("/main", function(require, exports, module, __filename, __dirname) { require('./bar').sayHello(); });
```

