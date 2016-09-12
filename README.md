![JS Logo](performance.jpg)
# ptt
## Performance Tips and Tricks for effective development on JavaScript

_This is an attempt to collect various JavaScript optimizations and approaches for achievement better performance.
Most of them work well in most major browsers, but some may be browser specific/JavaScript engine specific.
Some of them have links to tests and other additional information._

- advices as for caching elements are make sense in case multiple usages and/or long prototype chain. For example, Nicholas Zakas
considers: "Always cache DOM values that are used more than once to avoid a performance penalty";

- cache template’s raw text and/or pre-compiled version of the template (if you’re using HTML templates);

- perform as many changes as possible outside of the live DOM structure;

- event delegation is typically more performant than binding each element individually, so use it;

- alternative way to avoid unnecessary _reflow_, one of the most expensive functions of a browser, is to remove a node
from the live DOM before operating on it. You can remove a node from the live DOM in two ways:
 - literally remove the node from the DOM via `removeChild()` or `replaceChild()`
 - setting the display style to `none`
Once the DOM modifications have been complete then the process must be reversed and the node must be added back into the live DOM;

- cache DOM elements:

```javascript
var doc = document,
    div = doc.getElementById('id_0'),
    p   = doc.getElementById('id_1');
```

- cache repeatedly evaluated properties:

```javascript
var len = list.length; // useful in for/while/do-while loops
```

- cache function references, property values:

```javascript
var hasOwn = Object.hasOwnProperty; // useful in for-in loops
```

- use additional/subtractional/... assignment ( `+=`, `-=`, ...) instead increment/decrement (`++`, `--`).
It does the same things more straightforward and hasn't ambiguous behaviour (prefix and postfix notation of increment/decrement)
Oh yeah, and it's almost twice as fast in Gecko and slightly faster in WebKit;

- avoid global variables or store the copies in local scope to avoid digging into scope chain;

- avoid long prototype inheritance. Sometimes it's useful to use so-called _dict pattern_ and create objects without prototype
chain which prevents undesired property lookups:

```javascript
var obj = Object.create(null);
```

- avoid to change prototype chain in runtime;

- closures can increase memory usage due to the storing it's scope in the stack of execution contexts, so use it wisely;

- use literal notation for constructing objects (`/regexp/`, `[]`, `{}`, `true`/`false` etc.);

- if it's necessary use `new Function` instead of `eval`;

- avoid using `unshift` on large arrays;

- `for-in` loop is very, very slow, use `Object.keys` + `forEach`/`for`/`while`;

- use bitwise operators ( `4 / 2 === 4 >> 1`, `4 * 2 === 4 << 1`, `~~` for rounding etc.). It may be less readable but
sometimes increase performance;

- avoid using captured groups in regular expressions if you don't want to save result of matching,
use `(?:)` - non-capturing groups instead;

- use sets of simple expressions instead of one overcomplicated;

- use explicit parts and anchors in RegExp instead of unnecessary implicit, when possible (`/^function\s/` instead of `/[^\s]\s/`);

- create/compile RegExp before its multiple usage;

- use `===` (strict equality operator) instead of `==`;

- use `switch-case` construction instead of large `if-else` structure;

- use mapping instead of `switch-case` in case of simple data accessing for elegance and performance:

```javascript
var weight = 'some',
    map = {
        light: '2kg',
        middle: '4kg',
        heavy: '6kg'
    };

console.log(map[weight]);
```

- prefer `call` rather than `apply`, as `call` is slightly faster;

- use cloning objects istead of creating (`element.cloneNode()` instead of `document.createElement(element)`) ([link](https://github.com/spicyj/innerhtml-vs-createelement-vs-clonenode),
[link](https://jsperf.com/clonenode-vs-createelement-performance/58));

- avoid performance pitfalls with _long running scrips_ with the following technique (use this if the loop doesn't have to execute
synchronously and the order in which the loop’s data is processed in no matter:

```javascript
function listAsyncExec (list, executor, interval, context) {
    var list = list.slice();
    setTimeout(function _() {
        executor.call(context, list.shift());

        list.length && setTimeout(_, interval);
    }, interval);
}
```

- multiplication is faster than division in some cases, so you can increase performance of expression (`n / 8`) with (`n * 0.125`);

- in addition to previous note use _Daff's device_ loops unwinding/unrolling in loops with huge amount of iterations ([link](http://jsperf.com/duffs-device)):

```javascript
var value = 0,
    n = iterations % 8;

while ( n-- ) value += 1;

n = (iterations * 0.125) ^ 0; //multiplication is faster than division in some cases

while ( n ) {
	value += 1;
	value += 1;
	value += 1;
	value += 1;
	value += 1;
	value += 1;
	value += 1;
	value += 1;

	n -= 1;
}
```

- use `Date.now()` instead of `+new Date` or `(new Date).getTime()`;

- don’t use the `with()` statement;

- use `try {} catch (ex) {}` with caution. This creates what is known as a _dynamic scope_, similar to the effect of the `with`
statement. Some JavaScript engines, such as **V8** do not optimize functions that make use of a `try/catch` block as the optimizing
compiler will skip it when encountered;

- for clearing arrays use `array.length = 0`, not `array.splice(0)`;

- define arrays for HTML collection objects
As the DOM Level 1 spec says, "collections in the HTML DOM are assumed to be live, meaning that they are automatically updated
when the underlying document is changed". HTML collection objects are extremely slow, so use any valid technique to minimize
amount of operations with them;

- keep in mind the difference in performance between DOM methods who get live HTMLCollection and those who get snapshot:

```javascript
var dynamicTagList = document.getElementsByTagName('div'); // live
var staticTagList = document.querySelectorAll('div'); // snapshot
var dynamicClassList = document.getElementsByClassName('some'); // live
var staticClassList = document.querySelectorAll('some'); // snapshot
```

Live `NodeList` objects can be created and returned faster by the browser because they don’t have to have all of the information up front
while static `NodeList`s need to have all of their data from the start.

- don't touching the DOM without real necessity, use `documentFragment` or other technique which minimize amount of _reflows_;

- use CSS classes instead of individual styles to change a number of styles at once, which incurs a single _reflow_;

- keep in mind that retrieve a measurement that must be calculated, such as accessing `offsetWidth`, `clientHeight`,
or any computed CSS value (via `getComputedStyle()` in DOM-compliant browsers or `currentStyle` in IE), while DOM changes are
queued up to be made, forces _reflow_;

- if it's necessary to define sufficient amount of styles in runtime the approach below is appropriate (or use external CSS
file via `link`):

```javascript
var css = document.createElement('style'),
    styles = '#header { color: #555 }';

css.media = 'screen'; // if necessary
styles += ' #content { color: #333; text-align: left; }';

// (0) check for IE
// (1) innerHTML fails here
css.styleSheet ? css.styleSheet.cssText = styles : css.appendChild(document.createTextNode(styles));

document.getElementsByTagName("head")[0].appendChild(css);
```

- keep in mind differences in performance of `textContent` and `innerText`, test them before use;

- if the functions in your application tend to be run again and again, the performance improvement will be greater
than if many different functions tend to run only once, so use function wrappers only for repeated code blocks. Use _inlining_
optimization (replacing a function call with the body of the callee) when you're in doubt;

- `setTimeout(some_code, 0)` doesn't guarantee the zero delay due to various environment implementations, so
for purposes when you want to use minimal timeout for async execution of some piece of code you can use:

```javascript
setZeroTimeout = (function () {
	var fn, ctx;

	window.addEventListener('message', function () {
		fn && fn.call(ctx);
	});

	return function (_fn, _ctx) {
		fn = _fn;
		ctx = _ctx;
		window.postMessage('', '*');
	};
}());
```

- remember about _tail call optimization_, for example:

```javascript
function evalFactorial (n) {
	function _(n, acc) {
		return n < 2 ? acc : _(n - 1, n * acc);
	}

	return _(n, 1);
};
```

- to prevent memory fragmentation and slower access to array members use instant memory allocation for large array (efficiency depends on JavaScript engine):

```javascript
var db = Array(1e6);
```

- you can save some memory by using one less variable when swapping values, compare:

```javascript
var c = a; a = b; b = c;
a ^= b; b ^= a; a ^= b;
```

- use _memoization_ as a useful optimization technique for caching the results of function calls
such that lengthy lookups or expensive recursive computations can be minimized where possible. Simple example:

```javascript
function memoize (fn) {
  return function () {
      var args = Array.prototype.slice.call(arguments);

      fn.memo = fn.memo || {};

      return (args in fn.memo)? fn.memo[args] : (fn.memo[args] = fn.apply(this, args));
  };
}
```

- manual de-referencing with `delete` operator for "forcing garbage collection" is misconception so avoid it, this also
applies to setting object reference to `null`. If the reference was not the last reference to the object, the object is reachable
and will not be garbage collected;

- global variables are not cleaned up by the garbage collector during the life of your page, it's yet another reason to avoid them;

- if you’re using a data cache locally, make sure to clean that cache or use an aging mechanism to avoid large chunks of data
being stored that you’re unlikely to reuse;

- ensure that you’re unbinding event listeners where they are no longer required, especially when the DOM objects
they’re bound to are about to be removed;

- keep your functions _monomorphic_ ([link](http://mrale.ph/blog/2015/01/11/whats-up-with-monomorphism.html));

- don’t write enormous functions, as they are more difficult to optimize;

- in cases when you need a bunch of numbers or similar (when you need vectors, for example) use arrays instead of objects.
So, it's not a good idea to mix values of different types (e.g. numbers, strings, undefined or true/false) in the same array
(i.e. `var arr = [1, '1', undefined, true, 'true']`) See [link](http://web.archive.org/web/20160203020622/http://jsperf.com:80/type-inference-performance/2);

- using `delete` for array elements causes appearing "holes" (empty slots) in them. **V8** works with "holey" arrays much slower than with "packed" ones:

```javascript
var list = [1, 2, 3];
delete list[1]; // bad, makes "hole" and length isn't changed
list.splice(1, 1); // good, length decrements by 1 and array still "packed"
```

- when you need to copy objects in a performance-critical code (and you can’t get out of this situation), use an array
or a custom “copy constructor” function which copies each property explicitly. This is probably the fastest way to do it:

```javascript
function Clone(original) {
  this.foo = original.foo;
  this.bar = original.bar;
}
var copy = new Clone(original);
```

- caching your functions when using the module pattern can lead to performance improvements:

```javascript
var fn_0 = function () {},
    fn_1 = function () {},
    Constructor = function () {
      return {
          method_0: fn_0,
          method_1: fn_1
      }
  }
```

- in general, don’t delete array elements. It would make the array transition to a slower internal representation
when key set becomes sparse (be aware that accessing elements in them is much slower than in full arrays);

- try to eliminate dead code;

- breaking a bigger function into smaller pieces could help to reduce compilation time. In addition, ability of _inlining_ (i.e. replacing a function call site with the body of the callee), for example, in **Crankshaft** (**V8** optimizing compiler), depends on the amount of characters in a function or callback, which is equal 600 (maximum source size in bytes considered for a single _inlining_) by default. There is an option for **Node V8** called `--max_inlined_source_size` (you can tweak it). Latter is especially important for callable objects that are called repeatedly;

- only some patterns in `arguments` usage are supported for optimized code (at least in **V8**):

```javascript
arguments.length
arguments[i]
f.apply(obj, arguments)
```

### Additional reading

- [Optimization killers](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)
- [Explaining JavaScript VMs in JavaScript - Inline Caches](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html)
- [Writing Fast, Memory-Efficient JavaScript](https://www.smashingmagazine.com/2012/11/writing-fast-memory-efficient-javascript/)

### NB

Doing too much can be as harmful as not doing anything. Too much of a good thing is a bad thing, so avoid _over-optimization_.
Modern JavaScript engines does many optimizations automatically and some kinds of techniques can potentially “cancel” some of them.

> If you optimize everything, you will always be unhappy.

**Donald Knuth**
