# Asynchronous with Redux Sagas

### Sophisticated Side Effect Flow Management and Testing

![Purple Smoke Header Image][sagalogo]

Building an app with React can get a bit confusing when data is being shared among components and different states lead to too much complexity and difficulty. Redux is a lightweight state management tool which can be used with any JavaScript framework or library to maintain a consistent and predictable state container. By keeping the state of an application in a single global store rather than at the component level, each individual component can access any state it needs at any time regardless of the shape of the component tree, as long as it is connected to the store via Redux.

The predictable nature of Redux comes from the immutable state which never changes, as well as pure function reducers. Because reducers are functional, a commonly used middleware to handle side effect logic and asynchronous calls is [redux-thunk][thunklink]. A Thunk allows action creators to return a function instead of an action.

While I will be assuming you have a basic understanding of React and Redux, this will be a guide into a different kind of Redux middleware for handling side effects: **Redux Sagas**

→ _Skip to the walkthrough with sample code [here][walkthrough]_

### Why Redux Saga?

So long as the same action is passed to the reducer, we can be sure the store will be updated in the same way every time. Sagas, similar to normal reducers, are functions which listen for dispatched actions, perform side effects, and return their own actions back to the normal reducer. Because Sagas intercept actions with side effects and handle them, Redux reducers remain pure.

Redux Saga utilizes ES6 [generator][generatorlink] functions for this. Generators allow for synchronously written asynchronous code. A generator will automatically pause — or yield — at each asynchronous call until it completes before continuing. This paradigm allows for much simpler and more readable code by centralizing asynchronous logic for more manageable and sophisticated async flows.

Saga generator functions remind me a bit of `async/await`, with some minor changes such as `yield` and `put()`. Some of the differences provide powerful benefits, such as `takeLatest()` ensuring that only the latest fetch call runs to completion despite having dispatched multiple simultaneous fetch actions. However, asynchronous calls which would normally be directly inside an action creator in a thunk will have a clear separation in Redux Sagas.

Beyond code organization and attributes, testing becomes _much_ easier. A Saga merely yields a description of what to call, thus saving the need for mocking data for every test.

Redux Saga becomes most useful when API or other asynchronous calls are being made with complex flows in which calls depend on the next.

**Pros**:
→ More readable code
→ Good for handling complex scenarios
→ Test cases become simple without necessity to mock the async behavior
**Cons**:
→ Brings in more complexity to the code
→ Additional dependency
→ A lot of concepts to learn
**Conclusion**:
→ Suited for complex async parts of the application that requires complex unit test cases

### A quick note on Thunks:

Given that Redux Saga seeks to orchestrate complex asynchronous operations with Redux, it is an alternative to Thunks. However, Sagas provide more functionality. Thunks work well for simple use cases, but may not be the best choice for more complicated scenarios.

Thunks add a layer of indirection for more flexibility, and pass dispatch functions to the function the action creator returns. This allows the component to be agnostic towards asking for a synchronous or asynchronous action.

**Pros**:
→ Simple code to maintain
**Cons**:
→ Struggles at handling complex scenarios
→ Async behavior needs mocking for test cases
**Conclusion**:
→ Suited for small, straight forward async parts of the application

### Generators

Denoted with an `*`, generators make use of the `yield` keyword to pause the function. While `async/await` can be transpiled into generators, the reverse cannot be done. Moreover, Sagas’ `takeLatest()` behavior and generator function cancellation are more attributes provided by Redux Saga.

When a generator function is invoked, it returns an iterator object. Each subsequent `next()` method call will execute the generator until the next yield statement and pause.

<script src="https://gist.github.com/irmerk/78b25ad6a4eb9408b741d9fa906e95ba.js"></script>

---

### <a href='#walk'></a> Walkthrough:

To guide through this concept, I will be referencing a codebase for the web app used by an open source software project I contribute to here:

[Accord Project][apsite] (AP)
[AP Github][apgithub]
[Template Studio repository][tsv2]

The project currently being built is a redesign of the _Template Studio_. Details are mostly unimportant, suffice it to say that the part I will be going through makes an API call to gather an array of templates, and displays them in a component. This redesign will consist of many interlocking React components, all housed in one app and controlled by the Redux store. Because this began complex and will only continue to be more-so, we chose to pursue Redux Saga to handle the complexity.

Unfortunately, as you may have experienced as well, there seems to be little out there as reference material. This is especially so when it comes to anything complicated.

This will be a guide to following the logic behind Redux Saga in _Template Studio_ for Accord Project. Hopefully this will prove to be a useful resource for you.

Setup
Common Redux Saga methods (_called Effects_):

**`fork`** → Performs a non-blocking operation on the function passed.

**`take`** → Pauses until action received.

**`race`** → Runs effects simultaneously, then cancels them all once one finishes.

**`call`** → Runs function. If it returns a promise, pauses the Saga until resolved.

**`put`** → Dispatches an action.

**`select`** → Runs a selector function to get data from the state.

**`takeLatest`** → Executes the operation, returns only the results of the last call.

**`takeEvery`** → Will return results for all the calls triggered.

---

The overall structure of the application’s flow of data will look akin to this:

![Redux Saga Flow Diagram][sagadiagram]

To begin, we set up the main render of the app, and applying a store to the `Provider` given by [react-redux][reduxlink] :

<script src="https://gist.github.com/irmerk/cc5600c96517f685dabb3d7c783b68e3.js"></script>

#### Store

Pulling in the `createSagaMiddleware` method from Redux Saga, we create `sagaMiddleware` and run it on our rootSaga, which we will see below. Moreover, we combine all of our reducers and include this in the store upon creation.

Similar to the reducers, Sagas will be registered with a rootSaga. Having the middleware use the rootSaga allows actions being dispatched to be successful.

<script src="https://gist.github.com/irmerk/e848a979469ff2176d2fef1a985e7499.js"></script>

#### Sagas

Sagas work in the background and `sagaMiddleware` controls them. Being generator functions, Sagas have control over every single step of the function. We yield objects to `sagaMiddleware` which tell it what to do with given arguments, which it will execute and resume upon completion, thus appearing to operate synchronously.

Sagas are broken down to the root, watchers, and workers. All other Sagas you write are consolidated into root.

##### → Root

All Sagas will be registered with a root Saga. Combined in an `all()` function, they are allowed to all start at the same time each time.

<script src="https://gist.github.com/irmerk/ed6ab3c583a501e6e1dd928677471737.js"></script>

##### → Watcher

Allowing the Saga to know when to start, this generator function watches for actions (_similar to reducers_) and calls worker Sagas to do an API call. This function is on `Line 62` below:

<script src="https://gist.github.com/irmerk/884913996eeaad0ff54cfd1aae90bf09.js"></script>

Similar to `takeLatest()`, `takeEvery()` allows multiple instances of Sagas to run simultaneously. These are both built on `take()`, which is synchronous.

##### → Worker

This Saga (`Lines 14`, `31`, and `46` above) will perform a side effect. Once data loads, the `put()` method is used to dispatch another action. This does not directly dispatch, but rather creates an effect description that tells Redux Saga to dispatch it. Because `put()` expects an action for an argument, it serves as an action creator. We modularized these actions out, though, as you will see below.

#### Reducer

Similar to actions, reducers are the same for Redux Saga. This is simply a function that takes state and action as arguments, and returns the next state of the app. While an action only describes what happened, a reducer describes _how the application’s state changes_.

<script src="https://gist.github.com/irmerk/43031216eac8011f504c1994ca842cc2.js"></script>

#### Component

Moving into the component, we have a straightforward approach to setting up state and dispatching to result in cleaner code.

<script src="https://gist.github.com/irmerk/38a35588fd46784899f39674f1c96fc2.js"></script>

#### Action Creator

Dispatched to the store for handling, actions are objects containing a description of an event. Because actions are made by action creator functions, the one dispatching the action does not need to know the exact structure.

With Sagas, actions are slightly different. Three actions happen for each API call. Beginning the action, successful response, and error response. While this pattern does not change, the location of each call may.

Beginning an action starts within the component, which may add necessary info for making the call. Worker Sagas will be dispatching success and error actions.

<script src="https://gist.github.com/irmerk/fe217e4c2dec34ba7f6dbc8ba6ef0772.js"></script>

#### Recap

1. (`TemplateLibrary.js`)
   When the `LibraryComponent` mounts, an action (`getTemplatesAction`) is dispatched
2. (`templatesActions.js`)
   As we can see, `getTemplatesAction` dispatches an object with a type: `‘GET_AP_TEMPLATES’`.
3. (`templatesSaga.js`)
   The watcher will pick up on the action of type `‘GET_AP_TEMPLATES’` and call `pushTemplatesToStore`.
4. (`templatesSaga.js`)
   When pushTemplatesToStore is called, a few things happen. We yield an API call made by the `TemplateLibrary` imported from `@accordproject/cicero-core` and put it in an array. From there, `getTemplatesSuccess` is called with the array of templates as an argument.
5. (`templatesReducer.js`)
   This action (`GET_AP_TEMPLATES_SUCEEDED`) ends up in the reducer, updating the state with the templates array which was attached to the action.
6. (`TemplateLibrary.js`)
   Because this component is subscribed to the store and has props *prop*agated to it, the templates array is now applied to this component through props.

### Tests

Approaching testing for Redux Saga may be intimidating. A general rule for efficiency in Redux Sagas is to ensure Sagas are doing as little as possible and to move any complex logic out into a separate regular function. A couple approaches I would recommend pursuing:

#### Unit Tests

This approach steps through yield effects individually with the `next()` method. A test may inspect the yielded effect and compare it to an expected effect with `next().value`. While this is straightforward, it leads to brittle tests. This is due to the tests being coupled so tightly with the implementation and order of effects. Refactoring code will likely break tests.

A helper function called `recordSaga` is used to start a given saga outside the middleware with an action. The options object (`dispatch` and `getState`)is used to define behavior of side effects. `dispatch` fulfills put effects, and `dispatched` accumulates all the actions in a list and returns it after the Saga finishes.

<script src="https://gist.github.com/irmerk/4a9a5a2d0d5e7420bcc21acd6c113fda.js"></script>

Utilizing `recordSaga` allows us to view the type of the dispatched action in a given test case.

<script src="https://gist.github.com/irmerk/880ed492fed77886c731c80feb923c68.js"></script>

#### Integration Tests

This approach tests the effects you’re interested in. In this, you will execute the Saga until the end, mocking effects along the way. Because this is not run in isolation, results are more secure. Now refactoring should not break the tests nearly as easily. To make this process easier, we utilize the module by [Jeremy Fairbank][fairbank] - [redux-saga-test-plan][sagatest] , which helps to make assertions in the effects generated by Sagas.

This module contains `expectSaga` which returns an API for asserting that a Saga yields certain effects. It takes the generator function as an argument, along with additional arguments to pass to the generator. While `expectSaga` runs on `runSaga`, which we used in `sagaTest`, it provides a bit easier usage. This also means that `expectSaga` is asynchronous.

After calling `expectSaga` with assertions, start the Saga with `run()`. This returns a `Promise` which can then be used with a testing framework. We use Jest. If all assertions pass, the `Promise` will resolve.

<script src="https://gist.github.com/irmerk/0bd972e6bbc04e510520be73943b3c54.js"></script>

### Conclusion

Redux Saga is amazing. They provide a very clean way of performing asynchronous calls in Redux, and encourages clear, modularized code. While it is possible to accomplish the same feat without it, this will prove quite difficult and I feel it is worth the time to learn.

Feel free to contact me with any questions or feedback.

[sagalogo]: ./images/sagaLogo.png
[walkthrough]: redux-saga.md#walk
[thunklink]: https://github.com/reduxjs/redux-thunk
[generatorlink]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*#Description
[apsite]: https://www.accordproject.org/
[apgithub]: https://github.com/accordproject
[tsv2]: https://github.com/accordproject/template-studio-v2
[sagadiagram]: ./images/sagaDiagram.png
[reduxlink]: https://redux.js.org/
[fairbank]: https://medium.com/u/be7a06b10aa3
[sagatest]: https://redux-saga-test-plan.jeremyfairbank.com/
