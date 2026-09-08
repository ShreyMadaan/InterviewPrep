## Question 1:
    What is JSX, and can a browser execute it directly?
    
    Beginner answer: JSX is a syntax extension that lets you write HTML-like code inside
    JavaScript. Browsers cannot run JSX directly. A compiler like Babel transforms JSX into
    React.createElement calls (or jsx helper calls) before the browser executes it.
    
    Experienced answer: JSX is syntactic sugar, not a separate language. It is defined as a
    JavaScript syntax extension (originally proposed alongside React but not part of the
    ECMAScript specification). A build-time transform converts each JSX expression into a
    function call that returns a React element, which is a plain, immutable JavaScript object with
    type, props, and internal fields. The classic transform emits React.createElement,
    requiring React to be in scope. The automatic transform (React 17+) emits jsx/jsxs calls
    from react/jsx-runtime, removing that import requirement. In either case, JSX produces
    the same element tree. Browsers never see JSX; they receive the transformed JavaScript. The
    distinction matters in production because the transform is a compile-time step (Vite,
    Webpack, SWC), whereas running Babel in the browser (as done in learning setups) adds
    parse-time overhead and is unsuitable for production.

## Question 2:
    What is the difference between a React element and a DOM element?

    Beginner answer: A React element is a plain JavaScript object that describes what should
    appear on screen (e.g., { type: "h1", props: { children: "Hello" } }). A DOM
    element is an actual node in the browser's document tree. React.createElement
    produces the former. The React DOM renderer creates the latter.

    Experienced answer: A React element is an immutable, lightweight descriptor. It carries a
    type (string for host elements, function/class for components), a props object (including
    children), and internal fields like key and ref. Creating one is cheap (no system resources,
    no event listeners). It is effectively a frame in a "UI flipbook." A DOM element, by contrast, is
    a heavyweight browser object inheriting from HTMLElement, backed by layout, paint, and
    compositing layers. When root.render is called, the React DOM renderer walks the
    element tree, compares it against the previous tree (reconciliation), and issues the minimum
    set of DOM mutations (create, update, or remove nodes). This separation is what enables
    React's renderer-agnostic architecture. The same React element tree can be consumed by
    React DOM (browser), React Native (mobile), react-three-fiber (WebGL), or a test
    renderer, because the element is just a description, decoupled from any rendering target.

## Question 3:
    Why are react and react-dom separate packages?
    
    Beginner answer: react contains the core API for defining components and elements.
    react-dom is the renderer that knows how to work with the browser DOM. They are
    separate so React's core can be reused with other renderers like React Native.
    Experienced answer: The separation enforces a clean architectural boundary between the
    reconciler (which diffs element trees and determines what changed) and the host
    environment (which executes platform-specific mutations). react owns the reconciler
    logic, the component model, Hooks, and the element/fiber data structures. react-dom
    implements the host config: it knows how to create DOM nodes, set attributes, handle events
    via a synthetic event system with delegation at the root, and flush updates. This design
    means you can swap the renderer without changing component code. react-native
    provides a renderer targeting iOS/Android views, react-three-fiber targets Three.js
    scenes, and ink targets terminal output. The packages share the same reconciliation
    algorithm but differ entirely in how they materialize the UI. The split also allows
    react-dom/server to exist as a separate entry point for server-side rendering without
    pulling in client-side DOM logic.

## Question 4:
    Why must React component names start with an uppercase letter?
    
    Beginner answer: JSX uses the first letter to decide whether a tag is a built-in HTML
    element or a custom component. <div> is treated as a DOM element. <Greeting> is treated
    as a component function call. If you write <greeting>, React looks for an HTML element
    called "greeting" and ignores your function.
    
    Experienced answer: The JSX transform uses a naming convention to disambiguate host
    elements from component invocations. A lowercase tag compiles to a string argument:
    React.createElement("div", ...), telling the renderer to create a DOM node of that
    type. An uppercase tag compiles to an identifier reference:

    React.createElement(Greeting, ...), telling the renderer to call that function (or
    instantiate that class) and recursively process its return value. This is a transform-time
    decision, not a runtime check. If you accidentally lowercase a component, the transform
    emits a string, and React either creates an unknown DOM element (which the browser
    ignores or renders as an inline element) or throws a warning in development mode. The
    convention also means you cannot use a dynamically chosen component type directly in JSX
    without first assigning it to a capitalized variable (e.g., const SelectedComponent =
    components[type]; return <SelectedComponent />;).

## Question 5:
    What are props, and what does it mean that they are read-only?
    
    Beginner answer: Props are the inputs a parent passes to a child component, like function
    arguments. They are read-only because a component should not change its own inputs. If
    the output needs to change, the parent re-renders the component with new props.
    
    Experienced answer: Props form a unidirectional data contract: the parent owns the data,
    and the child receives it as an immutable snapshot for that render. Mutating props would
    break React's ability to reason about UI consistency, because the reconciler assumes that a
    component's output is a pure function of its props (and state). If a child could silently
    mutate a prop, the parent's copy and the child's copy would diverge, making the UI
    unpredictable and breaking features like memoization (React.memo), concurrent
    rendering, and devtools inspection. Internally, React freezes the props object in
    development mode (Object.freeze) to surface mutations as errors early. The read-only
    contract also enables structural sharing during reconciliation. If props have not changed by
    reference, React can skip re-rendering the subtree entirely when combined with
    memoization. Props can carry any JavaScript value (strings, numbers, objects, functions,
    even other React elements), but the contract remains: receive, read, never write.

## Question 6:
    What is a React Fragment, and when would you use one?
    
    Beginner answer: A Fragment (<>...</> or
    <React.Fragment>...</React.Fragment>) lets you return multiple sibling elements
    from a component without adding an extra DOM node like a <div>. You use it when a
    wrapper element would break your layout or semantics.
    
    Experienced answer: Fragments solve the single-root-expression constraint of JSX without
    polluting the DOM tree. This matters in several practical scenarios: a component returning
    <td> elements inside a table row (a wrapping <div> inside <tr> is invalid HTML), CSS
    Flexbox/Grid layouts where an extra wrapper disrupts child ordering, and
    accessibility-sensitive markup where spurious container elements confuse screen readers.
    The short syntax <>...</> does not accept props. When you need a key prop (e.g., when
    mapping a list of grouped elements), you must use the explicit <React.Fragment
    key={id}> form. Under the hood, a Fragment has a special symbol type
    (Symbol.for('react.fragment')) and the reconciler processes its children as if they
    were direct children of the parent, without creating any host node.

## Question 7:
    What is JSX, and what happens to it before the browser runs it?
    
    Beginner answer: JSX is a syntax extension that lets you write HTML-like code inside
    JavaScript. The browser cannot understand JSX directly, so a tool like Babel or Vite's React
    plugin transforms it into regular JavaScript function calls before the browser sees it.
    
    Experienced answer: JSX is syntactic sugar that compiles to function calls describing UI
    elements. Under the classic runtime (pre-React 17), it compiled to
    React.createElement(type, props, ...children), which required React to be in
    scope in every file. Under the automatic runtime (React 17+), it compiles to _jsx() or
    _jsxs() imported from react/jsx-runtime, and the tooling injects that import
    automatically. The key point is that JSX is entirely a compile-time abstraction. By the time
    JavaScript reaches the browser, no JSX syntax remains. This is why JSX has constraints that
    mirror JavaScript (expressions only in curly braces, className instead of class) rather
    than constraints that mirror HTML.

## Question 8:
    Why can you not use an if statement inside JSX curly braces?
    
    Beginner answer: JSX curly braces only accept expressions, things that produce a value. An
    if statement is a control flow instruction, not something that evaluates to a value, so it
    causes a syntax error. You use ternaries or && instead.
    
    Experienced answer: JSX curly braces compile into arguments of _jsx() function calls. For
    example, <p>{x ? "A" : "B"}</p> becomes _jsx("p", { children: x ? "A" :
    "B" }). The ternary works because it is an expression that resolves to a value that can be
    passed as an argument. An if statement cannot appear in argument position in JavaScript,
    so it is a syntax error at the compiled output level. This is not a React limitation. It is a
    JavaScript language constraint. The workarounds (ternary, &&, variable assignment before
    return) all ensure that only an expression ends up in the function argument slot.

## Question 9:
    What is a dependency graph, and why does it matter?
    
    Beginner answer: A dependency graph is a map of all the files in your project and how they
    connect through imports. The build tool starts from the entry point and follows every
    import to find all connected files. If a file is not imported by anything, it is excluded from
    the final output.
    
    Experienced answer: The dependency graph is the data structure a bundler constructs by
    statically analyzing ES module import/export declarations starting from the configured
    entry point(s). It determines three things: what to include (only reachable modules), what is
    shared (common dependencies referenced by multiple modules, which can be split into
    shared chunks), and what is unused (exported bindings never imported anywhere, which
    tree shaking eliminates). The graph is why ES modules' static structure matters. Unlike
    CommonJS require(), which can appear inside conditionals and be dynamically computed,
    ES import statements are hoisted and deterministic, so the graph can be fully resolved at
    build time without executing the code.

## Question 10:
    What is tree shaking?
    
    Beginner answer: Tree shaking is a build optimization that removes exported code that no
    file in your project actually imports. If you export three functions but only import one, the
    other two are dropped from the production bundle to reduce file size.
    
    Experienced answer: Tree shaking is dead-code elimination based on ES module static
    analysis. The bundler walks the dependency graph, marks every imported binding as "used,"
    and removes any exported binding that was never marked. It relies on the static structure of
    import/export: because these declarations cannot be conditional, the bundler can
    determine at build time, with certainty, which exports are consumed. Side effects complicate
    this. If a module executes code at the top level (e.g., modifying a global or calling a function
    on import), the bundler cannot safely remove it even if none of its exports are used. That is
    why package.json supports a "sideEffects" field, an explicit declaration by the library
    author that modules are safe to tree-shake. Libraries like lodash-es are designed as
    individual ES modules specifically to enable aggressive tree shaking.

## Question 11:
    What is the automatic JSX runtime, and why was it introduced?
    
    Beginner answer: The automatic JSX runtime is a feature introduced in React 17 where the
    build tool automatically adds the JSX function import. This means you no longer need to
    write import React from 'react' at the top of every file that uses JSX.
    
    Experienced answer: Before React 17, JSX compiled to React.createElement(), which
    meant every file containing JSX needed React in scope, even if it never used React directly.
    This was a source of lint errors, confusion, and unnecessary imports. The automatic
    runtime, specified via "runtime": "automatic" in the Babel or SWC config (Vite's React
    plugin sets this by default), compiles JSX to jsx() and jsxs() functions imported from
    react/jsx-runtime. The compiler inserts these imports automatically. Beyond developer
    ergonomics, this enabled minor performance optimizations in the jsx function itself (e.g., it
    handles key differently from createElement) and removed an implicit coupling between
    "any file with angle brackets" and the React global. Vite's @vitejs/plugin-react uses
    this runtime by default, which is why no React import is needed in modern Vite projects.

## Question 12:
    How do you pass data from a parent component to a child component in
    React?
    
    Beginner answer: You use props. In the parent's JSX, you add attributes to the child
    component tag, like <Card name="Mouse" price={599} />. Inside the child, you
    receive them as a props object or destructure them in the function parameter.
    
    Experienced answer: Props support one-way data flow in React. The parent owns
    the data and passes JavaScript values through JSX attributes; React then provides the
    child component with a props object when rendering it. Props are read-only from
    the child's perspective, which keeps the component tree predictable. In deeper trees,
    repeatedly passing props through intermediate components that do not use them is
    often called "prop drilling." When that genuinely becomes cumbersome, Context
    may help. For most component hierarchies, direct prop passing remains the simplest
    and most debuggable approach.

## Question 13:
    Why does React need a key prop on list items, and what happens if you use
    the array index?
    
    Beginner answer: React uses each key to match an item with its previous render. If
    a key is omitted, React warns and falls back to the item's position. Using the array
    index as a key can cause identity problems when items are inserted, removed, or
    reordered because the positions change.
    
    Experienced answer: During reconciliation, React uses keys to match previous and
    next children in a list. A stable, unique key lets React preserve the correct
    component and its associated DOM and state. If an array index is used and the list is
    reordered, the same positional key may now refer to different data, allowing state
    such as an input value to remain with the wrong item. Stable IDs from the data are
    therefore the preferred keys. An index can be acceptable when a list and its item
    order are truly static.

## Question 14:
    What is component composition and why is it preferred over deeply nested
    component logic?
    
    Beginner answer: Component composition means building a page from small,
    focused components that are combined together. Instead of putting everything in
    one giant component, you break the UI into pieces like ProductCard, ProductList,
    and Section, then nest them. It makes code easier to read, reuse, and maintain.
    
    Experienced answer: Composition is React's primary code-reuse mechanism.
    Instead of inheritance (which React explicitly discourages), you combine
    components by nesting them and leveraging the children prop. This yields several
    architectural benefits. First, each component has a single responsibility, making it
    independently testable. Second, the children pattern enables inversion of control: a
    Section component does not need to know what it wraps, so it stays generic and
    reusable across the app. Third, composition keeps the component tree flat in terms
    of coupling. A ProductList depends only on ProductCard, not on App. This means you
    can refactor, replace, or test ProductList in isolation. In larger codebases,
    composition patterns evolve into compound components, render props, and
    slot-based layouts, all of which are built on the same children foundation taught
    here.

## Question 15:
    What is the children prop and how does it enable reusable layout
    components?
    
    Beginner answer: children is a special prop that contains whatever JSX you place
    between a component's opening and closing tags. For example,
    <Section><p>Hello</p></Section> makes <p>Hello</p> available as children inside
    Section. It lets you create wrapper components that provide structure without
    knowing what content they will wrap.
    
    Experienced answer: children is a standard prop (not a keyword) that React
    populates with the sub-tree between a component's tags. Its type can be a single
    element, an array of elements, a string, a number, or even null. This flexibility is
    what makes it powerful for layout components, modal wrappers, error boundaries,
    and provider patterns. A Card component can provide border, shadow, and padding
    while remaining agnostic about its content. The pattern follows the open/closed
    principle: the component is closed for modification (its layout logic does not change)
    but open for extension (any content can be injected). In advanced usage, you can
    inspect, filter, or transform children using the React.Children utility API and
    React.cloneElement, though these are rarely needed in typical applications. The
    children prop is also the mechanism behind Context providers
    (<ThemeContext.Provider>{children}</ThemeContext.Provider>), making it
    foundational to React's architecture.

## Question 16:
    What is useState and why can we not use a regular variable instead?
    
    Beginner Answer: useState is a React hook that lets you add state to a functional
    component. Regular variables reset every time the component re-renders and do not trigger
    a re-render when changed. useState keeps the value across renders and tells React to
    update the screen when the value changes.
    
    Experienced Answer: useState registers a piece of state in React's internal fiber tree for
    that component instance. On each render, React returns the current value from that slot.
    When you call the setter, React enqueues an update, marks the component as dirty, and
    schedules a re-render in the next microtask batch. A regular variable exists only for the
    duration of a single function call, so it is re-initialized every render and has no mechanism to
    notify React's reconciler. This is why React requires hooks for any value that should persist
    across renders and influence UI output.

## Question 17:
    What happens if you call setState three times in a row with the same state variable?
    
    Beginner Answer: If you write setCount(count + 1) three times, the count only goes up
    by 1 because count is a snapshot that does not change during that render. To increment by
    3, use the functional form: setCount(prev => prev + 1).
    
    Experienced Answer: React batches all three setCount(count + 1) calls within the
    same event handler. Since count is captured in the closure at render time, all three calls
    compute the same value (e.g., $0 + 1 = 1$) and the final state is 1. With the functional
    updater setCount(prev => prev + 1), React queues three updater functions. During the
    flush phase, it processes them sequentially: $0 \to 1 \to 2 \to 3$. Since React 18, batching
    also applies inside setTimeout, promises, and native event handlers (automatic batching),
    unlike React 17 where batching was limited to React-managed events.
## Question 18:
    What is the difference between a controlled and an uncontrolled input?
    
    Beginner Answer: A controlled input has its value set by React state (value={state}) and
    updates via onChange. An uncontrolled input lets the DOM manage the value, and you read
    it when needed, usually with a ref.

    Experienced Answer: In a controlled input, React is the single source of truth. The render
    output always reflects state, and user input is intercepted by onChange, validated or
    transformed if needed, and then committed to state. This enables per-keystroke validation,
    conditional formatting, and derived state. An uncontrolled input delegates value storage to
    the DOM node. You access it imperatively via useRef. This is appropriate for file inputs
    (which cannot be controlled), integration with non-React libraries, or performance-sensitive
    cases where you want to avoid a re-render on every keystroke. The trade-off is that you lose
    React's declarative data flow, so debugging becomes harder when form logic is complex.

## Question 19:
    Why do we write onClick={handleClick} instead of onClick={handleClick()}?
    
    Beginner Answer: handleClick passes a reference to the function so React can call it later
    when the user clicks. handleClick() calls it immediately during render, which is a bug.
    
    Experienced Answer: JSX expressions inside {} are evaluated during the render phase.
    handleClick() invokes the function and passes its return value (usually undefined) as
    the click handler. If that function calls setState, it triggers a re-render during the current
    render, leading to an infinite loop or a React warning about updates during rendering.
    handleClick passes the function object itself, which React stores and invokes from its
    event delegation system when the click event fires. When arguments are needed, the
    standard pattern is onClick={() => handleClick(id)}, which creates a new arrow
    function per render. For most components this is fine, but in performance-critical lists, you
    can memoize or restructure to avoid unnecessary re-creation.

## Question 20:
    How does a child component send data to a parent in React?
    
    Beginner Answer: The parent passes a function to the child as a prop. The child calls that
    function and passes data as an argument. This is how data flows upward.
    
    Experienced Answer: React enforces unidirectional data flow: props go down, and the only
    way to communicate upward is via callback functions passed as props. The parent defines a
    handler (e.g., handleAddItem), passes it to the child (e.g., onAddItem={handleAddItem}),
    and the child invokes onAddItem(data) when appropriate. The parent receives data,
    updates its own state, and the new state flows back down as props. This pattern is
    sometimes called "lifting state up" because the shared state lives in the nearest common
    ancestor. The on/handle naming convention (prop is onX, handler is handleX) is not
    enforced by React but is a widely adopted community standard that improves readability.

## Question 21:
    Why do we call e.preventDefault() in form submission handlers?
    
    Beginner Answer: Without it, the browser performs its default form behavior: it reloads
    the page and sends a request to the server. In a React SPA, we do not want that because
    React manages the UI and we handle submission in JavaScript.
    
    Experienced Answer: The browser's default submit event triggers a full-page navigation
    (GET or POST to the action URL, defaulting to the current page). This would unmount the
    entire React component tree, discard all in-memory state, and reload the document.
    e.preventDefault() cancels this navigation so we can handle submission
    asynchronously, typically by calling an API via fetch and updating local state with the
    response. The e here is a SyntheticEvent wrapping the native SubmitEvent. Calling
    preventDefault on the synthetic event delegates to the native event's preventDefault
    internally. This pattern is fundamental to any SPA framework, not just React.