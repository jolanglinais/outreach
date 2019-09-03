# Modern Javascript Techniques

## Clean and Scalable Syntax in Pursuit of Purity

![Colorful Paint Header Image][mjslogo]

As a beautifully complex and adaptive language, _[JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)_ has many advantages which grow every year. It is no wonder that the language and community is so extremely popular given that it had a large hand in bringing user interface interactivity and responsive web design to the internet. While sometimes complicated, the language proves to be easy to pick up and start, and allows for faster user experiences by being executed client-side.

A large period of time saw JavaScript as problematic and flawed. This was never an issue with the language, but rather the platform it ran on: the browser. This ecosystem was flawed, as there were so many branching factions — most notably Microsoft coming in and mucking everything up. _[Mozilla](https://www.mozilla.org/en-US/)_ was a voice of reason throughout this period, but it was not until Chrome gained enough market share to give incentive to realign people around a standard of how the engine should look and be built. Setting the standard with _[V8](https://v8.dev/)_ is how _[Node.js](https://nodejs.org/en/)_ was subsequently built. As a full programming language with server side execution support, JavaScript now powers modern web applications and scales across the tech stack.

### <a name="approach"></a> Approach

![Accord Project Logo Image][aplogo]

My experience both as a maintainer of the _[Accord Project](https://www.accordproject.org/)_ , an open source project for smart legal contracts, and as a Full Stack Engineer has shown me the powerful applications in which JavaScript can be implemented. Moreover, I have become quite keen to learn and adopt better and more efficient practices within the language. I will be sharing this as both a useful reference for others, as well as a historical reference for myself in the future. I am hoping to branch off this to expand on topics covered in here in subsequent, deeper dive articles.

The majority of our work at the Accord Project is in JavaScript , with some domain specific language mixed in. To architect a reliable tech stack which allows stability and efficiency for smart contracts, the Accord Project relies on JavaScript, as well as _[OCaml](https://ocaml.org/)_ and _[Ergo](https://www.accordproject.org/projects/ergo)_. JavaScript provides the best set of tools to handle this in a wide array of use cases and environments. We chose JavaScript because of its applicability, diversity of libraries, and ease of use. Syntax within this language is expressive yet simple.

The Accord Project core codebase contains more than 250k lines of code. Along with our template library and UI components, there is near a million.

## Outline:

→ [Approach][approach]
→ [Fundamentals][fundamentals]
→ [Workflow][workflow]
→ [Operations][operations]
→ [Functions][functions]
→ [Asynchronous][asynchronous]
→ [Functional Programming][functional]
→ [Conclusion][conclusion]
→ [Resources][resources]

---

### <a name="fundamentals"></a> Fundamentals

#### Comprehensible

Document code. Readability is paramount for programming, as it is humans who will need to interpret the code in order to collaborate. Being verbose enough to be legible at a later date or for another person is better practice than saving a few extra characters by naming variables with a single letter. Moreover, commenting and documentation - such as the _[JSDocs](https://devdocs.io/jsdoc/about-getting-started)_ format - are extremely useful for building accessible code which can be shared with a team or others.

It may seem redundant at first, but commenting code as best as possible will allow for easy refreshing through this built-in documentation months later when you circle back to a project or when pairing with a colleague.

#### Globals

Avoid variables in the global scope. Multiple reasons exist for avoiding variables in the global scope. Performance is reduced due to function execution causing JavaScript to search through the scope change from in to out until it hits the global object. Furthermore, security flaws exist from this because functions can be invoked through the browser when they’re defined in the global space. This point will come up again in the functional programming [section][functional].

#### Variables

Stop using `var`. The scoping behavior is inconsistent and confusing, which can result in bugs. ES6 brought in `const` and `let`. Aim for using strictly `const`, and only `let` if that is not possible. There is more restriction and const is un-reassignable, but not quite immutable. The variable will have an unchanging reference to the same object or primitive value, but the value held by the variable is not immutable. Still, this will be best practice moving forward.

#### Naming

A bit of digression, but programmers can expend 10x amounts of energy on naming conventions, yet struggle to be inclusive with their language.

Taking the time to be descriptive and appropriate for legibility and comprehensive readability will do wonders in the future of the code.

This is especially important for those looking to educate others; variable names should help explain and give context to what is happening in the code. Someone new to this code should be able to have a loose understanding of what is happening. Use verbs! An example for a Boolean variable could start with `is...` and examples of functions could be action verbs.

Good reference material can be found here: _[A Grammar-Based Naming Convention](https://dev.to/somedood/a-grammar-based-naming-convention-13jf)_

---

### <a name="workflow"></a> Workflow

A major key to maintainability is keeping logic in the right place and not cluttered or disorganized. The way a project or codebase is structured can make a large impact on how easy it is to understand and follow.

#### Importing Order

Starting at a granular level, the order in which different modules are imported can reduce confusion by having a predictable pattern. The specific structure you use is less important than there being _some_ sort of structure:

```js
/* Packages */
import React, { useState } from 'react';
import PropTypes from 'prop-types';
import * as R from 'ramda';

/* Styled Components */
import * as SC from './styles';

/* Components */
import Navigation from './Navigation';

/* Actions */
import * as ACT from './actions';

/* Utilities */
import { navigateToClause } from '../utilities';
```

#### Modularization

A goal to keep in mind is to keep packages, modules, functions, and scopes small. Reusability becomes much easier, as well as chaining, when this is in practice. Similar functions or those with many steps could be grouped into one module or class. Try to keep functions as simple as possible, and perform complex processes in steps.

Once a file has grown above 300-400 lines of code, there is a strong case for being too cluttered and unmaintainable. At this point, a lot of benefit can be gained from creating new modules and folders to break up processes. Think of a project as a tree with many branches, rather than a mountain of heaped-up code.

_[ESLint](https://eslint.org/)_ is a great tool to help here. Aim for keeping files less than four or five indentations deep. This keeps code specialized and encourages cleanup of dead code. Several functions that do one small process will be more useful than one function which does several things. The large function can only be used in that one way, whereas smaller functions may be able to be used in multiple processes around a project. Exposing these smaller, helper functions creates a robust API base in a project.

Great code can be improved upon without rewriting everything.

#### Isolate code

A function _should_ have one purpose and not do multiple actions. That purpose _should_ be something other than a side effect, but we will circle back to this in functional programming [section][functional]

A contrived example of this is encapsulating conditionals:

```js
// NO:
if (props.contract.errors === [] && isEmpty(parseErrors)) {
  // ... code
}

// YES:
const errorsExist = (props, parseErrors) =>
  props.contract.errors === [] && isEmpty(parseErrors);

if (errorsExist(contractProps, parseErrors)) {
  // ... code
}
```

#### Guard Clauses

A great way to construct functions which have edge cases which result in an error or empty result is to have checks for these invalid results early. If this condition is not met or there is an invalid use case, then the bulk of the computation is prevented because we already know the result. This is referred to as the _[Bouncer Pattern](http://rikschennink.nl/thoughts/the-bouncer-pattern/)_ or _[Guard Clauses](https://dev.to/lanecwagner/guard-clauses-how-to-clean-up-conditionals-2fdm)_:

```js
const parseContract = contract => {
  // Does a contract exist
  if (!contract) return 'Error, no contract!';

  // Are there already parsed errors
  if (contract.currentErrors.length > 0) return contract.currentErrors;

  // Parse the contract
  return contract.clauses.map(clause => doSomething(clause));
};
```

Not only will this optimize code, but will encourage thinking of functions and processes in a way which takes handling edge cases into consideration.

#### Prettier + Linting

A theme to my article here is that code should be easy to read and understand. With that comes consistent styling and structuring. A linter - any linter - will be greatly useful. ESLint is a linter, and will identify issues with code correctness such as warning from using `var`. _[Prettier](https://prettier.io/)_ is a formatter, which will identify issues with uniformity and consistency and automatically align brackets, for example. Using both in conjunction is encouraged.

_[StandardJS](https://standardjs.com/)_ and ESLint’s _[predefined config](https://github.com/feross/eslint-config-standard)_ are good sources for linting rules if you need a good starting point.

---

### <a name="operations"></a> Operations

#### Destructuring

Destructuring can help save a lot of typing and lines of code by keeping variables short and pulled from an object early on. Introduced with _[ECMAScript 6](https://www.w3schools.com/js/js_es6.asp)_, this allows access to specific fields from any object or module and immediately assign it to a variable.

Objects:

```js
// NO
const generateText = contract => {
 const clauses = contract.body.clauses;
 const text = contract.body.text;
 const errors = contract.errors;

 Cicero.parseContract( clauses, text )
};

// YES
const generateText = contract => {
 const { body: { clauses, text }, errors }, = contract;

 Cicero.parseContract( clauses, text )
};
```

Arrays (skipping elements consist of `, ,`):

```js
// NO
const lettersArray = ['A', 'B', 'C', 'D', 'E', 'F'];
const firstLetter = lettersArray[0]; // "A"
const thirdLetter = lettersArray[2]; // "C"

// YES
const [firstLetter, , thirdLetter, ...remaining] = lettersArray; // remaining = [ "D", "E", "F" ]
```

Functions (similar to objects):

```js
// NO
const generateText = contract => {
  if (contract.errors) return 'Errors exist!';
  if (!contract.clauses) return 'No clauses exist!';
};

// YES
const generateText = ({ errors = null, clauses = null }) => {
  if (errors) return 'Errors exist!';
  if (!clauses) return 'No clauses exist!';
};
```

#### Default Values

When destructuring, there is an ability to assign default values to parameters. This can also indicate to the user what values can be passed in or are required.

```js
const generateText = ({
  name = 'Stock Contract',
  language = 'English',
  text = 'No text exists yet!',
  errors = [],
  clauses = [],
}) => {
  Cicero.parseContract(clauses, text);
};
```

If no error should be thrown when a value is not passed, a default value could be useful.

#### Ternary

This operator works similar to _[logical operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators)_ and `if...else` statements, and has three sections:

1. Boolean conditional
2. Return value in case of truthy
3. Return value in case of falsy

```js
// condition ? truthyResult : falsyResult
const errorArrayLength = errors => (errorsExist(errors) ? errors.length : 'No');
```

Try to steer clear of negative conditionals - check if something _does_ exist, rather than if it does not exist.

#### Spread

Another form of object destructuring, the spread operator allows for value extraction from data without having to iterate over the data explicitly. This is common in _[Redux](https://redux.js.org/)_ and functional programming, as it is a short way to add to an object without mutating it - copy an old object by spreading it and adding a new value to it.

```js
const firstHalf = ['A', 'B', 'C'];
const secondHalf = ['D', 'E', 'F'];

const lettersArray = [...firstHalf, ...secondHalf];
// lettersArray = [ "A", "B", "C", "D", "E", "F" ];
```

```js
const contract = {
	text = "No text exists yet!",
	errors = []
};

const contractWithClauses = {
	...contract,
	clauses = []
};
```

#### Template Literals

This feature allows for embedding dynamic content into strings and writing strings which bridge multiple lines. These are designated with backquotes and template literal snippets (`${}`).

```js
// NO
var contractTitle =
  'Contract Name: ' +
  contract.name +
  ', Errors: ' +
  contract.errors.length +
  '.';

// YES
const contractTitle = `Contract Name: ${contract.name}, Errors: ${contract.errors.length}.`;

// OTHER USES
const conditionalTitle = `${
  contractExist() ? 'Contract Name: ' + contract.name : 'No contract exists.'
}`;
const multipleLines = `Hello,

Good to meet you`;
```

---

### <a name="functions"></a> Functions

#### Limit Scope

Functions _should_ do one thing. They become difficult to test and reason through once they begin performing multiple actions. Aim to have no more than one level of abstraction in functions - split functions up if necessary.

```js
// NO
const parseContract = contract => {
  contract.forEach(contract => {
    const contractText = generateText(contract);
    if (contractText.noErrors()) {
      execute(contract);
    }
  });
};

// YES
const isContractValid = contract => {
  const contractText = generateText(contract);
  return contractText.noErrors();
};

const parseContract = contracts =>
  contracts.filter(isContractValid).forEach(execute);
```

#### Arrow

This newer syntax for functions provides a concise and clear flow to the notation. These also have more practical scoping behavior by inheriting `this` from the scope in which the function was defined in.

Previously, a function would be written as:

```js
function someFunction(input) {
  // ... code
}
```

Now we define the same thing as:

```js
const someFunction = input => {
  // ... code
};
```

If the function is only returning something simple, we can write this in one line with an implicit `return` statement:

```js
const add = (a, b) => a + b;
const createObject = (a, b) => ({ a, b });
```

#### Parameters

Aim for limiting the amount of parameters passed into a function to improve testability. Ideally, this would be below three. Usually, if there are three or more arguments, the function may be trying to do many things itself and should be split up and consolidated.

#### Chaining

A source of current frustration comes from the inability to easily access a nested value within an object. Something like this may be used currently:

```js
if (
  contract &&
  contract.firstProp &&
  contract.firstProp.secondProp &&
  contract.firstProp.secondProp.thirdProp &&
  contract.firstProp.secondProp.thirdProp.fourthProp.data
)
  execute(contract.firstProp.secondProp.thirdProp.fourthProp.data);
```

Hideous.

The reason for doing this is if you go straight for the last line, you could run into this kind of error:

```bash
TypeError: Cannot read property ‘fourthProp’ of undefined
```

TC39 (the technical committee that determines what features become a part of the JavaScript standard) has moved the _[Optional Chaining proposal](https://github.com/tc39/proposal-optional-chaining)_ the later stages of acceptance.

I am really looking forward to this, because it would make the above code appear as such:

```js
const data = contract?.firstProp?.secondProp?.thirdProp?.fourthProp?.data;
if (data) execute(data);
```

If any property does not exist, the digging exits and returns `undefined`.

Another current solution to this is _[Ramda](https://ramdajs.com/)_, which uses a function called `path` to safely execute code at run-time and not run into `undefined` errors in the console.

---

### <a name="async"></a> Asynchronous

I have previously written about _[Asynchronous with Redux Sagas](https://dev.to/irmerk/asynchronous-with-redux-sagas-44dm)_, but will be focusing more on `async`/`await` and promises for this.

Asynchronous simply means things happen independently of the main program flow; computers are designed this way. A processor will not pause to wait for a side effect to happen to resume operations. JavaScript is synchronous by default and single threaded; code cannot run in parallel. However, JavaScript was designed to respond to user actions, which are asynchronous in nature. The browser, in which JavaScript lives, provides a set of APIs which handle this functionality. Moreover, _[Node.js](https://nodejs.org/en/)_ introduces a non-blocking I/O environment to extend this concept to files, network calls, etc.

![Asynchronous vs. Synchronous][asyncdiagram]

When this side function is handed over to a separate thread, such as an API call., it returns as a callback, which is a function passed into another function as an argument. This is then invoked inside the outer function to complete an action.

#### Async + Await

Previously, JavaScript relied on promises and callbacks for asynchronous code. This could easily result in _[Callback Hell](https://blog.hellojs.org/asynchronous-javascript-from-callback-hell-to-async-and-await-9b9ceb63c8e8)_. This syntactic sugar built on top of promises provides a much smoother way of handling asynchronous code, but cannot be used with plain callbacks or node callbacks. Now asynchronous code can be written more like synchronous code. Similar to promises, these are non-blocking.

Functions which use this require the `async` keyword before it, and `await` can only be used in functions which have this keyword. This `async` function implicitly returns a promise which will resolve to the value returned inside the function.

```js
// Promises
const outsideRequest = () =>
  retrieveData()
    .then(data => {
      execute(data)
      return “Executed”
    })

// Async/Await
const outsideRequest = async () => {
  execute(await retrieveData())
  return “Executed”
}
```

Benefits:
`+` Clarity - Less code and more readable.
`+` Error handling - `try/catch` can handle both synchronous and asynchronous code
`+` Conditionals - More straight forward handling of dynamic results
`+` Debugging - Error stack traces are much easier to track
`+` Await anything

---

### <a name="functional"></a> Functional Programming

There are two major paradigms when it comes to programming, imperative and declarative. An imperative way to approach writing a function would be to explain each minute step of the process, whereas declarative takes the approach of expressing computational logic without describing specific flow.

**Imperative**: How to do something
_Example_: Instruct someone to bake a cake, step-by-step
**Declarative**: What to do
_Example_: Telling someone to bake a cake by describing a cake

Functional programming is declarative. An intimidating and powerful programming paradigm, this treats computation as the evaluation of mathematical functions and avoids changing _[state](https://en.wikipedia.org/wiki/Program_state)_ and _[mutable](https://en.wikipedia.org/wiki/Immutable_object)_ data. Functions are first class entities in JavaScript, which means they are treated as values and can be used as data. Functions can be referred to from constants and variables, be passed as a parameter to other functions, and be returned as a result of a function.

In functional code, output values are contingent upon _only_ the arguments passed in, and will always result in the same value for the same input. Object-Oriented Programs, in contrast, can often depend on state and can produce different results at different times with the same arguments.

#### Pure Functions

A pure function is one which follows some guidelines of functional programming, namely it returns the same result given the same arguments (_[idempotent](https://en.wikipedia.org/wiki/Idempotence)_) and does not cause observable side effects. This makes it referentially transparent, and a benefit of this is that this code is much easier to test. With this concept, we are able to _[memoize](https://en.wikipedia.org/wiki/Memoization)_ these functions.

#### Side Effects

Mutability is avoided in functional programming, and an example of this would be modifying the global object or a value in the global scope. Instead of mutating, functional programming aims to create new copies of data with additions or subtractions rather than mutating the original data.

The main point is to avoid pitfalls like sharing state between objects or using mutable data that can be written to by anything. An action which is not pure, such as writing to a file, should be limited to one service which does it - minimize impure functionality.

In JavaScript, primitive data types are passed by value, whereas objects are passed by reference. So if a function makes a change to an array, any other function which references that array will be affected. This is a huge danger that functional programming seeks to avoid; if two separate and unrelated functions take the same input, but one of the functions mutates that input, the other function is now flawed. It can become taxing to performance to be cloning large objects all the time, but there are great libraries which are quite performant, such as _[Ramda](https://ramdajs.com/)_.

#### Ramda

![Ramda.js][ramdalogo]
An excellent library which provides extra utility to functional programming in JavaScript, making it easier to create code pipelines. All functions are automatically curried, which makes this library extremely useful. Their wiki has a helpful section to help you find "_[What Function Should I Use](https://github.com/ramda/ramda/wiki/What-Function-Should-I-Use)_"

_[Currying](http://www.sitepoint.com/currying-in-functional-javascript/)_ gives us the ability to use higher order functions (ones which take functions as input and return functions) and closures to great effect. Instead of a function with multiple arguments, a curried function would be one which takes a single argument and returns a function that takes a single argument. These are strung together to create a pipeline.

#### Piping

While Ramda is great for composing functions together in a pipe, JavaScript is a constantly evolving language and will soon have this natively. TC39 currently has a proposal for a _Pipeline Operator_ _[Pipeline Operator](https://github.com/tc39/proposal-pipeline-operator/wiki)_. In the meantime, check out Ramda and find some really powerful tools!

---

### <a name="conclusion"></a> Conclusion

The trope is old, criticism of JavaScript from many circles has lost merit. I suppose it is taking some 10x longer to get over their misgivings. This language has a high level of efficacy and is suitable for many environments and applications. There are a lot of exciting use cases all across technology, with the ability to touch the full stack.

Gatekeeping and toxicity in this field aside, the ability to access so many different sectors provides for a more collaborative and better experienced population in the community. This language has _so much_ power. Cross platform desktop apps can be built with JavaScript in Electron, mobile apps with React Native, and server-side solutions with Node.js.

While there is constant evolution to this language, there is not quite a new framework every week. Progression is good, and the community behind this language is quite progressive and innovative.

Feel free to contact me with any questions or feedback.

---

### <a name="resources"></a> Resources

#### Community

- [DEV #javascript](https://dev.to/t/javascript)
- [Javascript Weekly](https://javascriptweekly.com/)

#### Education

- [FreeCodeCamp](https://learn.freecodecamp.org/)
- [Khan Academy Computer Programming](https://www.khanacademy.org/computing/computer-programming)
- [A Re-Introduction to JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
- [The Modern JavaScript Tutorial](https://javascript.info/)

#### Books

- [You Don’t Know JavaScript](https://github.com/getify/You-Dont-Know-JS)
- [Eloquent Javascript](https://eloquentjavascript.net/)

#### Blogs

- [Eric Elliott](https://medium.com/@_ericelliott)

#### Podcasts

- [Javascript Jabber](https://pca.st/m5IV)
- [JS Party](https://pca.st/ijqf)
- [Syntax.fm](https://pca.st/fmx9)
- [Full Stack Radio](https://pca.st/fullstack)
- [Ladybug Podcast](https://pca.st/ZD17)
- [Javascript to Elm](https://pca.st/tr6K)
- [Elm Town](https://pca.st/i2d2)

#### Misc

- [JavaScript: Understanding the Weird Parts](https://www.youtube.com/watch?v=Bv_5Zv5c-Ts)
- 30 days of JavaScript challenges with corresponding videos by Wes Bos: [JS 30](https://javascript30.com/)
- [Fun Fun Function](https://www.youtube.com/channel/UCO1cgjhGzsSYb1rsB4bFe4Q/featured)
- Switch Case vs Object Literal:
  _ [Switch case, if else or a loopup map by May Shavin](https://medium.com/front-end-weekly/switch-case-if-else-or-a-lookup-map-a-study-case-de1c801d944)
  _ [Rewriting Javascript: Replacing the Switch Statement by Chris Burgin](https://medium.com/chrisburgin/rewriting-javascript-replacing-the-switch-statement-cfff707cf045)
- Static Typing
  _ [TypeScript (TS)](https://en.wikipedia.org/wiki/Microsoft_TypeScript)
  _ [Get Started With TypeScript in 2019](https://dev.to/robertcoopercode/get-started-with-typescript-in-2019-6hd)
  _ [Gentle Intro To TypeScript](https://scrimba.com/g/gintrototypescript)
  _ [Understanding TypeScript’s type notation](http://2ality.com/2018/04/type-notation-typescript.html)
- Functional Frontend
  _ [Elm](https://guide.elm-lang.org/)
  _ [Elm Tutorial](https://elmprogramming.com/)

[mjslogo]: ../images/modernJS.jpg
[aplogo]: ../images/APLogo.png
[approach]: #approach
[fundamentals]: #fundamentals
[workflow]: #workflow
[operations]: #operations
[functions]: #functions
[asynchronous]: #async
[functional]: #functional
[conclusion]: #conclusion
[resources]: #resources
[asyncdiagram]: ../images/AsyncDiagram.jpg
[ramdalogo]: ../images/ramda.jpg
