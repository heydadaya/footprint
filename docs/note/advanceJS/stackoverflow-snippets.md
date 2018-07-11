#### 1.check a string contains a substring in JavaScript

1. (ES6) includes
```js
var string = "foo",
    substring = "oo";
string.includes(substring);
```
2. ES5 and older indexOf
```js
var string = "foo",
    substring = "oo";
string.indexOf(substring) !== -1;
// String.prototype.indexOf returns the position of the string in the other string. If not found, it will return -1.
```
3. search
```js
var string = "foo",
    expr = /oo/;
string.search(expr);
```
4. lodash includes—go to answer
```js
var string = "foo",
    substring = "oo";
_.includes(string, substring);
```
5. RegExp
```js
var string = "foo",
    expr = /oo/;  // no quotes here
expr.test(string);
```
6. Match
```js
var string = "foo",
    expr = /oo/;
string.match(expr);
```

[引用](https://stackoverflow.com/questions/1789945/how-to-check-whether-a-string-contains-a-substring-in-javascript)

#### 2.redirect to another webpage

`window.location.replace(...)` will best simulate an HTTP redirect.

window.location.replace(...) is better than using window.location.href, because replace() does not keep the originating page in the session history, meaning the user won't get stuck in a never-ending back-button fiasco.

```js
If you want to simulate someone clicking on a link, use  location.href

If you want to simulate an HTTP redirect, use location.replace
```

For example:

```js
// similar behavior as an HTTP redirect
window.location.replace("http://stackoverflow.com");

// similar behavior as clicking on a link
window.location.href = "http://stackoverflow.com";
```

[引用](https://stackoverflow.com/questions/503093/how-do-i-redirect-to-another-webpage)

#### 3.deep clone an object in JavaScript

```js
JSON.parse(JSON.stringify(obj))
```

to be the `fastest` way to deep clone an object (it beats out jQuery.extend with deep flag set true by 10-20%).

 In order to clone JavaScript objects:
```js
// Shallow copy
var newObject = jQuery.extend({}, oldObject);

// Deep copy
var newObject = jQuery.extend(true, {}, oldObject);
```

[引用](https://stackoverflow.com/questions/122102/what-is-the-most-efficient-way-to-deep-clone-an-object-in-javascript)

#### 4.return the response from an asynchronous call

**Synchronous**

Imagine you make a phone call to a friend and ask him to look something up for you. Although it might take a while, you wait on the phone and stare into space, until your friend gives you the answer that you needed.

The same is happening when you make a function call containing "normal" code:

```js
function findItem() {
    var item;
    while(item_not_found) {
        // search
    }
    return item;
}

var item = findItem();

// Do something with item
doSomethingElse();
```

Even though `findItem` might take a long time to execute, any code coming after `var item = findItem();` has to wait until the function returns the result.

**Asynchronous**

You call your friend again for the same reason. But this time you tell him that you are in a hurry and he should call you back on your mobile phone. You hang up, leave the house and do whatever you planned to do. Once your friend calls you back, you are dealing with the information he gave to you.

That's exactly what's happening when you do an Ajax request.

```js
findItem(function(item) {
    // Do something with item
});
doSomethingElse();
```

Instead of waiting for the response, the execution continues immediately and the statement after the Ajax call is executed. To get the response eventually, you provide a function to be called once the response was received, a `callback `(notice something? call back ?). Any statement coming after that call is executed before the callback is called.

**Embrace the asynchronous nature of JavaScript! **

>JavaScript runs in the UI thread of the browser and any long running process will lock the UI, making it unresponsive. Additionally, there is an upper limit on the execution time for JavaScript and the browser will ask the user whether to continue the execution or not.

In the following we will look at three different solutions that are all building on top of each other:

* **Promises with**  `async/await` (ES2017+, available in older browsers if you use a transpiler or regenerator)
* **Callbacks** (popular in node)
* **Promises with** `then()` (ES2015+, available in older browsers if you use one of the many promise libraries)

All three are available in current browsers, and node 7+.

**ES2017+: Promises with `async/await`**

The new ECMAScript version released in 2017 introduced syntax-level support for asynchronous functions. `With the help of async and await, you can write asynchronous in a "synchronous style".` Make no mistake though: The code is still asynchronous, but it's easier to read/understand.

`async/await` builds on top of promises: an `async` function always returns a promise. `await` "unwraps" a promise and either result in the value the promise was resolved with or throws an error if the promise was rejected.

**Important**: You can only use await inside an async function. That means that at the very top level, you still have to work directly with the promise.

Here is an example that builds on top of delay above:

```js
// Using 'superagent' which will return a promise.
var superagent = require('superagent')

// This is isn't declared as `async` because it already returns a promise
function delay() {
  // `delay` returns a promise
  return new Promise(function(resolve, reject) {
    // Only `delay` is able to resolve or reject the promise
    setTimeout(function() {
      resolve(42); // After 3 seconds, resolve the promise with value 42
    }, 3000);
  });
}


async function getAllBooks() {
  try {
    // GET a list of book IDs of the current user
    var bookIDs = await superagent.get('/user/books');
    // wait for a second (just for the sake of this example)
    await delay(1000);
    // GET information about each book
    return await superagent.get('/books/ids='+JSON.stringify(bookIDs));
  } catch(error) {
    // If any of the awaited promises was rejected, this catch block
    // would catch the rejection reason
    return null;
  }
}

// Async functions always return a promise
getAllBooks()
  .then(function(books) {
    console.log(books);
  });
```

** Let functions accept callbacks **

`A callback is simply a function passed to another function. That other function can call the function passed whenever it is ready. In the context of an asynchronous process, the callback will be called whenever the asynchronous process is done. Usually, the result is passed to the callback.`

In the example of the question, you can make foo accept a callback and use it as success callback. So this

```js
var result = foo();
// Code that depends on 'result'
```

becomes

```js
foo(function(result) {
    // Code that depends on 'result'
});
```

Here we defined the function "inline" but you can pass any function reference:
```js
function myCallback(result) {
    // Code that depends on 'result'
}

foo(myCallback);
```

`foo` itself is defined as follows:
```js
function foo(callback) {
    $.ajax({
        // ...
        success: callback
    });
}
```

`callback` will refer to the function we pass to `foo` when we call it and we simply pass it on to `success`. I.e. once the Ajax request is successful, `$.ajax` will call `callback` and pass the response to the callback (which can be referred to with `result`, since this is how we defined the callback).

You can also process the response before passing it to the callback:
```js
function foo(callback) {
    $.ajax({
        // ...
        success: function(response) {
            // For example, filter the response
            callback(filtered_response);
        }
    });
}
```
It's easier to write code using callbacks than it may seem. After all, `JavaScript in the browser is heavily event-driven (DOM events). Receiving the Ajax response is nothing else but an event.`
Difficulties could arise when you have to work with third-party code, but most problems can be solved by just thinking through the application flow.

** ES2015+: Promises with then()**

The Promise API is a new feature of ECMAScript 6 (ES2015), but it has good browser support already. There are also many libraries which implement the standard Promises API and provide additional methods to ease the use and composition of asynchronous functions (e.g. bluebird).

`Promises are containers for future values.` When the promise receives the value (it is resolved) or when it is cancelled (rejected), it notifies all of its "listeners" who want to access this value.

The advantage over plain callbacks is that they allow you to decouple your code and they are easier to compose.

Here is a simple example of using a promise:

```js
function delay() {
  // `delay` returns a promise
  return new Promise(function(resolve, reject) {
    // Only `delay` is able to resolve or reject the promise
    setTimeout(function() {
      resolve(42); // After 3 seconds, resolve the promise with value 42
    }, 3000);
  });
}

delay()
  .then(function(v) { // `delay` returns a promise
    console.log(v); // Log the value once it is resolved
  })
  .catch(function(v) {
    // Or do something else if it is rejected 
    // (it would not happen in this example, since `reject` is not called).
  });
```

Applied to our Ajax call we could use promises like this:
```js
function ajax(url) {
  return new Promise(function(resolve, reject) {
    var xhr = new XMLHttpRequest();
    xhr.onload = function() {
      resolve(this.responseText);
    };
    xhr.onerror = reject;
    xhr.open('GET', url);
    xhr.send();
  });
}

ajax("/echo/json")
  .then(function(result) {
    // Code depending on result
  })
  .catch(function() {
    // An error occurred
  });
```

 provide a great abstraction and separation of your code.

More information about promises: [HTML5 rocks - JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/)

[引用](https://stackoverflow.com/questions/14220321/how-do-i-return-the-response-from-an-asynchronous-call)