<!--
.. title: JavaScript promises deep-dive
.. slug: javascript-promises-deep-dive
.. date: 2024-01-31 14:09:07 UTC+05:30
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

## Agenda

* Promise syntax
* Creating a promise
* Resolver function
* Handling a promise's resolved and rejected values

## Promise

Promise is a JavaScript provided construct to support asynchronous programming.

MDN docs define a Promise as:

<code>A Promise is a proxy for a value not necessarily known when the promise is created. It allows you to associate handlers with an asynchronous action's eventual success value or failure reason.</code>

## Creating a promise

Let's consider you have a JavaScript file called `script.js`. This could be used from `index.html`.

Let's create a promise in `script.js`.

    let promise = new Promise()

Load `index.html`. You would notice the following error on the console.

    Uncaught TypeError: Promise resolver undefined is not a function

JavaScript engine expects a function to be passed as the first argument to `Promise` constructor. This first argument is treated as `Promise resolver`.
As we haven't passed any argument, hence JavaScript finds the first argument as `undefined`. That's why it's complaining `undefined is not a function`.

Let's create a function and pass that as the first argument during promise creation.

    const promiseResolver = function () {
      console.log("Promise resolver")
    }
    let promise = new Promise(promiseResolver)

Let's load index.html again. You would notice the following logged on the console.

    Promise resolver

Thus we were able to successfully create a promise.

This also illustrates that the resolver function is immediately invoked on promise creation.

We used a function expression to create the resolver function. In other articles or code, you might see the arrow syntax being used. The equivalent code using arrow syntax would be:

    let promise = new Promise(() => {
      console.log("Promise resolver")
    })

Arrow syntax is a compact alternative to a traditional function expressions.

A promise is either eventually fulfilled or rejected. The key term here is **eventually**. Whether the promise would be fulfilled or rejected isn't known at promise creation time, this might happen after some elapsed time.

## resolve

As mentioned earlier, Promises are heavily used during asynchronous operations. As the name asynchronous suggests, the result of any asynchronous operation isn't known immediately and would only be known eventually.

Promises are often used to wrap an asynchronous operation. When the asynchronous operation completes, the promise should have ability to return the result of asynchronous operation.

That's where `resolve` kicks-in. It allows us to return the result of asynchronous operation from a promise.

`resolve` is a function passed to the resolver function by the JavaScript engine during promise creation time. It is passed as the first argument to the resolver function.

Hence, let's modify the resolver function to accept this first argument. The updated code would look like:

    const promiseResolver = function (resolve) {
      console.log("Promise resolver")
    }
    let promise = new Promise(promiseResolver)

We as engineers don't need to perform any extra step to pass `resolve`. The JavaScript engine passes `resolve` to `promiseResolver` when the promise is being constructed.

Let's assume we are doing an asynchronous operation inside the promise. This generates some string value.
To simplify things, we will hardcode a string value instead of performing an asynchronous operation.

    const promiseResolver = function (resolve) {
      const asynchronousOperationResult = "hello world"
      console.log("Promise resolver")
      resolve(asynchronousOperationResult)
    }
    let promise = new Promise(promiseResolver)

Reload the page, you would still see only `Promise resolver` logged on the console.

As mentioned, we don't have to write anything for `resolve`. It's a function passed by the engine to our code, we only need to invoke it.
We could very well use another variable to get hold of this function. Let's call it `fulfill` instead of `resolve`.

    const promiseResolver = function (fulfill) {
      const asynchronousOperationResult = "hello world"
      console.log("Promise resolver")
      fulfill(asynchronousOperationResult)
    }
    let promise = new Promise(promiseResolver)

The equivalent with arrow syntax would be:

    let promise = new Promise((fulfill) => {
      const asynchronousOperationResult = "hello world"
      console.log("Promise resolver")
      fulfill(asynchronousOperationResult)
    })

When `resolve` or `fulfill` gets invoked from inside the promise, the promise transitions from state `pending` to `fulfilled`. Essentially we are fulfilling the promise with a value.

## then

When an asynchronous operation is happening, it would be wrapped in a Promise. The operation eventually succeeds and the promise fulfills that value using `resolve`.

However we need a mechanism to capture the fulfilled value.

That's where `then` kicks-in.

`then` is used to associate a **handler** that can act on the fulfilled value. Let's create a handler first.

    const fulfillHandler = function (data) {
      console.log(data)
    }

Let's associate this handler with the promise using promise method `then`.

    promise.then(fulfillHandler)

Load the page. You would see 'hello world' logged on the console too.

The equivalent arrow syntax would be:

    let promise = new Promise((fulfill) => {
      const asynchronousOperationResult = "hello world"
      console.log("Promise resolver")
      fulfill(asynchronousOperationResult)
    })

    promise.then((data) => {
      console.log(data)
    })

## reject

At times, the asynchronouse operation would not succeed. The promise needs ability to convey the same as well. That's where `reject` kicks-in.

`reject` is a function passed to the resolver function by the JavaScript engine during promise creation time. It is passed as the second argument to the resolver function.

Hence, let's modify the resolver function to accept this second argument. The updated code would look like:

    const promiseResolver = function (fulfill, reject) {
      const asynchronousOperationResult = "hello world"
      console.log("Promise resolver")
      fulfill(asynchronousOperationResult)
    }
    let promise = new Promise(promiseResolver)

We as engineers don't need to perform any extra step to pass `reject`. The JavaScript engine passes `reject` to `promiseResolver` when the promise is being constructed.

Let's assume the asynchronous operation is resulting in some integer. Our desired behaviour is to fulfill the promise with an even number. If the generated number is odd, then we treat it as an unsuccessful operation. Hence instead of fulfilling we want to reject the promise.

Let's modify script.js to look like:

    const promiseResolver = function (fulfill, reject) {
      console.log("Promise resolver")
      const asynchronousOperationResult = Math.floor(Math.random() * 10)
      if (asynchronousOperationResult % 2 == 0) {
        fulfill(asynchronousOperationResult)
      }
      else {
        reject("Odd number")
      }
    }
    let promise = new Promise(promiseResolver)

    const fulfillHandler = function (data) {
      console.log(data)
    }

    promise.then(fulfillHandler)

Refresh the page several times.

When even numbers are generated, you would see the even number being logged.

When an odd number gets generated, you would see the following uncaught exception on the console.

  Uncaught (in promise) Odd number

It is conventional to refer to this function as `reject`. Similar to `resolve/fulfill` case, we can refer to it with any other name.
We can equally well refer to it as `unfulfill` instead of `reject`.

Similar to how the code captured the fullfilled value of a promise, our code should have ability to capture the rejected reason too.

`then()` accepts a handler which can be used to capture the rejected reason. This handler needs to be provided as second argument to `then`.

    const fulfillHandler = function (data) {
      console.log(data)
    }

    const rejectHandler = function (reason) {
      console.log(reason)
    }

    promise.then(fulfillHandler, rejectHandler)

The complete code would look like:

    const promiseResolver = function (fulfill, unfulfill) {
      console.log("Promise resolver")
      const asynchronousOperationResult = Math.floor(Math.random() * 10)
      if (asynchronousOperationResult % 2 == 0) {
        fulfill(asynchronousOperationResult)
      }
      else {
        unfulfill("Odd number")
      }
    }
    let promise = new Promise(promiseResolver)

    const fulfillHandler = function (data) {
      console.log(data)
    }

    const rejectHandler = function (reason) {
      console.log(reason)
    }

    promise.then(fulfillHandler, rejectHandler)

Refresh the page several times.

The output would either log the generated even number or would log the message `Odd number`.

## Returning promise from a function

In the previous examples, we were creating our promise in the global scope.

In all likelihood, your promises would be defined inside a function and returned from there.

Let's declare our promise inside a function and return it on function invocation.

    const oddEven = function () {

      const promiseResolver = function (fulfill, unfulfill) {
        const asynchronousOperationResult = Math.floor(Math.random() * 10)
        if (asynchronousOperationResult % 2 == 0) {
          fulfill(asynchronousOperationResult)
        }
        else {
          unfulfill("Odd number")
        }
      }
      let promise = new Promise(promiseResolver)
      return promise
    }

We can then invoke oddEven() which will return a promise.

    let promise = oddEven()

We can then define the handlers and register it with the promise using then().

    const fulfillHandler = function (data) {
      console.log(data)
    }

    const rejectHandler = function (reason) {
      console.log(reason)
    }

    promise.then(fulfillHandler, rejectHandler)

## Rejecting with an error

In the previous example, we were only rejecting with a string message.

A more conventional way would be to reject with an instance of error. That allows us to add some contextual information too.

Thus reject part should be modified to return an `Error` instance.

    const oddEven = function () {

      const promiseResolver = function (fulfill, unfulfill) {
        const asynchronousOperationResult = Math.floor(Math.random() * 10)
        if (asynchronousOperationResult % 2 == 0) {
          fulfill(asynchronousOperationResult)
        }
        else {
          const error = new Error("Odd number")
          error.number = asynchronousOperationResult
          unfulfill(error)
        }
      }
      let promise = new Promise(promiseResolver)
      return promise
    }

And reject handler should be modified to deal with an error instance.

    const rejectHandler = function (error) {
      console.log(error.message)
      console.log(error.number)
    }

## fetch

Until now we have been cooking up our own examples to understand Promise, promise resolution, resolve, reject and then.

It would make more sense if we could relate it with a real-world example.

JavaScript fetch API is promise based. Let's explore that.

Let's assume we are building an application which displays emojis. API endpoint https://emojihub.yurace.pro/api provides random emojis.

fetch() can be used to retrieve data from servers.
fetch() returns a promise. This is very similar to how oddEven() returned a promise.

Let's modify script.js to pull a random emoji.

    let promise = fetch('https://emojihub.yurace.pro/api/random')
    promise.then((response) => {
      console.log(response.status)
    })

fetch() returns a promise. We have used `then` with the promise and passed it one argument. Thus we have passed it the handler that would be called when the promise is fulfilled.

In the handler, we are logging the response status code.

We haven't added the second argument to `then()`, thus a rejected promise wouldn't be gracefully handled by our code.

Let's create the reject handler too and add it.

    let promise = fetch('http://raajakshar.com')
    const fulfillHandler = (response) => {
      console.log(response.status)
    }
    const rejectHandler = (exception) => {
      console.log(exception.message)
    }
    promise.then(fulfillHandler, rejectHandler)

The domain raajakshar.com isn't configured. Hence, no connection can be established, and in such cases the promise is rejected.
We have added a reject handler.

Load the page, and you would see the message 'Failed to fetch'.

## Conclusion

This article covered the following:

1. Creating a promise
2. Passing a resolver function to promise
3. Fulfilling a promise by invoking resolve()
4. Rejecting a promise by invoking reject()
5. Creating resolve handler and reject handler.
6. Registering the handlers using promise's then().

In a follow-up article, we will discuss chaining of promises and async/await

<hr/>
### Shameless Plug

If you work extensively with JavaScript, you should attempt our [quiz](http://javascript.softwarequizzes.com) to test your JavaScript knowledge.
