## Question 1:
        What is Node.js, and why would you choose it over other backend technologies?
        
        Beginner Answer: Node.js is a JavaScript runtime built on Chrome's V8 engine that lets us
        run JavaScript outside the browser. I would choose it because it lets me use the same
        language on both frontend and backend, and it handles many concurrent connections well
        because of its non-blocking I/O model.
        
        Experienced Answer: Node.js is an event-driven, non-blocking runtime ideal for
        I/O-intensive workloads. Its strength is not raw computation speed but concurrency
        throughput. Traditional threaded servers (like Java's Tomcat with thread-per-request)
        allocate a thread for each connection, and each blocked thread consumes memory. Node
        uses a single-threaded event loop with asynchronous I/O. When a database query or
        network call is initiated, Node delegates it to the OS-level thread pool (managed by libuv),
        frees the main thread immediately, and processes the callback when the result arrives. This
        makes it excellent for API gateways, real-time apps, and microservices where the bottleneck
        is waiting on I/O, not CPU. However, for CPU-bound work (image processing, ML inference),
        the single thread becomes a liability. In those cases, you either offload to worker threads, a
        separate service in a language like Go or Rust, or a job queue.


## Question 2:
        What is middleware in Express, and why is the order important?
        
        Beginner Answer: Middleware is a function that runs between receiving the request and
        executing the route handler. For example, express.json() parses the request body. Order
        matters because Express runs middleware top to bottom, so a catch-all 404 handler placed
        above routes would intercept everything.
        
        Experienced Answer: Middleware in Express is any function with the signature (req,
        res, next). When a request arrives, Express walks through the middleware stack in the
        exact order of registration. Each middleware can read/modify req and res, end the cycle by
        sending a response, or call next() to pass control to the next middleware. This is a pipeline
        pattern. Order is critical for several reasons. Body-parsing middleware (express.json())
        must run before any handler that reads req.body. Authentication middleware must run
        before protected routes. Error-handling middleware (which has four parameters: err,
        req, res, next) must be registered last so it catches errors from all preceding layers. A
        misplaced catch-all app.use() will shadow every route below it. In production, the
        middleware stack typically looks like: CORS, body parser, request logger, auth, route
        handlers, 404 handler, error handler. Each layer has a specific responsibility and a deliberate
        position.


## Question 3:
        What is the difference between HTTP status codes 200, 201, 400, and 404?
        
        Beginner Answer: 200 means the request was successful. 201 means something new was
        created. 400 means the client sent bad or missing data. 404 means the route or resource
        was not found.
        
        Experienced Answer: These codes fall into two families. 2xx codes signal success: 200 (OK)
        is the general-purpose success code, while 201 (Created) specifically indicates a new
        resource was created and is semantically correct for POST endpoints that create records.
        Well-designed APIs often include a Location header with 201 responses, pointing to the
        newly created resource's URL. 4xx codes signal client errors: 400 (Bad Request) means the
        server understood the request but the payload is invalid (missing fields, wrong types, failed
        validation). 404 (Not Found) means the target resource or route does not exist. The
        distinction matters for frontend error handling. A 400 tells the client "fix your input and
        retry." A 404 tells the client "this thing does not exist." In REST API design, you also
        encounter 401 (Unauthorized, meaning not authenticated), 403 (Forbidden, meaning
        authenticated but lacking permission), and 409 (Conflict, such as a duplicate entry).
        Choosing the precise code lets clients implement granular error handling without parsing
        message strings.


## Question 4:
        Explain the request-response lifecycle in an Express application.
        
        Beginner Answer: The client sends an HTTP request. Express matches the request method
        and URL to a route. Any middleware runs first (like body parsing). Then the route handler
        function executes and sends back a response using res.json() or res.send().
        
        Experienced Answer: The full lifecycle has several layers. The request hits Node's http
        module, which Express wraps. Express creates enriched req and res objects, then begins
        walking the middleware stack. Application-level middleware registered via app.use() runs
        first (CORS, body parsing, logging). Then Express enters the router layer, where it matches
        req.method and req.path against registered routes. If a match is found, route-specific
        middleware (if any) runs, followed by the handler. The handler can be synchronous or
        asynchronous. It reads from req (params, query, body, headers), executes business logic,
        and calls a res method (res.json(), res.status().send(), etc.) to finalize the
        response. Calling a res method writes to the underlying TCP socket and ends the cycle. If no
        route matches, execution falls through to the catch-all 404 middleware. If any middleware or
        handler throws (or calls next(err)), Express jumps to the error-handling middleware. One

        critical rule: a handler must send exactly one response. Calling res.json() twice causes an
        "ERR_HTTP_HEADERS_SENT" crash, which is why we use return before early responses in
        validation logic.

## Question 5:
        What is the difference between SQL and NoSQL databases? When would you choose one
        over the other?
        
        Beginner Answer: SQL databases store data in tables with fixed columns and use SQL for
        queries. NoSQL databases like MongoDB store data as flexible JSON-like documents. I would
        choose SQL when data has strict relationships (like banking transactions) and NoSQL when
        the schema is flexible or when working with a JavaScript stack.
        
        Experienced Answer: The distinction goes beyond "tables vs documents." SQL databases
        enforce ACID transactions (Atomicity, Consistency, Isolation, Durability) at the database
        level, making them ideal when data integrity is non-negotiable: financial systems, inventory
        management, anything where a half-completed operation is unacceptable. They also excel
        when data is heavily relational (users have orders, orders have products, products have
        categories) because JOINs are built into the query engine. NoSQL databases trade some of
        these guarantees for flexibility and horizontal scalability. MongoDB, for example, supports
        multi-document transactions since version 4.0, but the ecosystem is optimized around
        single-document operations. The document model shines when your data is naturally
        hierarchical (a blog post with embedded comments), when your schema evolves rapidly
        during early product development, or when you need to scale reads across many nodes. In
        practice, many production systems use both: a relational database for transactional data and
        a document store for product catalogs, user profiles, or event logs. The choice is not
        ideological. It is driven by access patterns, consistency requirements, and team expertise.

## Question 6:
        Why use Mongoose instead of the native MongoDB driver?
        
        Beginner Answer: Mongoose provides schemas that define the shape of your data,
        validation that checks data before saving, and clean methods like .create() and
        .findById(). It makes working with MongoDB easier and more structured than using the
        raw driver.
        
        Experienced Answer: The native MongoDB driver is a thin wrapper over the database wire
        protocol. It gives you maximum flexibility but zero guardrails. You can insert { x: 1 } into
        a users collection and the driver will not complain. Mongoose adds an application-level
        contract through schemas. This matters because MongoDB itself is schemaless, which means
        data integrity becomes the application's responsibility. Mongoose handles type casting (a
        string "42" becomes the number 42 if the schema says type: Number), required field
        validation, custom validators, pre/post hooks (middleware that runs before or after save,
        update, delete), virtuals (computed properties), and population (resolving references
        between collections, similar to JOINs). The tradeoff is performance. Mongoose adds
        overhead through validation, hydration (converting raw BSON into full Mongoose
        documents with methods), and change tracking. For read-heavy, performance-critical paths,
        some teams bypass Mongoose and use the native driver or Mongoose's .lean() method,
        which returns plain JavaScript objects instead of Mongoose documents. In most application
        code, the safety and developer experience Mongoose provides far outweigh the minor
        performance cost.

## Question 7:
        What happens if you omit { new: true } in findByIdAndUpdate?
        
        Beginner Answer: Without { new: true }, findByIdAndUpdate returns the document
        as it was before the update. So the response would show the old data even though the
        database was updated. Adding { new: true } makes it return the updated version.
        
        Experienced Answer: This behavior comes from MongoDB's underlying findAndModify
        command, which defaults to returning the pre-modification document. The rationale is that
        you might need the original values for auditing, comparison, or rollback logic. Mongoose
        inherits this default. In API development, returning the old document after an update is
        almost never what you want to send to the client, so { new: true } becomes effectively
        mandatory for API responses. But the default behavior is useful in specific patterns. For
        example, in a queue system where you atomically "claim" a task by updating its status
        from "pending" to "processing," you want the pre-update document to confirm it was indeed
        "pending" when you grabbed it. Another important option is runValidators: true. By
        default, Mongoose does not run schema validators on update operations, only on create
        and save. This means a required field can be blanked out via findByIdAndUpdate unless
        you explicitly opt in. In production code, I always pass { new: true, runValidators:
        true } together.

## Question 8:
        Explain route parameters in Express. How are they different from query parameters?
        
        Beginner Answer: Route parameters are dynamic segments in the URL path, defined with a
        colon like /items/:id. They are accessed via req.params. Query parameters come after ?
        in the URL like /items?color=red and are accessed via req.query. Route parameters
        identify a specific resource, while query parameters filter or modify the request.
        
        Experienced Answer: Route parameters and query parameters serve different semantic
        purposes in REST API design. Route parameters identify a specific resource in the
        hierarchy: /users/42/orders/7 means "order 7 belonging to user 42." They are part of
        the resource's identity and the URL would be meaningless without them. Express matches
        them positionally using its route pattern, and they are available in req.params as strings.
        Query parameters are optional modifiers: /items?sort=name&limit=10&page=2 means
        "give me items, sorted by name, 10 per page, page 2." The base URL /items is still valid
        without them. They handle filtering, sorting, pagination, and search. In Express, req.query
        automatically parses the query string into an object. A common design mistake is using
        query parameters for resource identification (/items?id=42 instead of /items/42). This
        works technically but breaks REST conventions, makes URLs harder to cache, and loses the
        ability to use Express route matching for 404 detection. Another subtlety: Express route
        parameters can include regex constraints (/items/:id(\\d+)) to restrict matches to
        numeric patterns, providing route-level validation before the handler executes.

## Question 9:
        How do you handle errors in async Express route handlers? What happens if you do not?
        
        Beginner Answer: You wrap the handler code in a try/catch block. If something goes
        wrong (like a database error), the catch block sends a 500 status response. Without
        try/catch, an unhandled promise rejection could crash the server.
        
        Experienced Answer: Express was designed before async/await existed. Its error-handling
        mechanism relies on calling next(err) to pass errors to error-handling middleware. But
        with async handlers, if a promise rejects and there is no try/catch, Express never sees the
        error. The promise rejection goes unhandled, the client gets no response (the request hangs
        until timeout), and in Node.js 15+, an unhandled rejection can terminate the process
        entirely. The try/catch pattern works but creates boilerplate. Every handler repeats the
        same wrapping logic. A cleaner production pattern is a higher-order wrapper function:
        const asyncHandler = (fn) => (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
        };
        You then use it as app.get("/items", asyncHandler(async (req, res) => { ...
        })). Any thrown error automatically forwards to Express's error-handling middleware via
        next(err). Express 5 (currently in beta) handles this natively, automatically catching
        promise rejections from async handlers. Until then, the wrapper pattern or a package like
        express-async-errors is the standard production approach. Error-handling middleware
        itself has a distinct signature with four parameters (err, req, res, next) and must be
        registered after all routes. It centralizes error formatting, logging, and status code decisions
        in one place.