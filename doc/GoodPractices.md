# Good pratices

##  <a name="parallel-variable-modifications">Avoid modifying values accessed in parallel context

### Problem

Even if it's possible, modifying values accessed in parallel is generally not a good idea because it
can easily lead to unpredictable behavior.

``` javascript
const warp = require('warp');

let v = 4;
function compute(num) {
  // compute increment from input
  let inc = num;
  v += inc;
  return;
}

let promises = []
promises.push(warp.callAsPromise(compute, 3));
promises.push(warp.callAsPromise(compute, 4));

await Promise.all(promises);

console.log(`Value: ${v}`);
```

There, you don't know in which order the additions will be made, but it is guaranteed that each
_compute_ sees the initial value of `v` as `4`. This means you don't know what the final value of `v`
will be, you just know it will be either `7` or `8`.

### How to solve the problem

To obtain the correct behavior in this example, you'll need to apply the changes to the shared
variable in the main thread by using the `.then` of the promise.

``` javascript
const warp = require('warp');

let v = 4;
function compute(num) {
  // compute increment from input
  let inc = num;
  return inc;
}

let promises = []
promises.push(warp.callAsPromise(compute, 3).then((inc) => v += inc));
promises.push(warp.callAsPromise(compute, 4).then((inc) => v += inc));

await Promise.all(promises);

console.log(`Value: ${v}`);
```

This way it'll end up with `11` as the final value.
