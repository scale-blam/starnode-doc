# Limitations of StarNode

## Code features preventing remote execution

In the following, all restrictions described to affect objects also affect functions (because a function is an object).
When a function is executed with a call to `warp.callAsPromise(fn, ...args)`, the function, its arguments, and all the data it uses is transported.
This data includes all variables, functions or objects accessed by the function that are defined outside of the function.
Anything defined inside the function does not need to be transported, thus many of the following restrictions do not apply for these.

### Symbols
Symbols cannot be transported. This affects plain symbols as well as any object containing a symbol or having a symbolic property.

Simple symbols:
``` javascript
function f(arg, cb) {
    cb();
}

let symbol = Symbol();

warp.callAsPromise(f, symbol);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Invalid argument: Symbol not supported

Objects with symbolic properties:
``` javascript
let o = {};
o[symbol] = 0;

warp.callAsPromise(f, o);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Invalid argument: object with symbolic property unsupported

### Custom properties in function object
Functions with user-defined properties cannot be transported.

``` javascript
function ff() {
}

ff.customProperty = 1;

warp.callAsPromise(f, ff);
```

The returned `Promise` is rejected with an `Error`. The error message is:
>  Invalid argument: function with custom properties unsupported. List of properties: length,name,prototype,customProperty

### Getter / Setter properties
Any object with a getter or a setter cannot be transported.

``` javascript
let o = {
    get getter() {
        return 0;
    },
};

warp.callAsPromise(f, o);
```

The returned `Promise` is rejected with an `Error`. The error message is:
>  Cannot serialize: property with a getter or a setter unsupported

### Functions
Functions can be transported back only if there are coming from the main flow and are kept unmodified.

``` javascript
function ff() {
}

warp.callAsPromise(f, ff);
```

Can run in parallel.

``` javascript
function f() {
    function ff() {
    }
    return ff;
}

warp.callAsPromise(f);
```

\'ff\' cannot be transported back to the main flow because it was defined in a parallel flow.

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot diff: function 'ff' is not instrumented

### Object methods
Objects initialized with method definitions cannot be transported, and these methods cannot be executed with a `callAsPromise`.

``` javascript
let o = {
    method() {},
};

warp.callAsPromise(f, o);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot serialize: function 'method' is not instrumented

Object properties that are functions are supported.

``` javascript
let o = {
    m: function ff() {},
};

warp.callAsPromise(f, o);
```

Can run in parallel.

### Function with outer scoped variable accesses
A function accessing external variables cannot be transported, except for the function executed by the `callAsPromise`.

``` javascript

let outer = 1;
function f() {
    return outer;
}

warp.callAsPromise(f);
```

Can run in parallel.

``` javascript
let outer = 1;
function f() {
    return outer;
}

function ff() {
    return f();
}

warp.callAsPromise(ff);
```

The returned `Promise` is rejected with an `Error`. The error message is:
>  Cannot serialize: function 'f' uses scoped variables

### require

A function using `require` can only be transported or executed elsewhere in the expert mode.	
If not in expert mode, any access to `require` will cause a failure of the remote execution.	
Use of `require` are limited to expert mode because their shared resources, natives and	
modifications to the global object are not shared between parallel executions, in particular	
exported objects will be created several times even for the same version of the required module.	

 ``` javascript	
function f() {
    require('./lib/required');
}

warp.callAsPromise(f);	
```	

The returned `Promise` is rejected with an `Error`. The error message is:
>  Cannot diff: function 'StarnodeServiceError' is not instrumented

``` javascript	
let required = require('../lib/required');

function f() {
    required();
}

warp.callAsPromise(f);	
```	

Can run in parallel.

### Classes
Classes cannot be transported and class methods cannot be transported or executed with a `callAsPromise`.

``` javascript
class A {}

function f() {
    new A();
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot serialize: function 'A' is not instrumented

``` javascript
class A {
    do() {
    }
}
let o = new A();

warp.callAsPromise(o.do);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Function 'do' is not instrumented

Furthermore, class instances cannot be transported.

``` javascript
function f(arg) {
}

class A {}

warp.callAsPromise(f, new A());
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot serialize: function 'A' is not instrumented

It is possible to create classes and instances of classes during a remote execution, but only as
long as they are not returned by the function (either as a callback argument, as an object property,
in a variable or any other way).

``` javascript
function f() {
    class A {
    }
    return new A();
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot diff: function 'A' is not instrumented

``` javascript
function f() {
    class A {
        get() {
            return 1;
        }
    }
    let o = new A();
    o.get();
}

warp.callAsPromise(f);
```

Can run in parallel.

### Recursive functions
The only recursive functions that can be executed during a `callAsPromise` are [named function expressions](https://developer.mozilla.org/en-US/docs/web/JavaScript/Reference/Operators/function#Named_function_expression) and they can only be transported as parameter of `callAsPromise`.
All other type of recursive function cannot be transported.

``` javascript
function recursive(value) {
    return value === 0 ? 1 : value * recursive(value -1);
}

warp.callAsPromise(recursive, 3);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot serialize: function 'recursive' uses scoped variables

``` javascript
let f = function recursive(value, done) {
    return value === 0 ? 1 : value * recursive(value - 1);
};

warp.callAsPromise(f, 3);
```

Can run in parallel.

### Unknown usage
An unknown usage is an access to a variable for which we cannot find a definition. It can happen in the following cases:

* Undefined variable

``` javascript
function f() {
    try {
        notDeclared();
    } catch (v) {
    }
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Function 'f' is not instrumented

* In non-strict, variable declared in `eval` or context added by `with` are not supported and thus
can be reported as unknown usages.

``` javascript
eval(`var evaled = 1;`);

function f() {
    return evaled;
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Function 'f' is not instrumented

We do not support unknown usages, thus any function having `unknown` usage cannot be transported, and cannot be executable in an `callAsPromise`.
There is an exception for a selection of implicit global access, described in [Implicit global](#implicit-global)

### Global properties accesses

#### Explicit global
Explicit access to global properties are forbidden during remote execution.

``` javascript
function f() {
    global.console.log('message');
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot serialize: usage of global forbidden

#### <a name="implicit-global"></a> Implicit global
Normally implicit access to global properties are allowed for a restricted set of properties.
This is the currently supported implicit globals:
* console
* undefined
In expert mode, on the other hand, all implicit accesses to global properties are allowed, it is up to
the developer to ensure that those uses won't cause any problem.

``` javascript
function f() {
    console.log(undefined);
}

warp.callAsPromise(f);
```

Can run in parallel.

``` javascript
function f() {
    clearImmediate();
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cause: Function 'f' is not instrumented

### Native functions
Native functions cannot be transported.

``` javascript
let toString = Object.prototype.toString;
function f() {
    toString();
}

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Cannot serialize: native function unsupported

### Bound functions
Bound functions cannot be transported.

_Bound functions appear as native when printed, so they are treated as native ones_

``` javascript
let f = function ff() {
}.bind({});

warp.callAsPromise(f);
```

The returned `Promise` is rejected with an `Error`. The error message is:
> Function 'bound ff' is not instrumented

### Strict code only
Starnode runs in strict mode, and is designed to execute strict code. It can however execute non-strict code, with only a slight risk of incorrectness when using language features causing a different behaviour between strict and non-strict code.
Particularly, any function accessing a scope modified by an `eval` (for example variable declared
by `eval`) is unsupported, any function declared by `eval` is unsupported. Any
use of `with` is unsupported and Starnode handling of `with` may change between versions.

## Modification of runtime behaviour

### Functions properties
We add symbolic properties to the functions we instrument.
These properties are symbolic and non-enumerable, so they shouldn't be visible in most common cases
(for example during `forEach` or `for ... in`) but they can still be listed, for example with `Object.getOwnPropertySymbols()`.
This could modify the application's behavior in some cases.
These properties are non-(enumerable, writable, configurable), so the end user shouldn't be able to tamper with them.

### Functions names
Functions have a name property that can be accessed with `function.name` and is displayed when the function is printed.
This name can either be explicit (for function declarations and named function expressions), or implicitly drawn from the context.
The implicit naming is sometimes broken by the instrumentation of the code (meaning that some functions will have their name replaced
with an empty string when executing on the service node) and is always broken during a remote execution.
Explicit names are always preserved, both in local and remote executions.

## Exit
Starnode does not support automatic exit after execution of last task yet. This means you'll have to
explicitly call `process.exit` at the end of your programs.
