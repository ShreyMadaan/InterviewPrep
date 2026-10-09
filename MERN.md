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