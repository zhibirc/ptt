# ptt
## Performance Tips and Tricks for effective development on JavaScript

_This is an attempt to collect various JavaScript optimizations and approaches for achievement better performance. 
Most of them work well in most major browsers, but some may be browser specific/JavaScript engine specific.
Some of them have links to tests and other additional information._

### Caching

Advices as for caching elements are make sense in case multiple usages and/or long prototype chain
	
- Cache DOM elements

```javascript
var doc = document;
```

- Cache repeatedly evaluated properties

```javascript
var len = list.length;
```

- Cache function references, property values

```javascript
var hasOwn = Object.hasOwnProperty
```

- Use additional/subtractional/... assignment ( `+=`, `-=`, ...) instead increment/decrement (`++`, `--`). 
It does the same things more straightforward and hasn't ambiguous behavior (prefix and postfix notation of increment/decrement)

- Avoid global variables or store the copies in local scopes

- Avoid long prototype inheritance

- Avoid to change prototype chain in runtime

- Closures can increase memory usage due to the storing it's scope in the stack of execution contexts, so use it wisely

- Use literal notation for constructing objects (`/regexp/`, `[]`, `{}`, `true`/`false` etc.)
	
- If it's necessary use `new Function` instead of `eval`
	
- Avoid using `unshift` on large arrays
	
- for-in loop is very, very slow, use `Object.keys` + `forEach`/`for`/`while`
	
- Use bitwise operators ( 4 / 2 === 4 >> 1, 4 * 2 === 4 << 1, ~~ for rounding etc.). It may be less readable but sometimes increase performance
	
- Avoid using captured groups in regular expressions if you don't want to save result of matching,
use `(?:)` - non-capturing groups instead

- Use sets of simple expressions instead of one overcomplicated

- Use explicit parts and anchors in RegExp instead of unnecessary implicit, when possible (`/^function\s/` instead of `/[^\s]\s/`)

- Create/compile RegExp before its multiple usage

- Use `===` (strict equality operator) instead of `==`

- Use `switch-case` construction instead of large `if-else` structure

- Prefer `call` rather than `apply`, as `call` is slightly faster

- Use cloning objects istead of creating (`element.cloneNode()` instead of `document.createElement(element)`)

Link:
https://github.com/spicyj/innerhtml-vs-createelement-vs-clonenode
https://jsperf.com/clonenode-vs-createelement-performance/58

- Multiplication is faster than division in some cases, so you can increase performance of expression (n / 8) with (n * 0.125)

- In addition to previous note use Daff's device loops unwinding/unrolling in loops with huge amount of iterations

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

### Additional literature

Stoyan Stefanov "JavaSctipt Design Patterns"
Nicholas Zakas "High performant JavaScript"
Douglas Crockford "JavaScript: the good parts"
