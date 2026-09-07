## Question 1: 
    What happens when you type a URL and press Enter? Include DNS, connection setup, request, response, and rendering.

    Beginner: When you type a URL and press Enter, the browser first uses DNS to convert the domain name into an IP address. It then establishes a connection with the server, sends an HTTP request, and receives an HTTP response. The browser processes the response, downloads required resources such as CSS, JavaScript, and images, and then renders the page on the screen.

    Experienced: The process involves several stages. DNS resolves the domain to an IP address. The browser then establishes a connection with the server, typically using TCP and TLS for HTTPS. It sends an HTTP request containing information such as the method, path, and headers. The server returns an HTTP response containing a status code, headers, and usually the HTML document. The browser parses the HTML into the DOM, parses CSS into the CSSOM, combines them to determine what should be rendered, and performs layout and painting. JavaScript can modify the DOM and trigger additional network requests or rendering work. Understanding this pipeline helps explain performance issues such as slow DNS, connection latency, render-blocking CSS, and JavaScript delaying rendering.

## Question 2: 
    Why does the browser not simply receive a screenshot from the server?

    Beginner: A browser does not receive a screenshot because web pages are interactive and need to adapt to different devices and screen sizes. The server usually sends HTML, CSS, JavaScript, images, and other resources. The browser uses these resources to build and render the page locally.

    Experienced: Sending a screenshot would make the page essentially a static image. The browser would not be able to provide normal interactions such as clicking buttons, submitting forms, selecting text, changing content with JavaScript, or adapting the layout to different screen sizes. By receiving HTML, CSS, and JavaScript separately, the browser can build the DOM, apply styles, execute logic, respond to user interactions, and re-render parts of the page when necessary. This separation also allows browsers to cache resources and developers to create responsive, accessible, and dynamic applications.

## Question 3: 
    What is the difference between HTML source code and the DOM shown in DevTools?

    Beginner: HTML source code is the original markup sent by the server or written in the HTML file. The DOM is the browser's in-memory representation of that HTML. JavaScript can modify the DOM after the page loads, so the DOM shown in DevTools may be different from the original HTML source.

    Experienced: The HTML source represents the initial document received by the browser, while the DOM represents the document structure currently maintained by the browser. During parsing, the browser can also correct invalid HTML or implicitly create elements, so the resulting DOM is not always an exact copy of the source. JavaScript can further add, remove, or modify elements and attributes, which changes the DOM without changing the original source file. This is why inspecting an element in the Elements panel may show changes that cannot be found in View Source. DevTools primarily shows the current live DOM, which is what the browser is actually working with.

## Question 4: 
    What is the difference between a 200, 404, and 500 response?

    Beginner: A 200 response means the request was successful. A 404 means the requested resource could not be found on the server. A 500 means the server encountered an internal error while trying to process the request.

    Experienced: These status codes represent different stages of failure or success. 200 means the server successfully processed the request and returned the expected response. 404 means the server was reachable and understood the request, but the requested resource does not exist at that location. 500 indicates a server-side failure while processing an otherwise valid request. A common debugging mistake is treating all errors as frontend problems. The Network tab in DevTools helps identify whether the browser received a successful response, requested a missing resource, or encountered a server-side failure.

## Question 5: 
    Why are DevTools changes temporary unless you update the source file?

    Beginner: DevTools lets you modify the current page running in the browser, but those changes only affect the browser's current copy of the page. They do not modify the original HTML, CSS, or JavaScript files on the server or on your computer. When you refresh the page, the browser loads the original source again and the changes disappear.

    Experienced: DevTools is primarily an inspection and debugging environment, not a source-code editor. When you change a CSS property or modify the DOM through DevTools, you are changing the browser's current in-memory representation. The original source files remain unchanged. A page refresh causes the browser to load and process those original files again, so the temporary changes are lost. This distinction is important during debugging: use DevTools to experiment and identify the correct change, then apply that change to the actual source file so it becomes persistent.

## Question 6:
    What is semantic HTML and why does it matter?

    Beginner: Semantic HTML uses tags that describe the meaning or purpose of their content. For example, nav for navigation, section for grouped content, article for self-contained items. It makes the page more readable for developers and easier for tools like DevTools and search engines to interpret.
    
    Experienced: Beyond readability, semantic HTML creates document landmarks that screen readers use to help users navigate the page. It contributes to SEO by giving search engines structured signals about content hierarchy. It also reduces the cognitive load for developers maintaining the codebase, since tags like header and footer communicate intent without comments or class names.

## Question 7:
    What is the difference between section, article, and div?
    
    Beginner: section is for a block of related content within the page. article is for a self-contained item that could stand on its own. div is a generic container with no semantic meaning, used when nothing more specific fits.
    
    Experienced: The key test for article is: could this content make sense if lifted out of the page and published elsewhere? A blog post, a project entry, a news item. If yes, it is an article. section groups content that belongs together thematically but is not self-contained. div has no meaning and should only be used for layout or styling hooks when no meaningful tag exists.

## Question 8:
    Why must label for match input id?
    
    Beginner: The for attribute on a label points to the id of its input. When they match, clicking the label moves focus to the input. When they do not match, the label appears visually but has no connection to the input in the browser's understanding.
    
    Experienced: The label-for-id connection is also how assistive technologies associate a visible text description with a form control. Screen readers read the label text before announcing the input type. Without this connection, screen reader users hear only the input type with no context. This is a common form accessibility bug found in production code.

## Question 9:
    What is the difference between id and name on a form input?
    
    Beginner: id is used to uniquely identify an element on the page. It is used by label for to connect the label, and by href="#id" for same-page navigation. name is what the form uses to identify the data when it is submitted.
    
    Experienced: id and name serve different audiences. id is for the browser's DOM and CSS selectors. name is for the server or form handler receiving the data. Both should be set on form inputs. A field with only an id but no name will not have its data included in the form submission.

## Question 10:
    What browser validation attributes does HTML provide natively?
    
    Beginner: HTML has four main validation attributes: required to block submission if empty, type="email" to validate email format, minlength to enforce a minimum character count, and maxlength to cap input length.
    
    Experienced: Native HTML validation runs before any JavaScript. It is reliable, zero-cost, and accessible by default. It should always be the first layer. JavaScript validation adds more complex rules and custom error messages on top. The mistake to avoid is relying only on JavaScript and skipping native attributes, which leaves a validation gap if JavaScript fails to load.

## Question 11:
    Why must all radio buttons in a group share the same name?
    
    Beginner: The browser groups radio buttons by their name attribute. Buttons with the same name are treated as one group, meaning only one can be selected at a time. Different names create separate independent groups, which means multiple options can be selected simultaneously.
    
    Experienced: This is a frequent bug in production forms. It often appears after copy-pasting a radio button and forgetting to update the name. The symptom is that multiple options stay selected. The fix is always to confirm that every radio button in a mutually exclusive group shares exactly the same name value.

## Question 12:
    What is the difference between a placeholder and a label?
    
    Beginner: A placeholder is hint text inside the input that disappears when the user starts typing. A label is a persistent visible text element connected to the input. A placeholder cannot replace a label.
    
    Experienced: Placeholders fail on two counts. First, they disappear on input, meaning users with short-term memory issues or users who make mistakes cannot see what field they are in. Second, many screen readers do not reliably read placeholder text as a field description. The accessibility-correct pattern is always to include a visible label and use placeholder only as an optional supplementary hint.

## Question 13:
    What does the CSS cascade mean?
    
    Beginner: When multiple rules target the same element, the browser follows rules to decide which one applies. A class selector beats an element selector. An ID beats a class. When specificity is equal, the later rule wins.
    
    Experienced: The cascade resolves conflicts in a specific order: specificity first, then source order. A crossed-out rule in DevTools is not an error: it is the rule that lost. Understanding this means you debug by reading the winning rule rather than guessing why a style is not applying.

## Question 14:
    What is the difference between class and ID selectors?
    
    Beginner: Classes are reusable and can be applied to multiple elements. IDs must be unique on the page. Classes use a dot, IDs use a hash in CSS.
    
    Experienced: IDs carry higher specificity than classes. Because of this, using IDs for styling creates rules that are difficult to override without escalating to !important. The professional convention is to use classes for all styling and reserve IDs for JavaScript hooks and anchor links.

## Question 15:
    Explain the box model.
    
    Beginner: Every element is a box with four layers: content, padding, border, and margin. Padding is space inside the element. Margin is space outside. Border is the visible edge.
    
    Experienced: The default box model adds padding and border on top of the declared width, so a 300px element with 20px padding is actually 340px wide. Setting box-sizing: border-box globally fixes this by including padding and border inside the declared width. Every major CSS framework does this.

## Question 16:
    What is the difference between padding and margin?
    
    Beginner: Padding is space inside an element between its content and its border. Margin is space outside the element between it and other elements.
    
    Experienced: Vertical margins between adjacent block elements collapse into a single margin equal to the larger of the two, not the sum of both. This is called margin collapse and is a common source of unexpected spacing. It does not happen horizontally, and it does not apply inside flex or grid containers.

## Question 17:
    What is the difference between display: none and visibility: hidden?
    
    Beginner: display: none removes the element completely from the layout, taking up no space. visibility: hidden makes the element invisible but it still occupies space in the layout.
    
    Experienced: A third option, opacity: 0, hides the element visually but keeps it in the layout and it remains interactive (clickable, focusable). The choice matters for layout stability and accessibility: screen readers handle these three differently depending on the browser.

## Question 18:
    What is the difference between display: none and visibility: hidden?
    Beginner: display: none removes the element completely from the layout. It takes up no space and nothing fills the gap. visibility: hidden makes the element invisible but it still occupies its space on the page.
    Experienced: A third option, opacity: 0, makes the element visually transparent but keeps it in layout and keeps it interactive (clickable, focusable). The right choice depends on whether you need the space preserved and whether the element should still be reachable by keyboard or screen reader. display: none is universally excluded from accessibility trees; visibility: hidden may still be read by some screen readers.

## Question 19:
    What is the difference between position: absolute and position: fixed?
    
    Beginner: absolute places an element relative to its nearest non-static ancestor. fixed places it relative to the viewport. An absolute element scrolls with the page. A fixed element stays in the same position while scrolling.
    
    Experienced: absolute is removed from normal flow and uses the nearest positioned ancestor as its coordinate system. If no positioned ancestor exists, it falls back to the initial containing block (the viewport root). fixed always uses the viewport, which is why a fixed element appears to float on screen. On mobile devices, fixed can behave unexpectedly with virtual keyboards and URL bars, which is a real-world edge case worth knowing.

## Question 20:
    Why is position: relative used on a parent when the child is position: absolute?
    
    Beginner: An absolutely positioned element uses the nearest ancestor whose position is not static as its coordinate reference. Setting position: relative on the parent makes it that reference without moving it visually.
    
    Experienced: Without position: relative on an intended parent, the absolutely positioned child walks up the DOM tree looking for any non-static ancestor. If it finds one several levels up, or defaults to the viewport, the badge or tooltip ends up in the wrong place entirely. This is the most common positioning bug. The fix is always to confirm that the intended containing parent has position: relative explicitly set.

## Question 21:
    What does box-sizing: border-box do?
    
    Beginner: It changes how width is calculated. By default, padding and border are added on top of the declared width. With border-box, they are included inside it. A 300px element with 24px padding stays 300px wide.
    
    Experienced: The default model (content-box) causes layout calculations to break whenever padding or border is added or changed. border-box makes width predictable because the declared value is the actual rendered width regardless of padding or border. Applying it globally with the universal selector is standard practice. Tailwind and Bootstrap both do this. The performance concern about the universal selector is negligible in modern browsers.

## Question 22:
    What is the difference between block, inline, and inline-block?
    
    Beginner: Block elements take the full width and start on a new line. Inline elements sit in the text flow and ignore width and height. Inline-block sits in text flow like inline but respects width, height, and all four sides of margin and padding.
    
    Experienced: The distinction matters most when you want elements side by side with controlled sizing. Setting display: inline-block on nav links or badges allows them to flow horizontally while still responding to padding and margin. The limitation of inline-block is that whitespace between elements in HTML creates small gaps in the rendered output, which is one reason Flexbox is generally preferred for multi-item layout in modern code.

## Question 23:
    When should you use positioning vs normal flow?
    
    Beginner: Positioning is for special placement of specific elements: badges inside cards, floating buttons, sticky headers. Normal flow handles most of the page layout.
    
    Experienced: A common beginner mistake is using absolute or fixed positioning to arrange the main page structure because it feels precise. This creates fragile layouts that break on different screen sizes and removes elements from the flow that other elements depend on. The professional approach is to use normal flow for overall structure, then positioning only for elements that genuinely need to step outside that flow. The rule of thumb: if you are placing a single specific element relative to its container, use positioning. If you are arranging a group of elements, use Flexbox or Grid.

## Question 24:
    What is the difference between justify-content and align-items?
    
    Beginner: justify-content aligns items along the main axis, which is horizontal when flex-direction is row. align-items aligns items along the cross axis, which is vertical when flex-direction is row.
    
    Experienced: Both properties are axis-relative, not direction-absolute. When flex-direction changes to column, justify-content controls vertical distribution and align-items controls horizontal alignment. A common interview follow-up: what does justify-content: center do when flex-direction is column? It centres items vertically, not horizontally.

## Question 26:
    What is the difference between flex-grow, flex-shrink, and flex-basis?
    
    Beginner: flex-grow controls how much an item expands into available space. flex-shrink controls how much it compresses when space is tight. flex-basis sets the starting size before growing or shrinking.
    
    Experienced: flex: 1 is shorthand for flex-grow: 1, flex-shrink: 1, flex-basis: 0%. The 0% basis means items start from zero and grow equally into available space. flex: auto expands to flex-grow: 1, flex-shrink: 1, flex-basis: auto, where items start from their content size and then grow. The difference matters when items have unequal content: flex: 1 makes them equal width, flex: auto distributes remaining space proportionally from their natural sizes.

## Question 27:
    Why is Flexbox called one-dimensional?
    
    Beginner: Flexbox arranges items along one axis at a time: either a row or a column. It does not control both simultaneously.
    
    Experienced: One-dimensional means Flexbox cannot enforce alignment across multiple rows or columns at once. When items wrap with flex-wrap, each row is an independent flex line. Items in row 2 do not align with items in row 1 unless they happen to be the same size. For two-dimensional alignment (rows and columns together), CSS Grid is the right tool. Flexbox and Grid are complementary: Flexbox for single-axis component layout, Grid for two-dimensional page structure.

## Question 28:
    What does flex: 1 actually mean?
    
    Beginner: It is shorthand for flex-grow: 1, flex-shrink: 1, flex-basis: 0%. It makes the item grow to fill available space, starting from zero.
    
    Experienced: The key is flex-basis: 0%. This means all items start with a base size of zero and then grow proportionally. If all items have flex: 1, they end up equal width regardless of content size. If one item has flex: 2 and another has flex: 1, the first gets twice the available space. This is different from flex: auto (basis is the content size), where larger content produces a wider final size even with equal grow values.

## Question 29:
    When would you use align-self instead of align-items?
    
    Beginner: align-items applies the same cross-axis alignment to all flex items. align-self overrides that for a single item. Use align-self when one item in a group needs different alignment than the rest.
    
    Experienced: A common real-world pattern: a flex navbar where all items are vertically centred with align-items: center, but a badge or notification indicator needs to sit at the top. Setting align-self: flex-start on just the badge achieves this without changing the container rule. align-self accepts the same values as align-items plus auto, which inherits from the container.

## Question 30:
    What is the difference between gap and margin for spacing flex items?
    
    Beginner: gap is applied to the container and adds space between every adjacent pair of items with no extra space at the edges. margin is applied to individual items and adds space around each item, including before the first and after the last.
    
    Experienced: gap is the preferred approach for Flexbox spacing because it is semantically correct (the container controls layout), it does not produce edge overflow, and it handles both row and column gaps when items wrap. The old margin-right pattern required a separate rule to remove the margin from the last item, or used the lobotomised owl selector (* + *). gap eliminates both workarounds. Note: gap also works in CSS Grid and between block elements in modern browsers.

## Question 31:
    What is the difference between CSS Grid and Flexbox?
    
    Beginner: Flexbox is one-dimensional: it arranges items in a row or a column. Grid is two-dimensional: it arranges items in rows and columns at the same time. You use Flexbox for components like navbars and badge lists, and Grid for page-level layouts and galleries.
    
    Experienced: Flexbox distributes space along a single axis. Grid defines a coordinate system with explicit or implicit tracks in both dimensions. They are complementary: you typically use Grid for overall page structure and Flexbox for the layout within each component. Flexbox has no concept of aligning items across multiple rows because each row is independent. Grid can align items across both rows and columns, which is what makes it the right tool for grids.

## Question 32:
    What is the fr unit in CSS Grid?
    
    Beginner: fr stands for fraction. It represents a proportional share of the available space in the grid container. Three columns each set to 1fr each get one-third of the container width.
    
    Experienced: The browser calculates fr values after subtracting any fixed-size tracks and gaps from the total available space. This means fr values flex with the container, making them better than percentage values in most Grid contexts because percentages do not account for gap widths. repeat(3, 1fr) is one of the most common Grid patterns in production code.

## Question 33:
    What are CSS custom properties and why use them?
    
    Beginner: CSS custom properties are variables declared with a -- prefix on :root that store reusable values. You reference them with var(). Changing the value in one place updates every reference across the stylesheet.
    
    Experienced: CSS custom properties are resolved at runtime in the browser, unlike Sass variables which compile away before the browser sees them. This means they can be changed dynamically with JavaScript, scoped to specific components, and overridden inside media queries. They are the foundation of design token systems and CSS-based theming, including dark mode implementations.

## Question 34:
    What is BEM and why do teams use it?
    
    Beginner: BEM stands for Block, Element, Modifier. It is a CSS naming convention where every class name includes the component it belongs to (block), the part it represents (element after __), and any variation (modifier after --). It prevents naming collisions and makes classes self-documenting.
    
    Experienced: Without a naming system, CSS class names become ambiguous as a project grows. Generic names like .title or .active can refer to dozens of unrelated things. BEM eliminates ambiguity: .project-card__title tells you immediately which component, which part. It also prevents accidental style leaking between components, since each component's elements are namespaced under the block name.

## Question 35:
    What is the difference between implicit and explicit grid rows?
    
    Beginner: Explicit rows are defined by the developer using grid-template-rows. Implicit rows are created automatically by the browser when more items exist than the defined grid structure can hold.
    
    Experienced: For most page layouts, you define columns explicitly with grid-template-columns and let rows be implicit. The browser creates new rows as needed, sizing them to fit their content by default. You can control the size of implicit rows using the grid-auto-rows property, for example grid-auto-rows: 200px makes all auto-created rows a fixed height. This is important for dashboards and galleries where row consistency matters.

## Question 36:
    What is grid-template-areas and when would you use it?
    
    Beginner: grid-template-areas lets you define a page layout as a text map. You name regions in a quoted string pattern where each word is a column and each quoted line is a row. Child elements connect to their region using grid-area.
    
    Experienced: grid-template-areas is most useful for page-level layouts with named, distinctly sized regions: header, sidebar, main content, footer. For identical repeating items like a card gallery, auto-placement with repeat() is simpler. The real power of grid-template-areas appears in responsive design, where different area maps are defined for different screen sizes inside media queries, rearranging the entire page layout without touching any child element's CSS.

## Question 37:
    What is the viewport meta tag and why does it matter for responsive design?
    
    Beginner answer:  It tells mobile browsers to use the device's actual width as the viewport, rather than pretending the page is ~980px wide. Without it, media queries fire at wrong breakpoints and the page looks like a shrunken desktop site on phones.
    
    Experienced answer:  Mobile browsers historically used a "virtual viewport" (usually 980px) to render desktop-era sites, then scaled them down. The viewport meta tag disables this by setting the layout viewport to device-width. initial-scale=1.0 prevents the initial zoom. The distinction matters because CSS media queries evaluate against the layout viewport, not the physical screen pixels. On a high-density display like an iPhone with 3x pixel ratio, device-width still reports in CSS pixels (390 CSS px on iPhone 14), not physical pixels (1170 physical px).

## Question 38:
    What is the difference between px, rem, em, and vw? When would you use each?
    
    Beginner answer:  px is absolute. rem is relative to the root font size (16px default). em is relative to the current element's font size. vw is 1% of the viewport width. Use px for borders, rem for font sizes and spacing, vw for full-width or hero elements.
    
    Experienced answer:  em is inherited from the current element, which creates compounding: if a parent has font-size: 1.25em and a child also has 1.25em, the child ends up at 1.25 x 1.25 = 1.5625em of the root. rem avoids this by always referencing the html element. For accessibility, px font sizes prevent browser zoom from working properly because most browsers implement zoom by scaling the root font size, not by multiplying px values. rem values inherit the zoom; px values do not. This is why WCAG recommends rem or em for font sizes.

## Question 39:
    What is the difference between mobile-first and desktop-first responsive design?
    
    Beginner answer:  Desktop-first writes base CSS for large screens and uses max-width media queries to handle smaller screens. Mobile-first writes base CSS for small screens and uses min-width media queries to add complexity for larger screens.
    
    Experienced answer:  Mobile-first is preferred in production for three reasons. First, progressive enhancement: if media queries fail, users get a usable mobile layout rather than a broken desktop layout. Second, performance: mobile devices are often on slower connections. Base CSS is the smallest and simplest stylesheet; extra styles only load for devices that can handle them. Third, specificity: adding rules for larger screens (min-width) is generally cleaner than overriding rules for smaller screens (max-width). The mental model shift is: start with constraints, then add possibilities.

## Question 40:
    Why does the order of media queries matter in a stylesheet?
    
    Beginner answer:  CSS uses source order to resolve conflicts when specificity is equal. A base rule that appears after a media query will always override the media query, even when the media query is active. Media queries must go at the end of the stylesheet.
    
    Experienced answer:  This is a manifestation of the CSS cascade. Specificity is evaluated first. When two rules have equal specificity, the one that appears later in the source wins. A media query does not increase a selector's specificity; it is a conditional wrapper around a normal rule. So @media (max-width: 768px) { .grid { columns: 1 } } followed by .grid { columns: 3 } will always produce 3 columns, because the non-conditional rule comes later. This also applies to multiple overlapping media queries: a 500px screen matches both (max-width: 1024px) and (max-width: 768px). Both blocks apply; the later one wins for any conflicting properties.

## Question 41:
    What does flex-direction: column do in a media query, and why is it so commonly used for responsive design?
    
    Beginner answer:  It changes a horizontal (row) flex layout into a vertical (column) stack. On mobile, most elements that sit side by side on desktop need to stack vertically because horizontal space is limited.
    
    Experienced answer:  Flexbox's direction model is what makes it well suited to responsive design. The main axis switches from horizontal to vertical, but all other flex properties (gap, align-items, justify-content) continue to work relative to the new axis. This means a single property change restructures the entire layout. The about section, the navbar, and the nav links in Class 7 all used this pattern. The cascading effect of axis change also means that align-items: center (which centered items vertically in row mode) now centers them horizontally in column mode, often achieving the desired centered-stack layout with no additional rules.

## Question 42:
    What is the img { max-width: 100%; height: auto; } rule and why is it applied globally?
    
    Beginner answer:  It prevents images from overflowing their parent container. max-width: 100% means an image can never be wider than its parent. height: auto preserves the aspect ratio when the width changes.
    
    Experienced answer:  Images have natural dimensions defined by their source file. Without constraints, an image with a natural width of 1200px renders at 1200px regardless of its container width, causing horizontal overflow. max-width: 100% resolves this at any container size. height: auto is critical because if width shrinks and height stays fixed, the image distorts. The global selector applies this to all images in the document, providing a safe default that individual components can override if needed. This rule is typically placed early in the stylesheet as a global reset. It is part of most CSS resets and normalizers precisely because it solves such a ubiquitous responsive bug.

## Question 43:
    What is CSS specificity and how is it calculated?
    
    Beginner: Specificity is how the browser decides which rule wins when several rules target the same property on the same element. Scoring goes: inline styles strongest, then IDs, then classes (including pseudo-classes and attributes), then elements. Ties break by source order (last one wins).
    
    Experienced: A four-tier weighting system written as (inline, IDs, classes, elements). Tiers are non-overflowing: 100 element selectors never outweigh a single class. Combinators and * contribute zero. The architectural strategy is keeping all selectors at a similar low specificity (single classes) so cascade order becomes the predictable tiebreaker. IDs force escalation. !important creates its own tier above inline styles, but the same comparison applies inside it, so it solves nothing structurally.

## Question 44:
    What is the difference between the descendant selector and the child selector?
    
    Beginner: Descendant (space) matches at any depth. .card p matches a p whether it is a direct child or deeply nested. Child (>) matches only direct children, one level deep.
    
    Experienced: Descendant traverses the entire subtree, flexible but prone to unintended matches as components grow. Child restricts to one level, providing encapsulation. In BEM, CSS Modules, or scoped styles, the child combinator prevents parent styles from leaking into nested sub-components. The tradeoff is brittleness: adding a wrapper div breaks >. Choose based on whether encapsulation or structural flexibility matters more.

## Question 45:
    Why should you avoid !important?
    
    Beginner: It overrides everything, which makes it very hard to override later. If two rules use it, you are back to specificity comparison anyway, except now everything needs !important to compete. Hard to maintain.
    
    Experienced: It breaks the natural cascade by elevating a declaration above the tier system. Once one declaration uses it, competitors must too, creating an order-dependent arms race. Legitimate uses: utility classes in a design system (.hidden { display: none !important; }) and overriding third-party CSS you cannot edit. In your own codebase, needing !important is a code smell pointing to a specificity architecture problem to be fixed by flattening selectors, not escalating them.

## Question 46:
    Which CSS properties inherit, and why does it matter?
    
    Beginner: Text properties (font-family, color, line-height) inherit. Layout properties (margin, padding, border) do not. This lets you set typography once on body and have every element use it unless overridden.
    
    Experienced: Inheritance follows the DOM tree and applies to properties where per-element repetition would be impractical. Non-inheriting properties are those where inheritance would cause chaos (every child copying its parent's border). Practically, inheritance is the first cascade layer to leverage: a well-structured body rule removes dozens of redundant declarations. The classic pitfall is links: the user-agent stylesheet sets an explicit color on a, beating inheritance because an explicit rule always wins.

## Question 47:
    Where should the transition property go, and why?
    
    Beginner: On the base state, not on :hover. That way the animation plays both on hover-in and hover-out. On :hover only, it animates in but snaps back instantly.
    
    Experienced: Placing transition on the base state ensures bidirectional animation because the definition persists in both states. On :hover, the definition vanishes the 
    moment the cursor leaves, so the browser has no instructions for the return trip. There is a deliberate use case: a longer transition on :hover and a shorter one on the base state creates an asymmetric effect (eases in slowly, snaps back quickly). For standard UI, base state, named properties (never all), 150ms to 400ms.

## Question 48:
    What is the difference between :hover and :focus? Can both be active at once?
    
    Beginner: :hover is about the mouse pointer being over an element. :focus is about keyboard focus (or clicking into an input). Yes, both can be active at once: if you click a button, it is both hovered and focused, and styles stack.
    
    Experienced: :hover reflects pointer position and does not exist on touch devices, so it should never gate essential information. :focus reflects keyboard or programmatic focus and is an accessibility requirement, not decoration. They are independent and can co-occur, which is why declaration order (:hover, :focus, :active) matters when they share specificity. In production, prefer :focus-visible to show focus rings only for keyboard users.



