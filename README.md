![JS Logo](performance.jpg)
# ptt
## Performance Tips and Tricks for effective development on JavaScript

_This is an attempt to collect various JavaScript optimizations and approaches for achievement better performance. 
Most of them work well in most major browsers, but some may be browser specific/JavaScript engine specific.
Some of them have links to tests and other additional information._

### Caching

Advices as for caching elements are make sense in case multiple usages and/or long prototype chain
	
- Cache DOM elements:

```javascript
var doc = document,
    div = doc.getElementById('id_0'),
    p   = doc.getElementById('id_1');
```

- Cache repeatedly evaluated properties:

```javascript
var len = list.length;
```

- Cache function references, property values:

```javascript
var hasOwn = Object.hasOwnProperty;
```

- Use additional/subtractional/... assignment ( `+=`, `-=`, ...) instead increment/decrement (`++`, `--`). 
It does the same things more straightforward and hasn't ambiguous behaviour (prefix and postfix notation of increment/decrement)
Oh yeah, and it's almost twice as fast in Gecko and slightly faster in WebKit.

- Avoid global variables or store the copies in local scope to avoid digging into scope chain

- Avoid long prototype inheritance. Sometimes it's useful to use so-called _dict pattern_ and create objects without prototype chain which prevents undesired property lookups:

```javascript
var obj = Object.create(null);
```

- Avoid to change prototype chain in runtime

- Closures can increase memory usage due to the storing it's scope in the stack of execution contexts, so use it wisely

- Use literal notation for constructing objects (`/regexp/`, `[]`, `{}`, `true`/`false` etc.)
	
- If it's necessary use `new Function` instead of `eval`
	
- Avoid using `unshift` on large arrays
	
- for-in loop is very, very slow, use `Object.keys` + `forEach`/`for`/`while`
	
- Use bitwise operators ( `4 / 2 === 4 >> 1`, `4 * 2 === 4 << 1`, `~~` for rounding etc.). It may be less readable but sometimes increase performance
	
- Avoid using captured groups in regular expressions if you don't want to save result of matching,
use `(?:)` - non-capturing groups instead

- Use sets of simple expressions instead of one overcomplicated

- Use explicit parts and anchors in RegExp instead of unnecessary implicit, when possible (`/^function\s/` instead of `/[^\s]\s/`)

- Create/compile RegExp before its multiple usage

- Use `===` (strict equality operator) instead of `==`

- Use `switch-case` construction instead of large `if-else` structure

- Use mapping instead of `switch-case` in case of simple data accessing for elegance and performance:

```javascript
var weight = 'some',
    map = {
        light: '2kg',
        middle: '4kg',
        heavy: '6kg'
    };

console.log(map[weight]);
```

- Prefer `call` rather than `apply`, as `call` is slightly faster

- Use cloning objects istead of creating (`element.cloneNode()` instead of `document.createElement(element)`). Links:
https://github.com/spicyj/innerhtml-vs-createelement-vs-clonenode
https://jsperf.com/clonenode-vs-createelement-performance/58

- Avoid performance pitfalls with _long running scrips_ with the following technique (use this if the loop doesn't have to execute synchronously and 
the order in which the loop’s data is processed in no matter:

```javascript
function listAsyncExec (list, executor, interval, context) {
    var list = list.slice();
    setTimeout(function _() {
        executor.call(context, list.shift());
        
        list.length && setTimeout(_, interval);
    }, interval);
}
```

- Multiplication is faster than division in some cases, so you can increase performance of expression (`n / 8`) with (`n * 0.125`)

- In addition to previous note use _Daff's device_ loops unwinding/unrolling in loops with huge amount of iterations:

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
Link: http://jsperf.com/duffs-device

- Use `Date.now()` instead of `+new Date` or `(new Date).getTime()`

- Don’t use the `with()` statement

- For clearing arrays use `array.length = 0`, not `array.splice(0)`

- Define arrays for HTML collection objects
As the DOM Level 1 spec says, "collections in the HTML DOM are assumed to be live, meaning that they are automatically updated when the underlying document is changed". 
HTML collection objects are extremely slow, so use any valid technique to minimize amount of operations with them.

- Don't touching the DOM without real necessity, use `documentFragment` or other technique which minimize amount of _reflows_

- Use CSS classes instead of individual styles to change a number of styles at once, which incurs a single _reflow_

- If it's necessary to define sufficient amount of styles in runtime the approach below is appropriate (or use external CSS file via `link`):

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

- Keep in mind differences in performance of `textContent` and `innerText`, test them before use

- If the functions in your application tend to be run again and again, the performance improvement will be greater 
than if many different functions tend to run only once, so use function wrappers only for repeated code blocks

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

- Remember about _tail call optimization_, for example:

```javascript
function evalFactorial (n) {
	function _(n, acc) {
		return n < 2 ? acc : _(n - 1, n * acc);
	}
	 
	return _(n, 1);
};
```

- To prevent memory fragmentation and slower access to array members use instant memory allocation for large array:

```javascript
var db = Array(1e6);
```

- You can save some memory by using one less variable when swapping values, compare:

```javascript
var c = a; a = b; b = c;
a ^= b; b ^= a; a ^= b;
```

- Use _memoization_ as a useful optimization technique for caching the results of function calls 
such that lengthy lookups or expensive recursive computations can be minimized where possible