![JS Logo](performance.jpg)
# ptt
## Performance Tips and Tricks for effective development on JavaScript

_This is an attempt to collect various JavaScript optimizations and approaches for achievement better performance. 
Most of them work well in most major browsers, but some may be browser specific/JavaScript engine specific.
Some of them have links to tests and other additional information._

- advices as for caching elements are make sense in case multiple usages and/or long prototype chain. For example, Nicholas Zakas 
considers: "Always cache DOM values that are used more than once to avoid a performance penalty";

- perform as many changes as possible outside of the live DOM structure;
 
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

- use cloning objects istead of creating (`element.cloneNode()` instead of `document.createElement(element)`). Links:
[link](https://github.com/spicyj/innerhtml-vs-createelement-vs-clonenode)
[link](https://jsperf.com/clonenode-vs-createelement-performance/58)

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

- in addition to previous note use _Daff's device_ loops unwinding/unrolling in loops with huge amount of iterations:

```javascript
var testVal = 0;
var n = iterations % 8;
while (n--) {
 testVal++;
}

n = (iterations * 0.125) ^ 0; //multiplication is faster than division in some cases
while (n--) {
 testVal++;
 testVal++;
 testVal++;
 testVal++;
 testVal++;
 testVal++;
 testVal++;
 testVal++;
}
```
[link](http://jsperf.com/duffs-device)

- use `Date.now()` instead of `+new Date` or `(new Date).getTime()`;

- don’t use the `with()` statement;

- for clearing arrays use `array.length = 0`, not `array.splice(0)`;

- define arrays for HTML collection objects
As the DOM Level 1 spec says, "collections in the HTML DOM are assumed to be live, meaning that they are automatically updated 
when the underlying document is changed". HTML collection objects are extremely slow, so use any valid technique to minimize 
amount of operations with them;

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

- to prevent memory fragmentation and slower access to array members use instant memory allocation for large array:

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
  
### Additional reading

[Optimization killers](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)

### NB

Doing too much can be as harmful as not doing anything. Too much of a good thing is a bad thing, so avoid _over-optimization_.

> If you optimize everything, you will always be unhappy.

**Donald Knuth**