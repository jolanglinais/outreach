# Comparison of JS, Ramda, and Lodash 

![JS Libraries Logo](../images/JSUtils.png)

With [_ECMAScript 2020_][ECMAscript] available, external libraries are not necessary for functional programming (FP) - specifically currying and composition. The two main libraries for this kind of work have been [Ramda][ramda] and [Lodash FP][lodashFP]. [UnderscoreJS][underscoreJS] is another, but Lodash is generally thought of as an improvement on this library. Lodash is a fork of Underscore, and the [history of why it forked][lodashHistory] is rather interesting.

However, it can still be a good idea to use one of these tried-and-tested libraries for more complex situations with FP. If these complex scenarios are not being taken advantage of, vanilla JavaScript can keep up with utility libraries for the most part. Some notable exceptions would be `debounce` from [Lodash][lodash] and `merge` from Ramda.

To reiterate, a lot of the benefits that lead to the use of Ramda and Lodash have been baked into vanilla JavaScript. Arrow functions allow for a version of currying, and along with chaining functions, can adequately compose functions. Similarly, prototypal methods are being added every version which makes Lodash less and less useful.

_Note_: Arrow functions don't allow for _actual_ currying (`(a, b) => {}` is the same as `a => b => {}`, i.e. the function itself tracks how many of its arguments have been defined), just quite close.

This article will:

- Give a brief overview of Ramda and Lodash (FP)
- Note the cases in which it makes sense to invest in the library or not
- Give context to a few methods which stand out
- Provide a table summary for which library is better in which regard
- Provide a [**REPL**][myRepl] and [**repository**][myRepo] for generating benchmarks

All this being public means you are free to contribute to the list and make adjustments

## JavaScript

As previously stated, native JavaScript has become _quite_ a bit more powerful over the past few years. While helper and utility libraries are still helpful, most everything in them can be reduced down to some combination of `filter()`, `map()`, and `reduce()`. 

I write more at length in my [Modern Javascript Techniques][modernJS] article.

### Use Cases:

- Functionality needed is straightforward, with few steps or transformations needed
- Complex functionality needing a few extra steps is not a deterrent
- Bundle size being important
- Learning the process that goes into these simplified helper functions from other libraries

## Ramda

Ramda emphasizes a purer functional style, with immutability and side-effect free functions being at the heart of the design philosophy. Ramda is about _transforming_ data and _composing_ functions. This is why things like `throttle` and `debounce` are not supported because they involve side-effects. To achieve this in a pure way, [functional reactive programming][frp] would be required to abstract over this with event streams.

Ramda functions are _automatically_ curried. This allows for easily building up new functions from old ones by not supplying the final parameters. The parameters to Ramda functions are arranged to make it convenient for currying. The data to be operated on is generally supplied last. These last two points together make it very easy to build functions as sequences of simpler functions, each of which transforms the data and passes it along to the next. Ramda is designed to support this style of coding.

> Ramda provides several functions that return problematic values such as `undefined`, `Infinity`, or `NaN` when applied to unsuitable inputs. These are known as [partial functions][partialFn]. Partial functions necessitate the use of guards or null checks.  

A remedy for this could be [Sanctuary][sanctuary], a JavaScript functional programming library inspired by [Haskell][haskell] and [PureScript][purescript]. It's stricter than Ramda, and provides a similar suite of functions.

### Use Cases:

- Composition, taking data last and always currying
- Specific methods, typically involving complex operations, e.g. `merge`, `assoc`, `pluck`...
- Similar common methods used in multiple places
- Complex, non-linear composition by using `R.converge()`

## Lodash

There's little to go into here. Lodash is an extremely performant utility library. While bundle size has been an issue in the past, Lodash has become much more modularized in format. This enables build tools like webpack and parcel to do tree-shaking and removing any unused functions, which reduces bundle size.

Keep in mind that there are many functions which [can be done natively][doNatively].

_Note_: While Lodash appears faster in the benchmarks below with the `_.toString()` method, the results were actually not identical to the same functions in JS and Ramda.

### Use Cases:

- `debounce`
- Similar common methods used in multiple places

### Lodash FP

Lodash provides `lodash/fp`, a module to promote a more functional programming style. This module allows for curried versions of the Lodash functions.  This makes Lodash a good alternative to Ramda.

### Use Cases:

- Composition, taking data last and always currying

---

# Benchmark Results

Note that I have begun this list with common methods I and my team use, and it is in no means exhaustive. Please feel free to look into the [repository][myRepo] and open a pull request to add further methods or tests.

|         | Speed | Readability | Does Have | Does Not Have |
| ------- | ----- | ----------- | ------------- | ------------- |
| Symbols | 🔵    | 🔶          | ✅            | ❌            |


|                    | Javascript | Lodash       | Ramda       |
| ------------------ | ---------- | ------------ | ----------- |
| Downloads (week)   | N/A ✅     | 41,323,748   | 7,952,372   |
| Size (unpacked)    | N/A ✅     | 1.41 MB      | 1.07 MB     |
| Size (minified)    | N/A ✅     | 69.9 kB      | 53.4 kB     |
| Size (mini+gzip)   | N/A ✅     | 24.4 kB      | 12.4 kB     |
| Download Time      | N/A ✅     | 488 ms       | 247 ms      |
| Issues             | N/A ✅     | 107          | 211         |
| Last Publish       | N/A ✅     | 4 month      | 5 month     |
| **FEATURES**       |            |              |             |
| Curry              | Yes        | Yes          | Yes         |
| Immutable          | No         | No           | Yes ✅      |
| Chainable          | Yes ✅     | Yes          | Yes         |
| Functional         | No         | Yes          | Yes ✅      |
| **SECURITY**       |            |              |             |
| Known Issues       | No         | [Yes][loSec] | [No][raSec] |
| Dependencies       | No         | No           | No          |
| **COMMON METHODS** |            |              |             |
| **Arrays**         |            |              |             |
| `all`              | ❌         | ❌           |             |
| `concat`           |            |              | 🔵          |
| `each`             |            |              | 🔵          |
| `filter`           |            |              |             |
| `find`             |            |              |             |
| `findIndex`        |            |              | 🔵          |
| `flatten`          |            | 🔵           |             |
| `fromPairs`        |            |              |             |
| `head`             |            |              |             |
| `map`              |            | 🔵           | 🔵          |
| `pluck`            | ❌         | ❌           |             |
| `range`            |            | 🔵🔶         | 🔶          |
| `reduce`           |            | 🔵           | 🔵          |
| `reject`           |            | 🔵           | 🔵          |
| `tail`             | 🔵         | 🔵           |             |
| `uniq`             | 🔵         | 🔵🔶         | 🔶          |
| `zip`              | ❌         |              | 🔵          |
| **Objects**        |            |              |             |
| `assoc`            | ❌         | ❌           |             |
| `keys`             | 🔵         |              | 🔵          |
| `merge`            | ❌         |              | 🔵          |
| `omit`             |            | 🔶           | 🔵🔶        |
| `path`             |            |              |             |
| `pick`             | 🔵         | 🔶           | 🔵🔶        |
| `toPairs`          | 🔵         |              | 🔵          |
| `values`           | 🔵         |              |             |
| `zipObj`           | ❌         | 🔶           | 🔵🔶        |
| **Strings**        |            |              |             |
| `toString` array   | 🔵         |              |             |
| `toString` object  |            | 🔵           |             |
| `toString` date    |            |              |             |
| `split`            |            |              |             |
| `toLower`          |            |              |             |
| `toUpper`          |            |              |             |
| **Utility**        |            |              |             |
| `clone`            | 🔵         | 🔵🔶         | 🔶          |
| `debounce`         | ❌         |              | ❌          |
| `isEmpty`          |            |              |             |
| `isEqual`          | ❌         | 🔵           |             |
| `isFunction`       |            |              |             |
| `isNil`            |            |              |             |
| `type`             |            |              |             |
| **Composition**    |            |              |             |
| Numbers            |            | 🔵           |             |
| Objects            | 🔵         |              | 🔵          |
| Functional         |            | 🔵           |             |
| **Overall**        | 🔵         | 🔵           |             |
| **Totals**         | **10**         | **16**           | **21**          |

---

Test: Arrays and Collections

| Name   | JS Time (ms) | _ Time [ms] | R Time [ms] | Diff to JS |
|--------|--------------|-------------|-------------|------------|
| concat            | 17          | 19         | 7          | +83%       |
| each              | 11          | 15         | 4          | +93%       |
| filter            | 17          | 22         | 14         | +19%       |
| find              | 10          | 10         | 7          | +35%       |
| findIndex         | 11          | 15         | 6          | +58%       |
| flatten (deep)    | 1438        | 174        | 1937       | +156%      |
| fromPairs         | 531         | 512        | 513        | +3%        |
| fromPairs (reduce)| 542         | 509        | 510        | +6%        |
| head              | 0           | 1          | 3          | N/A        |
| map               | 15          | 9          | 11         | +50%       |
| range             | 533         | 34         | 62         | +176%      |
| reduce            | 64          | 14         | 14         | +128%      |
| reject            | 1263        | 35         | 31         | +190%      |
| tail              | 1           | 3          | 6          | -100%      |
| uniq              | 5           | 4          | 43         | +22%       |
| zip               | N/A         | 19         | 7          | N/A        |

---

Test: Objects

| Name   | JS Time (ms) | _ Time [ms] | R Time [ms] | Diff to JS |
|--------|--------------|-------------|-------------|------------|
| keys           | 145          | 800         | 109         | +28%       |
| merge (triple) | N/A          | 100         | 7           | N/A        |
| omit           | 16           | 35          | 7           | +78%       |
| path (short)   | 1            | 3           | 3           | -100%      |
| path (long)    | 1            | 2           | 3           | -66%       |
| pick           | 2            | 12          | 2           | -0%        |
| toPairs        | 71           | 107         | 52          | +30%       |
| values         | 5            | 94          | 28          | -139%      |
| zipObj         | N/A          | 121         | 48          | N/A        |

---

Test: Strings

| Name   | JS Time (ms) | _ Time [ms] | R Time [ms] | Diff to JS |
|--------|--------------|-------------|-------------|------------|
| toString (array) NOTE _  | 46           | 151         | 2391        | -106%      |
| toString (object) NOTE _ | 163          | 4           | 693         | +190%      |
| toString (date) NOTE _   | 10           | 19          | 16          | -46%       |
| split                    | 592          | 633         | 601         | -1%        |
| toLower                  | 29           | 29          | 32          | -0%        |
| toUpper                  | 25           | 27          | 30          | -7%        |

---

Test: Utility

| Name   | JS Time (ms) | _ Time [ms] | R Time [ms] | Diff to JS |
|--------|--------------|-------------|-------------|------------|
| clone      | 0            | 0           | 15          | N/A        |
| debounce   | N/A          | 0           | N/A         | N/A        |
| isEmpty    | 1            | 0           | 0           | N/A        |
| isEqual    | N/A          | 25          | 106         | N/A        |
| isFunction | 0            | 0           | N/A         | N/A        |
| isNil      | 0            | 0           | 0           | N/A        |
| type       | 0            | N/A         | 0           | N/A        |

---

Test: Totals

| Name   | JS Time (ms) | _ Time [ms] | R Time [ms] | Diff to JS |
|--------|--------------|-------------|-------------|------------|
| Curried / Piping Numbers | 1452         | 3           | 2941        | +199%      |
| Curried / Piping Objects | 825          | 1167        | 748         | +9%        |
| Curried / Piping FP      | N/A          | 25          | 1094        | N/A        |
| Common Methods           | 528          | 554         | 1155        | -4%        |

---

# Conclusions

Both Ramda and Lodash overlap, and should likely not be used in the same project. Depending on what data you're working on and which method you're using, these libraries can be very beneficial or unnecessary.

An approach of **Vanilla-JavaScript-First** should be taken, and these libraries shouldn't be used as a blanket approach to methods on data. Once you come across something that it is especially difficult to do in vanilla JavaScript, switch to one of these libraries. Which one? Comes down to taste. Both have a quite similar semantic style.

Ramda is generally a better approach for functional programming as it was designed for this and has a community established in this sense.

Lodash is generally better otherwise when needing specific functions (esp. `debounce`).

Either way, **ensure you invest in tree shaking** to minimize the bundle sizes of these libraries, because odds are you will only be using a few methods and do not need the entire library.

[ECMAscript]: https://tc39.es/ecma262/
[ramda]: https://ramdajs.com/
[lodashFP]: https://github.com/lodash/lodash/wiki/FP-Guide
[underscoreJS]: https://underscorejs.org/
[lodashHistory]: https://stackoverflow.com/a/13898916
[lodash]: https://lodash.com/
[modernJS]: https://dev.to/irmerk/modern-javascript-techniques-4chc

[frp]: https://en.wikipedia.org/wiki/Functional_reactive_programming
[partialFn]: https://en.wikipedia.org/wiki/Partial_function
[sanctuary]: https://sanctuary.js.org/
[haskell]: https://www.haskell.org/
[purescript]: http://www.purescript.org/
[doNatively]: https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore

[loSec]: https://snyk.io/advisor/npm-package/lodash
[raSec]: https://snyk.io/advisor/npm-package/ramda
[myRepl]: https://repl.it/@irmerk/Comparing-Utility-Libraries
[myRepo]: https://github.com/irmerk/javascript-utilities-ramda-lodash