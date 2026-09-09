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
## Question 29:
## Question 30:










