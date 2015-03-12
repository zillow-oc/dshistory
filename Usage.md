dsHistory was designed to be extremely easy to use. All that is really required for one to implement dsHistory into their own project is some moderate JavaScript skills and a desire to improve accessibility throughout their site. To get started, follow the instructions below. If you're super-cool, you can just skip right to the [examples](Examples.md).

### Definitions ###

**window hash**: The window hash is the entire portion of the URI that comes after (or, in IE, includes) the # symbol. It's accessed by calling window.location.hash, but you won't have to worry about that when using dsHistory.

**history stack**: The history stack refers to all of the functions that are pushed onto an internal array in dsHistory whenever `addFunction` or `bindQueryVars` is called with a function representing the application state as the first argument.

**base state**: The initial state of your application is it's base state. If a user is visiting your app for the first time, the base state will be your app in whatever regard you deem it to be completely reset to the beginning. If the user is visiting a bookmark where the third tab is specified, your base state will have the third tab open.

**base function**: The first function you add to the history stack through dsHistory is your base function. The base function should contain everything necessary to reset your app back to the base state.



### Step 1 ###

First, you should make a roadmap of where it will make sense in your application to stack in the back button support. While this may initially seem like the easiest part, screwing this part up will make the rest of the work you put into this pretty much worthless. dsHistory works off of functions placed into a "history stack," and so each function that you add to the history stack needs to be written in such a way that it contains all of the JavaScript necessary to get your app back to a desired state. This usually doesn't mean that you'll have to rewrite any of your functions or anything, but it is something that you should keep in mind going forward.

### Step 2 ###

You'll need to include either dshistory.js or dshistory.compressed.js in your application.

```
<script src="/dshistory/dshistory.compressed.js"></script>
```

(As an aside, notice how there's no type="text/javascript" attribute? [It's not necessary](http://javascript.crockford.com/script.html).)

### Step 3 ###

At some point during the loading of your application, either during a `window.load` event, an [onDOMReady](http://www.google.com/search?q=ondomready) event, or an inline function on the page, you'll need to initialize the base state of your application in dsHistory. The base state is the state of your application when it is first opened and before the user has had a chance to click on any tabs, fire off any AJAX requests, etc. To initialize the state of the application, you'll need to make a function that resets your application back to this base state and then pass it to dsHistory as follows.

```
dsHistory.addFunction(functionToCall, functionScope, arbitraryObjectToBePassed);
```

The functionScope and arbitraryObjectToBePassed arguments are optional; the functionScope argument adjusts the scope of functionToCall, and the arbitraryObjectToBePassed argument ultimately passed to functionToCall. Your base function is the first function on the stack and is the last function to be called as a user is hitting the back button before going to the previous site.

You shouldn't ever need to change this function, but if for some reason you want to change what you consider your base state after the application is loaded, you can call `dsHistory.setFirstEvent` with the same arguments as above.

### Step 4 ###

From there, you need to add a function to the history stack at every point where you've planned out; again, each function should contain all of the logic necessary to get your app into the desired state. When you do add these functions to the history stack, they need to be added either before or during the function that actually takes your application into that state. In almost every case, the function that you add to the stack _and_ the function that adds _that_ function to the stack are the same exact function.

For example, let's say you've performed an AJAX request that returns a JSON object. Once the AJAX request has returned, let's pretend you have a function called `Results.load` that is executed in the `Results` scope and is passed that JSON object by whatever JavaScript library you have going on. If you want to have back button support for this request, your function will look something like this:

```
var Results = {
  timesResultsLoaded: 0,
  load: function(jsonResultsObject, historyObject) {
    if (!historyObject.calledFromHistory) {
      dsHistory.addFunction(Results.load, this, jsonResultsObject);
    }
    
    this.timesResultsLoaded++;
    ...
    //do processing
    ...
  }
}
```

In the example, you'll notice that an argument named `historyObject` was slipped in there. What is that you might ask? The `historyObject` argument, passed as the last argument to any function that is called from the dsHistory function stack, is a key ingredient to getting dsHistory to work at all. It allows the function that is called to determine whether or not to add itself back onto the stack. In every single case, you do _not_ want to add the function back onto the stack if it was called _from_ the stack; otherwise, you'll have issues. If `historyObject` isn't undefined (i.e., the function was called from the history stack), it will have two properties; `calledFromHistory` and `direction`. `calledFromHistory` is just a boolean value to indicate whether the function was, well, called from the history stack. Since dsHistory catches both the back _and_ forward buttons, `direction` is a string value that will have a value of either 'back' or 'forward'. In most cases, it won't matter to you if the function was called while the user was hitting the back button or the forward button.

### Bookmarking Support ###

dsHistory was set up from the get-go to allow you to handle the back and forward buttons with or without changing the window hash. In other words, if you want certain states of your app to be bookmark-able while other states don't change that bookmark, you can. To do so, you'll want to do something like this:

```
dsHistory.setQueryVar('Address', '20162 Tomas'); // key, then value
dsHistory.setQueryVar('City and State', 'Rancho Santa Margarita, CA'); // key and value can be anything you want, no limitations
dsHistory.removeQueryVar('Address'); // decided to remove the address key? no problemo. just remove it here, and both it and it's value will be stripped from the hash
dsHistory.bindQueryVars(functionToCall, functionScope, arbitraryObjectToBePassed); // as before, functionScope and arbitraryObjectToBePassed are optional
```

When `dsHistory.bindQueryVars` is called in this circumstance, it will do three things. It will change the window hash so that all of the keys / values you set up are encoded in the window hash for future use, it will add all of the keys and values to the `dsHistory.QueryVars` object so that they can be referenced as a hash, and it will add `functionToCall` as another point in the browser's history. You can continue doing this as many times as you want, and you can mix and match `dsHistory.bindQueryVars` and `dsHistory.addFunction` as you see fit.

Let's pretend the code in the above example has been executed. The window hash would look like this: 'City%20and%20State=Rancho%20Santa%20Margarita%2C%20CA'. Thankfully, it doesn't matter to you what it looks like; all you need to know is that you can look at `dsHistory.QueryVars['City and State']` and the value will be 'Rancho Santa Margarita, CA'. Now, let's pretend that the user has bookmarked the page in that state. Later, when the user comes back to that page or IM's it to their friend, dsHistory will automatically pick apart the window hash and put the keys and values back into the `dsHistory.QueryVars` object in the same way we just accessed them.

You can and should handle everything in the `dsHistory.QueryVars` object when the page loads and before you pushed the first function onto your history stack all the way back in Step 1, as the object will likely change what the first function is. For example, if you check `dsHistory.QueryVars['CurrentTab']` when the app loads and the value is set to 'Tab3', you'll want to call your tab changing function and pass it whatever is needed to tell it to open 'Tab3'. If the tab changing function has code in it to push functions onto the history stack, you're golden as your base function will be set up for you automatically by that function. Otherwise, if `dsHistory.QueryVars['CurrentTab']` is `undefined` and you don't have to call any functions to set up the default tab, you'll have to add the base function yourself as in Step 1.

### Examples ###

Now that we're through with that, take a look at some [examples](Examples.md).