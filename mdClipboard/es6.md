## Introduction

> ECMAScript 6 is the upcoming version of the ECMAScript standard. This standard is targeting ratification in June 2015\. ES6 is a significant update to the language, and the first update to the language since ES5 was standardized in 2009\. Implementation of these features in major JavaScript engines is [underway now](http://kangax.github.io/es5-compat-table/es6/).

See the [draft ES6 standard](https://people.mozilla.org/%7Ejorendorff/es6-draft.html) for full specification of the ECMAScript 6 language.

## ECMAScript 6 Features

### Arrows

Arrows are a function shorthand using the `=>` syntax. They are syntactically similar to the related feature in C#, Java 8 and CoffeeScript. They support both expression and statement bodies. Unlike functions, arrows share the same lexical `this` as their surrounding code.

<div class="highlight">

    // Expression bodies
        var odds = evens.map(v => v + 1);
        var nums = evens.map((v, i) => v + i);

        // Statement bodies
        nums.forEach(v => {
        if (v % 5 === 0)
        fives.push(v);
        });

        // Lexical this
        var bob = {
        _name: "Bob",
        _friends: [],
        printFriends() {
        this._friends.forEach(f =>
        console.log(this._name + " knows " + f));
        }
        }

</div>

### Classes

ES6 classes are a simple sugar over the prototype-based OO pattern. Having a single convenient declarative form makes class patterns easier to use, and encourages interoperability. Classes support prototype-based inheritance, super calls, instance and static methods and constructors.

<div class="highlight">

    class SkinnedMesh extends THREE.Mesh {
        constructor(geometry, materials) {
        super(geometry, materials);

        this.idMatrix = SkinnedMesh.defaultMatrix();
        this.bones = [];
        this.boneMatrices = [];
        //...
        }
        update(camera) {
        //...
        super.update();
        }
        static defaultMatrix() {
        return new THREE.Matrix4();
        }
        }

</div>

### Enhanced Object Literals

Object literals are extended to support setting the prototype at construction, shorthand for `foo: foo` assignments, defining methods and making super calls. Together, these also bring object literals and class declarations closer together, and let object-based design benefit from some of the same conveniences.

<div class="highlight">

    var obj = {
        // __proto__
        __proto__: theProtoObj,
        // Shorthand for ‘handler: handler’
        handler,
        // Methods
        toString() {
        // Super calls
        return "d " + super.toString();
        },
        // Computed (dynamic) property names
        [ "prop_" + (() => 42)() ]: 42
        };

</div>

> `__proto__` support comes from the JavaScript engine running your program. Although most support the now standard property, [some do not](http://kangax.github.io/compat-table/es6/#__proto___in_object_literals).

### Template Strings

Template strings provide syntactic sugar for constructing strings. This is similar to string interpolation features in Perl, Python and more. Optionally, a tag can be added to allow the string construction to be customized, avoiding injection attacks or constructing higher level data structures from string contents.

<div class="highlight">

    // Basic literal string creation
        `In ES5 "\n" is a line-feed.`

        // Multiline strings
        `In ES5 this is
        not legal.`

        // Interpolate variable bindings
        var name = "Bob", time = "today";
        `Hello ${name}, how are you ${time}?`

        // Construct an HTTP request prefix is used to interpret the replacements and construction
        GET`http://foo.org/bar?a=${a}&b=${b}
        Content-Type: application/json
        X-Credentials: ${credentials}
        { "foo": ${foo},
        "bar": ${bar}}`(myOnReadyStateChangeHandler);

</div>

### Destructuring

Destructuring allows binding using pattern matching, with support for matching arrays and objects. Destructuring is fail-soft, similar to standard object lookup `foo["bar"]`, producing `undefined` values when not found.

<div class="highlight">

    // list matching
        var [a, , b] = [1,2,3];

        // object matching
        var { op: a, lhs: { op: b }, rhs: c }
        = getASTNode()

        // object matching shorthand
        // binds `op`, `lhs` and `rhs` in scope
        var {op, lhs, rhs} = getASTNode()

        // Can be used in parameter position
        function g({name: x}) {
        console.log(x);
        }
        g({name: 5})

        // Fail-soft destructuring
        var [a] = [];
        a === undefined;

        // Fail-soft destructuring with defaults
        var [a = 1] = [];
        a === 1;

</div>

### Default + Rest + Spread

Callee-evaluated default parameter values. Turn an array into consecutive arguments in a function call. Bind trailing parameters to an array. Rest replaces the need for `arguments` and addresses common cases more directly.

<div class="highlight">

    function f(x, y=12) {
        // y is 12 if not passed (or passed as undefined)
        return x + y;
        }
        f(3) == 15

</div>

<div class="highlight">

    function f(x, ...y) {
        // y is an Array
        return x * y.length;
        }
        f(3, "hello", true) == 6

</div>

<div class="highlight">

    function f(x, y, z) {
        return x + y + z;
        }
        // Pass each elem of array as argument
        f(...[1,2,3]) == 6

</div>

### Let + Const

Block-scoped binding constructs. `let` is the new `var`. `const` is single-assignment. Static restrictions prevent use before assignment.

<div class="highlight">

    function f() {
        {
        let x;
        {
        // okay, block scoped name
        const x = "sneaky";
        // error, const
        x = "foo";
        }
        // error, already declared in block
        let x = "inner";
        }
        }

</div>

### Iterators + For..Of

Iterator objects enable custom iteration like CLR IEnumerable or Java Iterable. Generalize `for..in` to custom iterator-based iteration with `for..of`. Don’t require realizing an array, enabling lazy design patterns like LINQ.

<div class="highlight">

    let fibonacci = {
        [Symbol.iterator]() {
        let pre = 0, cur = 1;
        return {
        next() {
        [pre, cur] = [cur, pre + cur];
        return { done: false, value: cur }
        }
        }
        }
        }

        for (var n of fibonacci) {
        // truncate the sequence at 1000
        if (n > 1000)
        break;
        console.log(n);
        }

</div>

Iteration is based on these duck-typed interfaces (using [TypeScript](http://typescriptlang.org) type syntax for exposition only):

<div class="highlight">

    interface IteratorResult {
        done: boolean;
        value: any;
        }
        interface Iterator {
        next(): IteratorResult;
        }
        interface Iterable {
        [Symbol.iterator](): Iterator
        }

</div>

> #### Support via polyfill
> 
> In order to use Iterators you must include the Babel [polyfill](/docs/usage/polyfill).

### Generators

Generators simplify iterator-authoring using `function*` and `yield`. A function declared as function* returns a Generator instance. Generators are subtypes of iterators which include additional `next` and `throw`. These enable values to flow back into the generator, so `yield` is an expression form which returns a value (or throws).

Note: Can also be used to enable ‘await’-like async programming, see also ES7 `await` [proposal](https://github.com/lukehoban/ecmascript-asyncawait).

<div class="highlight">

    var fibonacci = {
        [Symbol.iterator]: function*() {
        var pre = 0, cur = 1;
        for (;;) {
        var temp = pre;
        pre = cur;
        cur += temp;
        yield cur;
        }
        }
        }

        for (var n of fibonacci) {
        // truncate the sequence at 1000
        if (n > 1000)
        break;
        console.log(n);
        }

</div>

The generator interface is (using [TypeScript](http://typescriptlang.org) type syntax for exposition only):

<div class="highlight">

    interface Generator extends Iterator {
        next(value?: any): IteratorResult;
        throw(exception: any);
        }

</div>

> #### Support via polyfill
> 
> In order to use Generators you must include the Babel [polyfill](/docs/usage/polyfill).

### Comprehensions

Array and generator comprehensions provide simple declarative list processing similar as used in many functional programming patterns.

<div class="highlight">

    // Array comprehensions
        var results = [
        for(c of customers)
        if (c.city == "Seattle")
        { name: c.name, age: c.age }
        ]

        // Generator comprehensions
        var results = (
        for(c of customers)
        if (c.city == "Seattle")
        { name: c.name, age: c.age }
        )

</div>

> #### Disabled by default
> 
> These are only available if you enable experimental support. See [experimental usage](/docs/usage/experimental) for more information.

### Unicode

Non-breaking additions to support full Unicode, including new unicode literal form in strings and new RegExp `u` mode to handle code points, as well as new APIs to process strings at the 21bit code points level. These additions support building global apps in JavaScript.

<div class="highlight">

    // same as ES5.1
        "𠮷".length == 2

        // new RegExp behaviour, opt-in ‘u’
        "𠮷".match(/./u)[0].length == 2

        // new form
        "\u{20BB7}" == "𠮷" == "\uD842\uDFB7"

        // new String ops
        "𠮷".codePointAt(0) == 0x20BB7

        // for-of iterates code points
        for(var c of "𠮷") {
        console.log(c);
        }

</div>

### Modules

Language-level support for modules for component definition. Codifies patterns from popular JavaScript module loaders (AMD, CommonJS). Runtime behaviour defined by a host-defined default loader. Implicitly async model – no code executes until requested modules are available and processed.

<div class="highlight">

    // lib/math.js
        export function sum(x, y) {
        return x + y;
        }
        export var pi = 3.141593;

</div>

<div class="highlight">

    // app.js
        import * as math from "lib/math";
        alert("2π = " + math.sum(math.pi, math.pi));

</div>

<div class="highlight">

    // otherApp.js
        import {sum, pi} from "lib/math";
        alert("2π = " + sum(pi, pi));

</div>

Some additional features include `export default` and `export *`:

<div class="highlight">

    // lib/mathplusplus.js
        export * from "lib/math";
        export var e = 2.71828182846;
        export default function(x) {
        return Math.exp(x);
        }

</div>

<div class="highlight">

    // app.js
        import exp, {pi, e} from "lib/mathplusplus";
        alert("2π = " + exp(pi, e));

</div>

> #### Module Formatters
> 
> Babel can transpile ES6 Modules to several different formats including Common.js, AMD, System, and UMD. You can even create your own. For more details see the [modules docs](/docs/usage/modules).

### Module Loaders

Module loaders support:

*   Dynamic loading
*   State isolation
*   Global namespace isolation
*   Compilation hooks
*   Nested virtualization

The default module loader can be configured, and new loaders can be constructed to evaluated and load code in isolated or constrained contexts.

<div class="highlight">

    // Dynamic loading – ‘System’ is default loader
        System.import("lib/math").then(function(m) {
        alert("2π = " + m.sum(m.pi, m.pi));
        });

        // Create execution sandboxes – new Loaders
        var loader = new Loader({
        global: fixup(window) // replace ‘console.log’
        });
        loader.eval("console.log(\"hello world!\");");

        // Directly manipulate module cache
        System.get("jquery");
        System.set("jquery", Module({$: $})); // WARNING: not yet finalized

</div>

> #### Additional polyfill needed
> 
> Since Babel defaults to using common.js modules, it does not include the polyfill for the module loader API. Get it [here](https://github.com/ModuleLoader/es6-module-loader).

> #### Using Module Loader
> 
> In order to use this, you'll need to tell Babel to use the `system` module formatter. Also be sure to check out [System.js](https://github.com/systemjs/systemjs)

### Map + Set + WeakMap + WeakSet

Efficient data structures for common algorithms. WeakMaps provides leak-free object-key’d side tables.

<div class="highlight">

    // Sets
        var s = new Set();
        s.add("hello").add("goodbye").add("hello");
        s.size === 2;
        s.has("hello") === true;

        // Maps
        var m = new Map();
        m.set("hello", 42);
        m.set(s, 34);
        m.get(s) == 34;

        // Weak Maps
        var wm = new WeakMap();
        wm.set(s, { extra: 42 });
        wm.size === undefined

        // Weak Sets
        var ws = new WeakSet();
        ws.add({ data: 42 });
        // Because the added object has no other references, it will not be held in the set

</div>

> #### Support via polyfill
> 
> In order to support Maps, Sets, WeakMaps, and WeakSets in all environments you must include the Babel [polyfill](/docs/usage/polyfill).

### Proxies

Proxies enable creation of objects with the full range of behaviors available to host objects. Can be used for interception, object virtualization, logging/profiling, etc.

<div class="highlight">

    // Proxying a normal object
        var target = {};
        var handler = {
        get: function (receiver, name) {
        return `Hello, ${name}!`;
        }
        };

        var p = new Proxy(target, handler);
        p.world === "Hello, world!";

</div>

<div class="highlight">

    // Proxying a function object
        var target = function () { return "I am the target"; };
        var handler = {
        apply: function (receiver, ...args) {
        return "I am the proxy";
        }
        };

        var p = new Proxy(target, handler);
        p() === "I am the proxy";

</div>

There are traps available for all of the runtime-level meta-operations:

<div class="highlight">

    var handler =
        {
        get:...,
        set:...,
        has:...,
        deleteProperty:...,
        apply:...,
        construct:...,
        getOwnPropertyDescriptor:...,
        defineProperty:...,
        getPrototypeOf:...,
        setPrototypeOf:...,
        enumerate:...,
        ownKeys:...,
        preventExtensions:...,
        isExtensible:...
        }

</div>

> #### Unsupported feature
> 
> Due to the limitations of ES5, Proxies cannot be transpiled or polyfilled. See support from various [JavaScript engines](http://kangax.github.io/compat-table/es6/#Proxy).

### Symbols

Symbols enable access control for object state. Symbols allow properties to be keyed by either `string` (as in ES5) or `symbol`. Symbols are a new primitive type. Optional `name` parameter used in debugging - but is not part of identity. Symbols are unique (like gensym), but not private since they are exposed via reflection features like `Object.getOwnPropertySymbols`.

<div class="highlight">

    (function() {

        // module scoped symbol
        var key = Symbol("key");

        function MyClass(privateData) {
        this[key] = privateData;
        }

        MyClass.prototype = {
        doStuff: function() {
        ... this[key] ...
        }
        };

        })();

        var c = new MyClass("hello")
        c["key"] === undefined

</div>

> #### Support via polyfill
> 
> In order to support Symbols you must include the Babel [polyfill](/docs/usage/polyfill).

### Subclassable Built-ins

In ES6, built-ins like `Array`, `Date` and DOM `Element`s can be subclassed.

<div class="highlight">

    // User code of Array subclass
        class MyArray extends Array {
        constructor(...args) { super(...args); }
        }

        var arr = new MyArray();
        arr[1] = 12;
        arr.length == 2

</div>

> #### Partial support
> 
> Built-in subclassability should be evaluated on a case-by-case basis as classes such as `HTMLElement` **can** be subclassed while many such as `Date`, `Array` and `Error` **cannot** be due to ES5 engine limitations.

### Math + Number + String + Object APIs

Many new library additions, including core Math libraries, Array conversion helpers, and Object.assign for copying.

<div class="highlight">

    Number.EPSILON
        Number.isInteger(Infinity) // false
        Number.isNaN("NaN") // false

        Math.acosh(3) // 1.762747174039086
        Math.hypot(3, 4) // 5
        Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2

        "abcde".includes("cd") // true
        "abc".repeat(3) // "abcabcabc"

        Array.from(document.querySelectorAll("*")) // Returns a real Array
        Array.of(1, 2, 3) // Similar to new Array(...), but without special one-arg behavior
        [0, 0, 0].fill(7, 1) // [0,7,7]
        [1,2,3].findIndex(x => x == 2) // 1
        ["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
        ["a", "b", "c"].keys() // iterator 0, 1, 2
        ["a", "b", "c"].values() // iterator "a", "b", "c"

        Object.assign(Point, { origin: new Point(0,0) })

</div>

> #### Limited support from polyfill
> 
> Most of these APIs are supported by the Babel [polyfill](/docs/usage/polyfill). However, certain features are omitted for various reasons (e.g. `String.prototype.normalize` needs a lot of additional code to support). You can find more polyfills [here](https://github.com/addyosmani/es6-tools#polyfills).

### Binary and Octal Literals

Two new numeric literal forms are added for binary (`b`) and octal (`o`).

<div class="highlight">

    0b111110111 === 503 // true
        0o767 === 503 // true

</div>

> #### Only supports literal form
> 
> Babel is only able to transform `0o767` and not `Number("0o767")`.

### Promises

Promises are a library for asynchronous programming. Promises are a first class representation of a value that may be made available in the future. Promises are used in many existing JavaScript libraries.

<div class="highlight">

    function timeout(duration = 0) {
        return new Promise((resolve, reject) => {
        setTimeout(resolve, duration);
        })
        }

        var p = timeout(1000).then(() => {
        return timeout(2000);
        }).then(() => {
        throw new Error("hmm");
        }).catch(err => {
        return Promise.all([timeout(100), timeout(200)]);
        })

</div>

> #### Support via polyfill
> 
> In order to support Promises you must include the Babel [polyfill](/docs/usage/polyfill).

### Reflect API

Full reflection API exposing the runtime-level meta-operations on objects. This is effectively the inverse of the Proxy API, and allows making calls corresponding to the same meta-operations as the proxy traps. Especially useful for implementing proxies.

<div class="highlight">

    var O = {a: 1};
        Object.defineProperty(O, 'b', {value: 2});
        O[Symbol('c')] = 3;

        Reflect.ownKeys(O); // ['a', 'b', Symbol(c)]

        function C(a, b){
        this.c = a + b;
        }
        var instance = Reflect.construct(C, [20, 22]);
        instance.c; // 42

</div>

> #### Support via polyfill
> 
> In order to use the Reflect API you must include the Babel [polyfill](/docs/usage/polyfill).

### Tail Calls

Calls in tail-position are guaranteed to not grow the stack unboundedly. Makes recursive algorithms safe in the face of unbounded inputs.

<div class="highlight">

    function factorial(n, acc = 1) {
        "use strict";
        if (n <= 1) return acc;
        return factorial(n - 1, n * acc);
        }

        // Stack overflow in most implementations today,
        // but safe on arbitrary inputs in eS6
        factorial(100000)

</div>

> #### Partial support
> 
> Only explicit self referencing tail recursion is supported due to the complexity and performance impact of supporting tail calls globally.