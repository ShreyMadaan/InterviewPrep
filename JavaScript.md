## Question 1:
    What is the difference between var, let, and const?
    
    Beginner: var allows redeclaration and reassignment but ignores block scope. let allows reassignment but not redeclaration, and respects block scope. const allows neither. Default to const; use let only when the value must change.
    
    Experienced: The deeper distinction is scope and hoisting. var is function-scoped and hoisted with an initial value of undefined, so it can be read before its declaration line without error (just returns undefined). let and const are block-scoped and hoisted into a temporal dead zone, throwing a ReferenceError if accessed early. This predictability in nested blocks (loops, conditionals) is the real reason the language moved away from var, not just the redeclaration footgun. In practice, linting rules like no-var enforce this at the tooling level across production codebases.
## Question 2:
    What is the difference between == and ===?
    
    Beginner: == compares values after type coercion (converting to a common type). === compares both type and value with no conversion. Always use === to avoid surprises like 0 == false being true.
    
    Experienced: The coercion rules behind == follow the Abstract Equality Comparison algorithm in the ECMAScript spec, which has inconsistent special cases (null == undefined is true, but null == 0 is false). Relying on == means memorizing that algorithm correctly every time. === has zero ambiguity: different types means instant false. Production codebases ban == entirely via linter rules (e.g., eqeqeq in ESLint), eliminating a whole category of subtle bugs at zero cost.
## Question 3:
    Why does typeof null return "object"?
    
    Beginner: It is a bug from 1995. null is a primitive, not an object, but typeof has always misreported it. It was never fixed because too much existing web code depends on this behavior.

    Experienced: In the original C implementation of JavaScript, values used a type tag stored in their lower bits. The tag for objects was 0, and null was represented as the NULL pointer (0x00), so its tag was also 0. typeof simply read the tag. TC39 (the JavaScript standards body) treats web compatibility as near-inviolable: if even a fraction of the web relies on current behavior, breaking it is considered worse than the bug itself. This is a textbook case of how legacy decisions calcify in widely deployed languages.
## Question 4:
    What is the difference between primitive and reference types when copying?
    
    Beginner: Copying a primitive creates a fully independent value; changing the copy never affects the original. Copying a reference type copies the memory address, so both variables point to the same data. Changing one affects the other.
    
    Experienced: This distinction is the foundation for understanding mutability bugs in larger applications. In frameworks like React, state updates rely on creating new references (not mutating existing ones) because the framework compares references to decide whether to re-render. The practical fix for independent copies is the spread operator ({...obj} or [...arr]) for shallow copies, or structured cloning (structuredClone()) for deep copies when nested structures also need independence.
## Question 5:
    What happens if a function has no return statement?
    
    Beginner: It returns undefined automatically. The function can still do work inside (like logging), but the caller gets nothing usable unless return is explicitly used.
    
    Experienced: This connects to a broader principle: console.log is a side effect (it interacts with something outside the function), while return produces a value the program can compose with. A function with no return is sometimes intentional (its purpose is a side effect, like updating the DOM). The mistake is using such a function in an expression expecting a value, e.g., const result = logMessage();, which silently sets result to undefined with no error. This is a common source of confusing bugs when beginners transition from console-driven experimentation to writing functions that feed into each other.
## Question 6:
    What is the difference between the creation phase and the execution phase?
    
    Beginner Answer: JavaScript processes code in two passes. In the creation phase, it scans all declarations and registers them in memory without executing anything. In the execution phase, it runs the code line by line, performing assignments and function calls. This is why you can use function declarations before they appear in your code.
    
    Experienced Answer: The creation phase is part of building an execution context. Each execution context (global, or per function call) goes through its own creation phase where the engine performs three tasks: (1) creating the variable environment (registering var declarations with undefined, registering let/const in the TDZ), (2) storing function declarations in their entirety, and (3) establishing the scope chain by linking to the outer lexical environment. This per-context creation is why a var inside a function gets its own undefined, separate from any global variable with the same name. It also explains why function expressions are not fully hoisted: the variable name is registered, but the function body is a runtime value assigned during execution, just like any other right-hand-side expression.


## Question 7:
    What is the Temporal Dead Zone, and why does it exist?
    
    Beginner Answer: The Temporal Dead Zone (TDZ) is the period between when a let or const variable is hoisted and when it is actually declared in the code. Accessing the variable during this period throws a ReferenceError. It exists to prevent bugs from using variables before they are properly initialized.
    
    Experienced Answer: The TDZ is a deliberate design decision in ES6. let and const are technically hoisted (the engine knows they exist in the current scope), but they are placed in an uninitialized state rather than being set to undefined like var. This distinction matters because it enforces a stricter "declare before use" discipline. The alternative, the var approach of silently giving you undefined, has been a source of real-world bugs for decades: you get no signal that you are reading a variable too early. The TDZ makes that a hard error. It also exists per-scope, not per-file: a let in a block enters TDZ at the top of that block, not at the top of the file. This is important when reasoning about let inside conditionals or loops where the TDZ starts and ends with the block itself.


## Question 8:
    Why does var ignore block scope, and why does that matter?
    
    Beginner Answer: var was designed to be function-scoped, which means it only recognizes function boundaries. It treats if, for, and while blocks as if the curly braces are not there. This means a var declared inside a loop or conditional is accessible outside of it, which can cause unexpected bugs. let and const respect block scope and stay inside their curly braces.
    
    Experienced Answer: This is a historical artifact from JavaScript's original design in 1995. At the time, JavaScript only had var and function scope. Block scope simply did not exist in the language. When you write var x inside a for loop, the engine attaches x to the nearest enclosing function (or global scope), not to the loop block. This creates several well-known problems: loop counter leakage, accidental overwrites when an if block re-declares a variable, and the classic closure-in-a-loop bug with setTimeout. ES6 introduced let and const specifically to give developers block-scoped declarations. In production codebases, the ESLint no-var rule bans var entirely, eliminating this entire category of bugs at the linting stage.


## Question 9:
    What is the scope chain, and how does JavaScript resolve variable lookups?
    
    Beginner Answer: The scope chain is the path JavaScript follows to find a variable. It starts in the current scope, then checks the parent scope, then the grandparent scope, and so on until it reaches the global scope. If the variable is not found anywhere, it throws a ReferenceError. Inner scopes can access outer variables, but outer scopes cannot access inner variables.
    
    Experienced Answer: The scope chain is a linked list of lexical environments. Each execution context holds a reference to its outer lexical environment, established during the creation phase based on where the function is defined in the source code (lexical scoping). When the engine resolves a variable, it walks this chain from the current environment outward. Two subtleties matter here. First, variable shadowing: if the same name exists in both an inner and outer scope, the inner one is found first and the lookup stops, effectively hiding the outer one. Second, and this is critical for interviews, the chain is lexical, not dynamic. A function's outer environment is determined by where it is written, not where it is called. This means if you define a function in the global scope and call it from inside another function, the called function's scope chain goes directly to global, not through the calling function. This lexical resolution is the mechanism that makes closures work: a returned inner function retains its reference to the outer lexical environment even after the outer function has returned.


## Question 10:
    What is the difference between a function declaration and a function expression in terms of hoisting?
    
    Beginner Answer: A function declaration (e.g., function greet() {}) is fully hoisted. You can call it before its declaration line because the entire function body is stored during the creation phase. A function expression (e.g., var greet = function() {}) is only partially hoisted. The variable name is hoisted (to undefined for var, or into the TDZ for let/const), but the function body is not stored until execution reaches that line.
    
    Experienced Answer: The distinction comes down to what the creation phase does with each form. For a function declaration, the engine stores the entire function object in memory during creation, making it immediately invocable. For a function expression, the engine sees a variable declaration and processes only that: var gets undefined, let/const enter the TDZ. The right-hand side (the actual function) is a runtime value, evaluated during the execution phase, no different from assigning a number or string. This has a practical consequence: calling a var function expression early gives TypeError: x is not a function (because you are calling undefined), while calling a let/const function expression early gives ReferenceError (TDZ violation). These are different errors with different causes, and distinguishing them in a debugging scenario demonstrates a solid understanding of the execution model. In practice, many teams use const with arrow functions for all function expressions, which means any accidental early call fails loudly at the TDZ boundary.


## Question 11:
    What is lexical scoping, and how is it different from dynamic scoping?
    
    Beginner Answer: Lexical scoping means the scope chain is determined by where functions are written in the source code, not by where they are called. If a function is defined in the global scope, its parent scope is always global, regardless of which function calls it.
    
    Experienced Answer: In lexical (static) scoping, the outer environment reference is fixed at parse time based on the nesting structure of the source code. In dynamic scoping (used by some languages like older versions of Bash and Emacs Lisp), the scope chain follows the call stack, so a variable lookup depends on which function called the current one. JavaScript uses lexical scoping exclusively. This design is what enables closures to be predictable: a function captures the variables from its definition site, not its call site. When interviewers ask about lexical scope, they are often setting up a follow-up question about closures. The key phrase to use is: "A function remembers the environment in which it was created, not the environment in which it is invoked."

## Question 12:
    What is a closure in JavaScript?
    
    Beginner Answer: A closure is when a function remembers and can access variables from its outer scope even after the outer function has returned. The inner function "closes over" those variables.
    
    Experienced Answer: A closure is the combination of a function and a reference to its lexical environment. Every function in JavaScript creates a closure at creation time, but we colloquially refer to closures when an inner function outlives its enclosing execution context, retaining access to the outer scope's variable bindings. Engines like V8 optimize this by only retaining variables the inner function actually references, not the entire outer scope. Closures are the mechanism behind the module pattern, data privacy, partial application, and event handler state management.


## Question 13:
    What is the output of this code and why?
    
    function outer() {
    let count = 0;
    return function() {
        return ++count;
    };
    }
    const a = outer();
    const b = outer();
    console.log(a()); // ?
    console.log(a()); // ?
    console.log(b()); // ?
    
    Beginner Answer: Output is 1, 2, 1. a and b are created from separate calls to outer(), so they have their own count variables. a() increments its count twice, and b() starts from 0 with its own count.
    
    Experienced Answer: Each invocation of outer() creates a new execution context with its own count binding. The returned function forms a closure over that specific binding. a and b are closures over different instances of count. This demonstrates that closures capture variable bindings, not values. If they captured values, count would always be 0. The fact that count persists and mutates proves the closure holds a live reference to the variable in the preserved scope.


## Question 14:
    What is the difference between a function declaration and a function expression?
    
    Beginner Answer: A function declaration uses the function keyword as a statement and is hoisted, so you can call it before it appears in code. A function expression assigns a function to a variable and is not hoisted.
    
    Experienced Answer: Function declarations are hoisted in their entirety during the creation phase of the execution context, meaning both the name binding and the function body are available before any code runs. Function expressions, whether using const, let, or var, follow the hoisting rules of their variable keyword. With var, the identifier is hoisted but initialized to undefined, causing a TypeError if called early (since undefined is not callable). With let/const, the identifier is in the TDZ, causing a ReferenceError. This distinction matters in conditional logic: function declarations inside blocks have inconsistent behavior across engines, while function expressions behave predictably.


## Question 15:
    Why does this code print 3, 3, 3 and how do you fix it?
    
    for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
    }
    
    Beginner Answer: var is function-scoped, so there is only one i. By the time the callbacks run, the loop is done and i is 3. Fix it by using let instead of var, which creates a new i for each iteration.
    
    Experienced Answer: var declarations are scoped to the nearest function (or global scope), so all three iterations share a single i binding. The setTimeout callbacks are closures over this shared binding. When the event loop processes the callbacks (after the synchronous loop completes), they all read the same i, which is now 3. Using let fixes this because let is block-scoped and the spec requires the runtime to create a new binding for each loop iteration (Section 14.7.4.2 of the ECMAScript spec). The pre-ES6 fix is an IIFE that captures the current value of i as a parameter, creating a new scope per iteration.


## Question 16:
    How do closures enable private variables in JavaScript?
    
    Beginner Answer: You can define variables inside a function and return an object with methods that access those variables. The variables are not accessible from outside, only through the returned methods.
    
    Experienced Answer: Closures simulate encapsulation by exploiting scope visibility. Variables declared in the outer function are not on any object, so they cannot be accessed via property lookup. The only way to interact with them is through the returned functions that close over them. This is stronger than convention-based privacy (like _private naming) and was the standard pattern before ES2022's #privateField syntax. Unlike #private, closure-based privacy works with functions and does not require classes. The trade-off is that each instance creates new function objects in memory (no prototype sharing), which matters at scale.


## Question 17:
    Can you explain what "lexical scope" means?
    
    Beginner Answer: Lexical scope means that a function can access variables from where it was written in the code, not where it is called. If a function is written inside another function, it can access the outer function's variables.
    
    Experienced Answer: Lexical (or static) scope means the scope of a variable is determined at parse time by its position in the source code. The alternative is dynamic scope (used in some Lisps and Bash), where variable resolution depends on the call stack at runtime. In JavaScript, when the engine encounters a variable reference, it walks the scope chain, which is a series of lexical environments linked by outer environment references. This chain is established when the function is created, not when it is called. This is precisely why closures work: the scope chain is baked into the function object at creation time and does not change regardless of where or how the function is invoked.


## Question 18:
    What does it mean that closures capture variables "by reference"?
    
    Beginner Answer: It means that a closure does not copy the value of a variable. It keeps a link to the actual variable. If the variable changes, the closure sees the updated value.
    
    Experienced Answer: When we say closures capture by reference, we mean the closure retains access to the variable binding itself, not a snapshot of its value at closure creation time. This is observable in two ways: (1) if the outer variable changes after the closure is created, the closure sees the new value, and (2) if the closure modifies the variable, the change is visible to other closures over the same scope. This is fundamentally different from, say, C++ lambdas which can capture by value or by reference explicitly. In JavaScript, it is always by reference. This behavior is what makes closures useful for persistent state (the variable can be mutated across calls) but also what causes the classic loop bug with var. Understanding this distinction is essential for reasoning about any code that uses closures.

## Question 19:
    What is a higher-order function? How is it different from a regular function?
    
    Beginner Answer: A higher-order function is a function that either takes another function
    as an argument, or returns a function, or both. Regular functions just take data values and
    return data values.
    function greet(name) { return "Hello " + name; } // regular
    function repeat(fn, times) { // higher-order
    for (let i = 0; i < times; i++) fn();
    }
    
    Experienced Answer: Higher-order functions are a direct consequence of functions being
    first-class citizens in JavaScript, meaning functions are values that can be assigned to
    variables, passed as arguments, and returned from other functions. HOFs enable powerful
    composition patterns. Array.prototype.map, filter, and reduce are the most
    commonly used built-in HOFs. In real codebases, HOFs appear as middleware chains
    (Express.js), event handler registrations, decorator patterns, and function composition (like
    Redux's compose). Understanding HOFs is the bridge to functional programming idioms in
    JavaScript: currying, partial application, and point-free style all rely on functions returning
    functions.

## Question 20:
    What will the following code log, and why?
    console.log(a);
    console.log(b);
    var a = 10;
    let b = 20;
    
    Beginner Answer: The first console.log(a) will print undefined because var
    declarations are hoisted to the top but not their values. The second console.log(b) will
    throw a ReferenceError because let is not accessible before its declaration.
    
    Experienced Answer: Both var and let declarations are hoisted, meaning the JavaScript
    engine is aware of them during the creation phase of the execution context. The difference is
    in initialization. var variables are initialized with undefined during hoisting, so accessing
    them before the assignment line returns undefined. let (and const) variables are hoisted
    but remain in the Temporal Dead Zone (TDZ) from the start of the block until the
    declaration line is executed. Any access during the TDZ throws a ReferenceError. This is a
    deliberate design choice in ES6 to catch bugs. The TDZ also applies to const. In practice,
    always declare variables at the top of their scope and prefer const by default, let when
    reassignment is needed, and avoid var entirely.

## Question 21:
    What is the difference between passing a primitive and an object to a function?
    
    Beginner Answer: Primitives (number, string, boolean) are passed by value, so the function
    gets a copy. Changing it inside the function does not affect the original. Objects are passed by
    reference, so the function gets a reference to the same object, and changes inside the
    function do affect the original.
    
    Experienced Answer: The precise terminology is "pass by value," but for objects, the value
    being passed is a reference (a pointer to the location in heap memory). This means you can
    mutate the object's properties through the reference, and the caller will see those changes.
    However, if you reassign the parameter to a completely new object inside the function, the
    original variable in the calling scope is unaffected, because you have only overwritten the
    local copy of the reference.
    function modify(obj) {
    obj.name = "changed"; // mutates the original
    obj = { name: "new" }; // reassigns local reference only
    }
    let person = { name: "original" };
    modify(person);
    console.log(person.name); // "changed", not "new"
    This distinction is critical when designing functions. If you need to avoid side effects, clone
    the object (Object.assign, spread operator, or structuredClone) before mutating.

## Question 22:
    Why should you follow the DRY principle, and when can it go wrong?
    
    Beginner Answer: DRY stands for "Don't Repeat Yourself." If you have the same logic in
    multiple places, you should move it into a single function and call that function wherever
    needed. This makes your code shorter and easier to maintain.
    
    Experienced Answer: DRY reduces the surface area for bugs: when logic exists in one place,
    fixing it once fixes it everywhere. It also improves readability because well-named extracted
    functions communicate intent. However, premature DRY (sometimes called "wrong
    abstraction") can be worse than duplication. If two pieces of code look similar today but
    serve different purposes and are likely to diverge in the future, forcing them into a shared
    function creates coupling that makes both harder to change later. The rule of thumb is the
    "Rule of Three": tolerate duplication until you see the same pattern three times, then extract.
    When you do extract, make sure the abstraction you create has a clear, single responsibility.
    Sandi Metz's advice captures this well: "duplication is far cheaper than the wrong
    abstraction."
## Question 23:
    What is a callback function?
    
    Beginner Answer: A callback is a function passed as an argument to another function. The
    outer function can then call it whenever needed. For example, forEach takes a callback that
    it runs on each array element.
    
    Experienced Answer: A callback is a function reference handed to another function for
    deferred or delegated execution. Callbacks are the foundation of asynchronous JavaScript
    (event listeners, setTimeout, API handlers), but they also drive synchronous patterns like
    array HOFs. The key design idea is inversion of control: the caller defines what happens,
    while the receiver controls when and how often. Understanding this distinction matters
    because it explains both the power and the risk of callbacks. When you pass a callback to a
    third-party library, you are trusting that library to call it correctly, which is one motivation
    behind Promises and async/await.

## Question 24:
    What is the difference between map, filter, and reduce?
    
    Beginner Answer: map transforms every element and returns a new array of the same
    length. filter returns a new array with only the elements that pass a test. reduce
    combines all elements into a single value.
    
    Experienced Answer: All three are higher-order functions on Array.prototype that
    iterate without manual loop management. map applies a one-to-one transformation (input
    length equals output length). filter applies a predicate for subset selection (output length
    is less than or equal to input length). reduce is the most general. Both map and filter can
    be implemented using reduce, making reduce the universal array iterator. In practice,
    choosing the right method signals intent to other developers: map says "I am transforming,"
    filter says "I am selecting," and reduce says "I am accumulating." Misusing them (e.g.,
    using map for side effects instead of forEach, or using reduce when filter plus map is
    clearer) is a common code-review flag.

## Question 25:
    What is a pure function, and why does it matter?
    
    Beginner Answer: A pure function always returns the same output for the same input and
    does not change anything outside itself. It matters because pure functions are easy to test
    and predict.
    
    Experienced Answer: Purity means referential transparency: you can replace the function
    call with its return value without changing the program's behaviour. This property enables
    memoization (caching results for repeated inputs), safe parallelism, and straightforward
    unit testing with no mocks or setup. In React, for example, components are conceptually
    pure functions of state and props. Understanding purity helps you structure code so that
    business logic lives in pure, testable functions while side effects (DOM manipulation,
    network calls, logging) are pushed to the boundaries of the system. Interviewers look for
    this because it shows you think about code architecture, not just code that runs.

## Question 26:
    Why should you always provide an initial value for reduce?
    
    Beginner Answer: If you do not provide an initial value, reduce uses the first element as
    the starting accumulator and begins iteration from the second element. This can cause
    errors with empty arrays.
    
    Experienced Answer: Omitting the initial value introduces two risks. First, calling reduce
    on an empty array without an initial value throws a TypeError at runtime, which can crash
    production code if the array comes from an API response. Second, when reducing objects or
    non-numeric values, the first element may not be the correct type for the accumulator (e.g.,
    trying to sum prices from an array of objects without an initial value of 0 gives you an object
    concatenated with numbers). Providing the initial value makes the accumulator's type
    explicit, makes the function work safely on empty arrays (returning the initial value), and
    acts as documentation of intent for anyone reading your code.

## Question 27:
    What is a Higher-Order Function?
    
    Beginner Answer: A higher-order function is a function that either takes another function
    as an argument, returns a function, or does both. Array methods like map, filter, and
    forEach are higher-order functions.

    Experienced Answer: Higher-order functions are a direct consequence of functions being
    first-class values. They enable abstraction over actions, not just values. Instead of writing a
    new loop every time you want to transform or filter an array, you abstract the iteration
    pattern into a HOF and inject the specific behaviour via a callback. This is the core idea
    behind functional composition: small, reusable functions combined through higher-order
    functions to build complex behaviour. In real-world codebases, HOFs appear everywhere,
    from Express middleware (a function that takes req, res, next) to Redux reducers, to event
    handler registration. Recognising and writing HOFs fluently is a strong signal of JavaScript
    maturity in interviews.
## Question 28:
    What is the DOM, and how is it different from HTML?
    
    Beginner Answer: The DOM is the browser's in-memory representation of an HTML
    document as a tree of objects. HTML is the static source code you write. The DOM is the live structure the browser builds from that source. JavaScript interacts with the DOM, not with the HTML file directly.
    
    Experienced Answer: HTML is a declarative text format. When the browser parses it, it
    constructs the DOM: a tree of node objects that implement interfaces defined in the DOM
    specification (Element, Text, Attr, Document). The DOM can diverge from the original HTML.
    JavaScript can add nodes that never existed in the source, and the browser itself may insert nodes to fix malformed markup (for example, adding a <tbody> inside a <table> even if the HTML omits it). The DOM is also the integration point for CSS (the CSSOM) and JavaScript. When we say "manipulate the page," we mean mutating this live object tree, which triggers the browser's rendering pipeline (style recalculation, layout, paint, composite).

## Question 29:
    What is the difference between querySelector and getElementById?
    
    Beginner Answer: getElementById finds an element by its unique id attribute.
    querySelector accepts any CSS selector (IDs, classes, tag names, combinators) and returns the first match. querySelector is more flexible.
    
    Experienced Answer: getElementById is scoped to the document object, is marginally
    faster in benchmarks because it uses the browser's internal ID hash map, and returns null for no match. querySelector can be called on any element (not just document), enabling scoped searches like container.querySelector('.child'). It uses the full CSS selectorengine, so you can write complex selectors such as 'form > input[type="email'. One subtle difference: getElementById does not require a # prefix (it expects a string),
    while querySelector requires '#id' with the hash. In practice, the performance gap is
    negligible, and querySelector is preferred for its consistency and flexibility.

## Question 30:
    Explain addEventListener. How does it relate to callbacks?
    
    Beginner Answer: addEventListener attaches a function to a DOM element for a specific
    event type. The function (the callback) runs every time that event fires. It is the same
    callback concept from higher-order functions: you pass a function to be invoked later.
    
    Experienced Answer: addEventListener registers a callback in the browser's internal
    event dispatch table for a given element and event type. It is inherently asynchronous in that
    the callback is not invoked immediately. It sits idle until the event loop picks up the
    corresponding event from the task queue. Unlike the older onclick property (which allows
    only one handler), addEventListener supports multiple listeners on the same element
    and event type, called in registration order. It also accepts a third argument: an options
    object or boolean controlling the capture phase, the once flag (auto-removes after first
    invocation), and passive (hints to the browser that preventDefault will not be called,
    improving scroll performance). Understanding this third argument becomes critical when
    dealing with event delegation and scroll-heavy UIs.

## Question 31:
    What is the difference between e.target and e.currentTarget?
    
    Beginner Answer: e.target is the element the user actually clicked (or interacted with).
    e.currentTarget is the element that the event listener is attached to. They are the same
    when the listener is on the clicked element itself, but different when the event bubbles up
    from a child.

    Experienced Answer: This distinction is the foundation of event delegation. In a pattern
    where you attach one listener to a parent (say, a <ul>) instead of individual listeners to each
    child (<li>), e.currentTarget is always the <ul>, while e.target identifies which
    specific <li> was clicked. This matters for performance (one listener instead of hundreds)
    and for dynamically added elements (new <li> nodes automatically inherit the parent's
    listener without re-registration). A common interview follow-up is: "What is
    e.currentTarget inside a setTimeout callback within the handler?" The answer is null,
    because currentTarget is only available synchronously during event dispatch. After the
    handler returns, the browser resets it.

## Question 32:
    Describe the three-step pattern for dynamically adding an element to the page.
    
    Beginner Answer: Step 1: Create the element with document.createElement('tag').
    Step 2: Configure it (set innerText, add classes, set attributes). Step 3: Attach it to the DOM
    with parent.appendChild(element) or parent.insertBefore(element,
    referenceNode).
    
    Experienced Answer: The create-configure-attach pattern minimizes DOM mutations,
    which is important because each mutation can trigger style recalculation and layout
    (reflow). When adding many elements (for example, 100 list items), a best practice is to
    build them in a DocumentFragment first, then append the fragment in a single operation.
    This batches the reflow into one pass. Additionally, configuring elements before attaching
    them is deliberate: changes to detached nodes are "free" because they are not yet part of the
    render tree. Once attached, every property change can trigger rendering work. This is also
    why frameworks like React use a virtual DOM, to diff changes in memory and apply only the
    minimum set of mutations to the real DOM.

## Question 33:
    What is a NodeList? How does it differ from an Array?
    
    Beginner Answer: A NodeList is a collection of DOM nodes returned by methods like
    querySelectorAll. It supports index access, .length, and .forEach(), but does not
    support array methods like .map(), .filter(), or .push(). You can convert it to an array
    with Array.from().
    
    Experienced Answer: A NodeList is an array-like object that implements the iterable
    protocol (so it works with for...of and spread syntax in modern browsers) but does not
    inherit from Array.prototype. The static NodeList returned by querySelectorAll is a
    snapshot: DOM changes after the query do not affect it. In contrast, properties like
    Node.childNodes return a live NodeList that reflects DOM mutations in real time.
    Converting with Array.from(nodeList) or [...nodeList] gives you the full array API. A
    subtlety: HTMLCollection (returned by getElementsByClassName) is also array-like but 
    does NOT support .forEach() natively in all environments, making it even more limited
    than NodeList.

## Question 34:
    Why use classList.add() instead of directly setting className?
    
    Beginner Answer: classList.add() adds a class without removing existing ones. Setting
    className directly replaces all existing classes with the new value.
    
    Experienced Answer: classList provides an atomic, token-based API. .add() is
    idempotent (adding a class that already exists is a no-op, no duplicates). .toggle() with an
    optional second boolean argument (classList.toggle('active', isActive)) gives
    declarative control. .replace('old', 'new') swaps one class for another atomically. The
    className property is a raw string, so managing it requires manual string splitting,
    checking for duplicates, and re-joining. It is error-prone: className += ' highlight'
    accidentally works but can create leading spaces and duplicate entries. classList avoids
    all of these pitfalls and aligns with how CSS class management is handled in
    component-based frameworks.

## Question 35:
    Why is addEventListener preferred over inline onclick attributes?
    
    Beginner Answer:addEventListener() keeps JavaScript separate from HTML, which is
    cleaner. It also lets you attach multiple handlers to the same event on the same element. An
    inline onclick provides only one click handler, so if you want multiple actions, you have to
    write them together inside the same onclick attribute.
    
    Experienced Answer: Beyond separation of concerns and multiple handler support,
    addEventListener provides control over the event phase (capturing vs bubbling) through
    its third argument. It enables clean programmatic removal via removeEventListener
    when a named function reference is used. Inline handlers execute in a different scope
    context, with the element itself injected into the scope chain via an implicit with binding,
    which can cause unexpected variable resolution. In larger applications, inline handlers make
    event auditing and debugging significantly harder because interaction logic is scattered
    across template files rather than centralized in JavaScript modules. Modern frameworks and
    Content Security Policies often block inline handlers entirely.

## Question 36:
    What is the Event Object and what are its most important properties?
    
    Beginner Answer: The Event Object is automatically created by the browser when an event
    fires and passed as the first argument to the callback function. Its most useful properties are
    event.type (the event name), event.target (the element the user interacted with), and
    event.preventDefault() (which stops default browser behavior like form submission).
    
    Experienced Answer: The Event Object is the browser's structured report of everything
    about the interaction. Beyond type, target, and preventDefault(), currentTarget
    provides a stable reference to the listener's host element, which becomes critical in
    delegation patterns. For keyboard events, event.key gives the produced character while
    event.code gives the physical key, and knowing the difference matters for
    internationalized keyboards where the same physical key produces different characters.
    preventDefault() cancels the native browser action without stopping propagation, while
    stopPropagation() does the opposite: it halts bubbling without canceling the default
    action. Understanding that these two are independent operations is a common interview
    differentiator.

## Question 37:
    What is the difference between event.target and event.currentTarget?
    
    Beginner Answer: event.target is the element the user actually clicked on.
    event.currentTarget is the element the event listener is attached to. When you attach a
    listener directly to a button and click that button, they are the same. They differ when a
    parent element holds the listener and a child receives the click.
    
    Experienced Answer: event.target references the deepest DOM node that originated the
    event, and it changes depending on exactly where the user clicks. event.currentTarget
    always references the element on which addEventListener was called, providing a stable
    reference throughout the handler. This distinction is foundational for event delegation,
    where a single parent listener handles interactions from many children. One important
    subtlety: event.currentTarget is only available during event dispatch. If you log it inside
    a setTimeout, an await, or a Promise.then() callback, it will be null because the event
    dispatch cycle has already completed.

## Question 38:
    Why should you use keyup instead of keydown for filtering an input field?
    
    Beginner Answer: keydown fires before the character appears in the input, so
    input.value still shows the old value. keyup fires after the key is released, so
    input.value is up to date with the latest keystroke.
    
    Experienced Answer: At keydown time, the keystroke has been detected but not yet
    processed by the browser's input handling pipeline. Reading input.value at that point
    gives the state before the current character is inserted, which means your filter is always
    one character behind. keyup fires post-insertion, so the value is current. However, for
    production search filters, the input event is more robust because it covers non-keyboard
    value changes: paste operations, autofill, drag-and-drop text, mobile predictive keyboards,
    and speech-to-text. keyup misses all of these. A practical upgrade path is to start with keyup
    for learning, then migrate to input for production code.

## Question 39:
    What is the difference between window.scrollY and getBoundingClientRect()?
    
    Beginner Answer: window.scrollY tells you how many pixels the whole page has been
    scrolled from the top. getBoundingClientRect() tells you where a specific element is
    positioned relative to the current viewport. You use scrollY for simple threshold checks
    (like changing a header) and getBoundingClientRect() when you need to know if a
    particular element is visible.
    
    Experienced Answer: window.scrollY is a single global number representing the
    document's vertical scroll offset. It is cheap to read and ideal for threshold-based logic (e.g.,
    add a class to the header after 80px of scrolling). getBoundingClientRect() is
    element-specific and returns a DOMRect object with top, bottom, left, right, width, and
    height relative to the viewport. It recalculates layout, which means calling it on many
    elements inside a scroll handler can trigger layout thrashing, a performance problem where
    the browser is forced to recalculate positions repeatedly within a single frame. The modern
    alternative for visibility detection is the IntersectionObserver API, which lets the
    browser handle visibility checks asynchronously without scroll event listeners or manual
    getBoundingClientRect calls.

## Question 40:
    What is the difference between element.remove() and setting display to none?
    
    Beginner Answer: remove() deletes the element from the DOM entirely. It is gone.
    display: none hides it visually but it stays in the DOM and can be shown again by
    resetting the display property.
    
    Experienced Answer: element.remove() detaches the node from the DOM tree. It no
    longer exists in the document, querySelector will not find it, and any event listeners
    attached directly to it become eligible for garbage collection (assuming no other references
    hold onto it). display: none is purely visual: the element remains in the DOM, retains its
    listeners, participates in form submissions if it is a form field, and can be queried and
    manipulated at any time. The choice depends on intent. Use display: none (or a CSS class
    toggle) for show/hide features like accordions and tabs. Use remove() for permanent
    session-level deletion like dismissing a notification or removing a task. If you might need the
    element back after removal, store a reference in a variable before calling remove(), then
    re-insert it with appendChild later.

## Question 41:
    What is event delegation and why is it important?
    
    Beginner Answer: Instead of attaching event listeners to every child element, you attach
    one listener to the parent. When a child is clicked, the event bubbles up to the parent, which
    handles it. This is important because it works for elements added dynamically after the page
    loads.

    Experienced Answer: Event delegation leverages DOM event bubbling: since click events
    propagate from the target upward through every ancestor, a single listener on a common
    ancestor can handle interactions for any number of descendants. The practical benefits are
    reduced memory usage (one listener instead of N), automatic coverage of dynamically
    inserted elements without re-attaching listeners, and simpler teardown (one
    removeEventListener call). The robust implementation uses
    event.target.closest(selector) rather than a direct classList check, because the
    actual click target may be a nested child element inside the intended interactive element.
    The delegation parent must be a stable DOM node present when the listener is attached. In
    very dynamic applications where even the container is generated, document.body can
    serve as a fallback delegation root.

## Question 42:
    Why use closest() instead of checking event.target directly in delegation?
    
    Beginner Answer: If a button contains nested elements like an icon or span,
    event.target might be the inner element, not the button itself. closest() walks up the
    DOM from wherever the click landed and finds the nearest matching ancestor, so it works
    regardless of what was directly clicked.
    
    Experienced Answer: event.target always references the deepest DOM node that
    received the interaction. In a button with inner markup (icons, spans, text nodes), the target
    is often a child, not the button. A classList.contains check on the target fails silently in
    this case, creating an intermittent bug that is difficult to reproduce because it depends on
    exactly which pixel the user clicks. closest(selector) eliminates this fragility by
    traversing upward from the target, checking each ancestor (including the target itself)
    against the selector. It returns the first match or null if none is found. A secondary benefit
    is structural resilience: using closest('.task-card') instead of parentElement means
    the code continues to work even if the HTML nesting depth changes during a redesign.

## Question 43:
    What is localStorage and what are its limitations?
    
    Beginner Answer: localStorage is a built-in browser storage that saves data as key-value
    pairs. It persists across page refreshes and browser restarts. Its main limitation is that it
    only stores strings, so you need JSON.stringify and JSON.parse for arrays and objects.
    
    Experienced Answer: localStorage is a synchronous, origin-scoped, persistent key-value
    store with approximately 5 MB of capacity per origin. It survives refresh, tab closure, and
    browser restart. Its limitations are significant for real applications. It is synchronous,
    meaning reads and writes block the main thread (problematic for large datasets). It is
    string-only, requiring JSON serialization for structured data. It has no built-in expiration
    mechanism (unlike cookies). It is not accessible from Web Workers or Service Workers
    (IndexedDB serves that role). It offers no encryption, so sensitive data should never be
    stored there. And it is local to the device: a user's data on their laptop is invisible on their
    phone. For anything beyond simple preferences or small datasets, server-side storage with
    API communication is the standard approach.

## Question 44:
    Why are JSON.stringify and JSON.parse needed with localStorage?
    
    Beginner Answer: localStorage can only store strings. If you try to store an array or object
    directly, it gets converted to useless text like "[object Object]". JSON.stringify
    converts structured data into a properly formatted string, and JSON.parse converts it back
    into a real JavaScript value.
    
    Experienced Answer: localStorage's setItem implicitly calls toString() on any
    non-string value. For arrays, toString() produces a flat comma-separated string with no
    brackets, losing all structural information. For objects, it produces the string "[object
    Object]", completely destroying the data. JSON.stringify serializes the value into a
    JSON-formatted string that preserves the structure: array brackets, object braces, nested
    values, and data types (except undefined, functions, and Symbol, which are silently
    omitted). JSON.parse deserializes that string back into a living JavaScript value. One
    critical defensive pattern: always guard against null from getItem before parsing, because
    JSON.parse(null) returns null (safe), but if the stored string is malformed, JSON.parse
    throws a SyntaxError. In production, wrapping the parse call in a try-catch is standard
    practice.

## Question 45:
    What is the data-driven rendering pattern and why does it matter?
    
    Beginner Answer: Instead of treating the HTML as the source of truth and manipulating it
    directly, you keep your data in a JavaScript array. A render function builds the entire UI from
    that array. When data changes, you update the array, save it, and re-render. This keeps the
    data and the display in sync.
    
    Experienced Answer: The data-driven rendering pattern separates state from
    presentation. The JavaScript array (or object) is the single source of truth. The DOM is a
    derived view of that data, regenerated by a render function whenever the state changes.
    This is the conceptual foundation of every modern UI framework (React, Vue, Angular).
    Direct DOM manipulation (finding a specific card and removing it) creates two sources of
    truth: the DOM and whatever variable tracks the data. These inevitably fall out of sync. The
    data-driven approach eliminates that risk. The tradeoff is performance: clearing innerHTML
    and rebuilding every element on every change is expensive for large lists. Frameworks solve
    this with virtual DOMs and diffing algorithms, but for learning purposes and small datasets,
    the clear-and-rebuild approach is correct and sufficient.

## Question 46:
    What is JSON, and why is it used in web development?
    
    Beginner Answer: JSON stands for JavaScript Object Notation. It is a text format for
    representing data as key-value pairs. APIs use it to send data between servers and browsers
    because both sides can read it easily.
    
    Experienced Answer: JSON is a language-agnostic, lightweight data interchange format.
    While its syntax is derived from JavaScript object literals, it is supported natively by virtually
    every programming language (Python's json module, Java's Jackson/Gson, Go's
    encoding/json). It dominates web APIs because it is human-readable, easy to parse, and
    has minimal overhead compared to XML. Two critical rules distinguish it from JS objects: all
    keys must be double-quoted strings, and it does not support functions, undefined, or
    comments. JSON.stringify() serializes, JSON.parse() deserializes, and
    response.json() is a convenience method that reads a fetch response stream and parses
    it in one step.

## Question 47:
    What does fetch() return, and how do you get usable data from it?
    
    Beginner Answer: fetch() returns a Promise that resolves to a Response object. You call
    .json() on the response to get the actual data as a JavaScript object.
    
    Experienced Answer: fetch() returns a Promise that resolves once the HTTP headers
    are received (not when the full body has downloaded). The resolved value is a Response
    object containing metadata (status, ok, headers) and a body stream. To extract the body,
    you call one of its consumer methods (.json(), .text(), .blob(), .arrayBuffer()),
    each of which returns another Promise because reading the stream is asynchronous. A key
    nuance: fetch() does not reject on HTTP error codes like 404 or 500. It only rejects on
    network-level failures (DNS errors, no connectivity). That is why you must manually check
    response.ok or response.status and throw your own error for non-2xx responses.

## Question 48:
    What is the purpose of try...catch, and when should you use it?
    
    Beginner Answer: try...catch lets you handle errors without crashing the app. You put
    risky code inside try, and if anything fails, the catch block runs instead of throwing an
    error to the console.
    
    Experienced Answer: try...catch provides structured exception handling. The try
    block wraps any code that might throw, whether that is a runtime error, a deliberately
    thrown Error via throw, or a rejected Promise (when using await). The catch block
    receives the error object and lets you log it, display a user-friendly message, attempt a retry,
    or fall back to cached data. In async functions, try...catch is the synchronous-style
    equivalent of .catch() on Promises. A critical best practice: never use an empty catch
    block. Silent failures are harder to debug than crashes. Always log or handle the error
    meaningfully. For granular control, you can also use a finally block that runs regardless of
    success or failure, commonly used for cleanup tasks like hiding a loading spinner.

## Question 49:
    What is the difference between JSON.parse() and response.json()?
    
    Beginner Answer: They both turn a JSON string into a JavaScript object. JSON.parse()
    works on any string, while response.json() is specifically for reading fetch responses.
    
    Experienced Answer: JSON.parse() is a synchronous function that takes a JSON string
    and returns a JavaScript value. response.json() is an asynchronous method on the
    Response object that reads the response body stream to completion and then internally
    performs the equivalent of JSON.parse(). Because reading the body is I/O-bound (the data
    may still be arriving over the network), .json() returns a Promise that must be awaited.
    Another difference: JSON.parse() can be called multiple times on the same string, but
    response.json() (and any body-reading method) can only be called once per Response,
    because the stream is consumed. Attempting to call it a second time throws a TypeError
    stating the body has already been read.

## Question 50:
    Why do we use event.preventDefault() on form submission?
    
    Beginner Answer: When you submit a form, the browser reloads the page by default.
    event.preventDefault() stops that reload so JavaScript can handle the submission
    instead.
    
    Experienced Answer: The browser's default form submission behavior sends an HTTP
    request (GET or POST) to the URL specified in the action attribute (or the current page if
    none is set) and navigates to the response, effectively reloading the page. In a
    JavaScript-driven application, this destroys the current DOM state, in-memory variables, and
    any open WebSocket connections. event.preventDefault() intercepts the event before
    the browser acts on it, giving JavaScript full control over what happens next. This is
    foundational to single-page application (SPA) architecture, where navigation and data
    submission are handled entirely in JavaScript without full page reloads. It is also essential
    when performing client-side validation before sending data to a server via fetch().

## Question 51:
    What is the DOM, and how does JavaScript interact with it?
    
    Beginner Answer: The DOM (Document Object Model) is a tree-like representation of an
    HTML page that the browser creates in memory. JavaScript uses methods like
    document.querySelector() to select elements from this tree and then reads or modifies
    their properties, classes, or content.
    
    Experienced Answer: The DOM is a programming interface that represents an HTML
    document as a hierarchical tree of node objects. Each node exposes properties and methods
    defined by Web API specifications (not by the JavaScript language itself). When JavaScript
    calls document.querySelector('.modal'), the browser traverses this tree and returns
    the first matching Element node. From there, the script can mutate the node's attributes,
    children, or computed styles, triggering the browser's rendering pipeline (style
    recalculation, layout, paint, composite). Performance-conscious developers batch DOM
    reads and writes to avoid "layout thrashing," where interleaved reads and writes force the
    browser to recalculate layout repeatedly within a single frame.

## Question 52:
    Explain event delegation with an example.
    
    Beginner Answer: Instead of adding a click listener to every child element, you add one
    listener to the parent. When a child is clicked, the event bubbles up to the parent, and you
    use event.target to figure out which child was clicked. For example, one listener on a
    <ul> can handle clicks on any <li> inside it.
    
    Experienced Answer: Event delegation exploits the bubble phase of DOM event
    propagation. A single listener on a common ancestor inspects event.target (the
    originating element) or walks up with event.target.closest(selector) to identify the
    intended child. This approach has three practical advantages. First, it reduces memory usage
    because one function reference replaces N references. Second, it inherently supports
    dynamically added children, since the parent listener does not need to know about elements
    that did not exist at bind time. Third, it simplifies teardown, requiring only one
    removeEventListener call. A common pitfall is clicking on a nested element inside the
    intended target (for example, a <span> inside a <button>). Using
    event.target.closest('.nav-tab') instead of checking event.target directly solves
    this, because closest walks up the DOM from the clicked element and returns the first
    ancestor matching the selector.

## Question 53:
    What is the difference between classList.add(), classList.remove(), and
    classList.toggle()?
    
    Beginner Answer: add() puts a class on an element, remove() takes it off, and toggle()
    adds it if it is missing or removes it if it is present. They are all methods on an element's
    classList property.
    
    Experienced Answer: All three are methods on the DOMTokenList returned by
    element.classList. add() and remove() are idempotent: calling add('active') when
    the class already exists does nothing (no error, no duplicate), and calling
    remove('active') when the class is absent is also a no-op. toggle() accepts an optional
    second boolean argument, a force flag. classList.toggle('active', true) always
    adds the class (equivalent to add), and classList.toggle('active', false) always
    removes it (equivalent to remove). This force flag is useful when the desired state is already
    known from a variable, eliminating the need for an if/else branch. All three methods are
    preferred over direct className string manipulation because they do not accidentally
    overwrite other classes on the element.

## Question 54:
    Why use event.key instead of event.keyCode?
    
    Beginner Answer: event.key returns a readable string like 'Enter' or 'a', while
    event.keyCode returns a number like 13. keyCode is deprecated and harder to read.
    event.key is the modern standard.
    
    Experienced Answer: event.keyCode was deprecated because its numeric values were
    inconsistently implemented across browsers and keyboard layouts. A physical key might
    produce different keyCode values on US-QWERTY versus German-QWERTZ layouts.
    event.key returns a standardised string based on the character produced (e.g., 'a',
    'Enter', 'ArrowDown'), while event.code returns a string based on the physical key
    position (e.g., 'KeyA', 'Enter', 'ArrowDown'). For text input validation (like checking if
    the user pressed Enter), event.key is correct. For game controls where physical position
    matters regardless of layout, event.code is the better choice. In interviews, mentioning
    this distinction signals awareness of internationalisation and input handling nuances.

## Question 55:
    What does data-* in HTML do, and how do you access it in JavaScript?
    
    Beginner Answer: data-* attributes let you store custom information on HTML elements.
    In JavaScript, you access them through the dataset property. For example,
    data-page="dashboard" is accessed as element.dataset.page.
    
    Experienced Answer: HTML5 data-* attributes provide a standards-compliant way to
    embed application-specific metadata on any element without conflicting with existing or
    future HTML attributes. The dataset property returns a DOMStringMap where each key is
    the camelCased version of the attribute name after the data- prefix. So data-user-id
    becomes element.dataset.userId. All values are strings; numeric or boolean values
    require explicit conversion (Number(), JSON.parse()). From a performance perspective,
    dataset access is slightly slower than reading a plain JavaScript property, so in hot loops
    (thousands of iterations per frame), caching the value is advisable. In typical application
    code, this is irrelevant. Frameworks like React discourage data-* in favour of component
    state, but in vanilla JS and jQuery codebases, data-* attributes remain the standard way to
    tie DOM elements to application data.

## Question 50:
    Q6: What is a guard clause, and why is it preferred over nested if/else?

    Beginner Answer: A guard clause checks for an invalid or edge-case condition at the top of
    a function and returns early if it is true. This avoids deeply nested if/else blocks and
    makes the main logic easier to read.

    Experienced Answer: A guard clause is an early-exit conditional that handles exceptional
    or invalid states before the function's primary logic executes. The pattern reduces
    cyclomatic complexity (the number of independent paths through a function), which
    directly correlates with bug density and cognitive load during code review. In a function
    with three validation checks, nested if/else produces three levels of indentation and
    forces the reader to mentally track which branch they are in. Three guard clauses produce
    flat, sequential code where each check is self-contained. Beyond readability, guard clauses
    make unit testing simpler: each early return is an isolated, testable path. Most style guides
    (Airbnb, Google) and linters (ESLint's no-else-return rule) explicitly recommend this
    pattern. The only caveat is overuse in functions that genuinely have two equally important
    branches (not "valid vs. invalid"), where an if/else block better communicates the
    symmetric structure.

