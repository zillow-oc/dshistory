### Properties ###

  * **deferProcessing**: It seems that updating / setting the window hash can sometimes make IE really chug when the page has tons of elements on it. My best guess is that IE looks for an `a` tag on the page that has a `name` attribute with the value of the hash, and for some reason has a hard time with it. By setting `dsHistory.deferProcessing = true`, the window hash will actually be set in a separate function which is deferred to execute when the calling function is finished. While this doesn't actually speed anything up, it can make things appear faster since a UI update can take place in between the execution of the two functions.

  * **QueryElements**: The `QueryElements` object holds all of the keys and values reflected in the window hash.

### Methods ###

  * **addFunction**: Adds a new function to the internal history stack. Accepts three arguments.
    1. `functionToAdd`: The function to add to the stack.
    1. `scope`: The scope that `functionToAdd` will be called in.
    1. `objectArg`: An arbitrary object to pass to `functionToAdd`. If you have more than one object that you want to pass, make then properties of `objectArg`.

  * **bindQueryVars**: Updates the window hash with the keys and values set with `setQueryVar` and adds a new function to the internal history stack. Accepts the same exact arguments as addFunction.

  * **initialize**: Exists for backward compatibility at the moment, but not deprecated. Ultimately, it will likely accept an object literal containing properties to initialize dsHistory for other browsers.

  * **removeQueryVar**: Queues a key and it's value to be removed from the window hash, but doesn't actually update it. Accepts one argument.
    1. `key`: The key (and value of that key) to ultimately remove from the window hash.

  * **setFirstEvent**: Once the base function is added, it can't be removed or changed without calling this function. You shouldn't need to change the base function, but if you do, you can use this method. Accepts the same exact arguments as addFunction.

  * **setQueryVar**: Queues a key and value pair to be added to the window hash, but doesn't actually update it. Accepts two arguments:
    1. `key`: The key of the hash item.
    1. `value`: The value of the hash item.