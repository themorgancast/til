# System Design

### Performant DOM Rendering
- Reflow trigger: Change to DOM tree or styles e.g. rendering content or making changes via DOM API
- Rendering Pipeline: Style/Layout (CPU), paint (GPU), Composite, initial DOM/CSS tree compilation
- Composite phase arranges DOM elements correctly
- Paint phase uses GPU, does not block rendering thread
- Render Layer: Packages child elements into the main renderobject if those elements do not have specific contexts such as `position: fixed` or having a CSS transformation applied. Can be inspected via Dev Tools => More Tools => Rendering
- Graphic Layer: Render layers applied to this. Used when we have acceleration such as transform properties. We have at least one attached to the `<html>` tag. Utilizes CPU. Go to Dev Tools => More Tools = Layers to see this.

### DOM API
- Window is our globally accessible object
- Assigned to `element` prototype to accommodate more than HTML elements e.g. SVG, XML.
- Querying builds a hashmap, has different time complexity, read cost, memory cost. Hashmap in query algorithm helps inform this. Browser will use that hashmap after a first pass.
- Best Practices
  - Simplify your selectors whenever possible
  - Use IDs on core containers
  - Avoid `innerHTML`, `appendChild` or `insertAdjacentElement` is more efficient, but all new element insertion is computationally expensive, all DOM modifications trigger a reflow
- Template elements and document fragments will prevent reflows when content within them is modified or inserted
- `cloneNode(true)` will create deep clone of template content, can help prevent triggering reflow when inserting content

### Observer API
- Three observers: __Intersection, Mutation, Resize__
- __Intersection__ use cases: Lazy loading, infinte scroll, scroll-based triggers/animations, visibility analytics, scroll spying, video autoplay
  - Anatomy: root, threshold, callback
- Intersection observers are ~50x faster than vanilla JS methods
- __Mutation__ observer watches for changes in the DOM tree
  - Faster than JS polyfills that create proxy objects to track mutations
  - ⚠️ Recursion can happen ⚠️ so stay frosty with how you use this
- Apps like Notion and Linear take heavy advantage of mutation observers where you can edit content without using a standard form
- __Resize__ observer tracks the sizes of elements, ideal for performantly creating adapative layouts on events such as window resize but can also listen for element tracking
  - Better than CSS queries when you need a callback
  - Much faster than JS `resize` event which relies on event bubbling
- You should debounce your callback to prevent it from executing too often, especially when doing things like applying styles

### Virtualization
- Maintain data in virtual memory while rendering only a subset
  - Minimizes elements in the DOM and the amount of mutations required, saving the CPU and GPU a lot of effort
- Use cases: "Sliding window" type UIs, handling large amounts of UI elements e.g. chat/message interfaces, image galleries, infinite scroll situations
- Recycling elements: Re-use already created components with new information based on which observer fires
- You can keep your data in a pool (array) or lazy load more data with optimistic rendering

### State Management Design
- KNOW YOUR SCALE and optimize accordingly
- Always start with how you structure your data
- Use normal forms to optimize access cost
- Use indexes if in-app search is required
- Offload data when needed
- Pick a suitable storage
- Data we use and store has two parametsrs: __Data Classes and Data Properties__
  - Classes: UI State. e.g. App Config, UI Element State, Server Data
  - Properties: Access Level, Read/Write Frequency, Size
- General Principles
  - Minimize access cost
  - Minimize search cost
  - Minimize RAM usage
- Principle 1: Data normalization __minimizes data access cost__
  - Optimizes access performance
  - Optimizes storage structure
  - Readable and maintainable
- Normal Forms can go 3 levels deep, each one building on the previous
  - 1NF: Make data atomic (no nested objects)
  - 1NF: Give data a primary key
  - 2NF: Non-primary keys depend on entity primary key
  - 3NF: Non-primary keys __only__ depend on entity primary key
  - In general, 2NF is more than sufficient for frontend
- Principle 2: Inverted index tables __minimize search cost__
  - Web workers or idle cycles can build this in the application
  - Memory offloading: Create shards to store data in pieces, query from the different shards.
  - Session Storage is good for non-persistent data
  - Local Storage is good for non-persistent, but provides persistence for preferences & configs. Only stores string type and is synchronous. Data we read/write to a lot will create a blocking thread.
  - Indexed DB has unlimited storage, has advanced search, stores multiple data types, nonblocking asynchronous operations.

## Network Connectivity
- You use these in conjunction with normal GET and POST requests
- Protocols: UDP and TCP
  - UDP is used heavily in video streaming, it will tolerate data loss or incomplete response data
  - TCP ensures the data requested by teh client and sent from the server is received in full
    - 3-way handshake: Client calls server, server acknowledges, client acknowledges
    - After this, client sends the request
    - http1.1, http2, Web Sockets, and Server Sent Events are TCP
  - UDP is faster than TCP
    - WebRTC, QUIK, http3 are UDP
- Ways to get data
  - __Long/Short Polling__: Making a GET call every interval
    - Cheap, fast, easy
    - Best for desktop applications, terrible for mobile
    - Long polling via TCP is slow b/c of 3way handshake protocol
    - Costs a lot of CPU time, drains energy keeping the TCP socket open
    - Latency is also a problem, state can get lost, reconnection has to happen on the client
  - __Server-Sent Events__ are http2 based, can re-use TCP connection
    - Useful for mobile
    - Only support receive-only mode
    - Scales easily, has a mutliplex feature (200 requests per TCP socket)
    - Doesn't send unnecessary headers
    - Can only send string data
    - Use cases: Desktop/mobile app where data needs to have minimal latency, can be an alternative to web sockets, good for large text data streaming (e.g. blog posts, articles)
  - __Web Sockets__ is the fastest way to transfer data between the client and server
    - Client sends handshake for http1, server sends upgrade response (upgrade to TCP), client push/server push continues
    - Pros: Almost real-time communication mechanism, unlimited number of connections
    - Cons: Very complex to maintain these connections, consumes resources, stateful, high infa + eng cost, drains energy and utilizes CPU, stateful so things get lost easily
    - Use cases: Working with sensors, online gaming, trading, precise location tracking, any situation where real-time updates are needed
  - Things like messaging apps would make benefit of server-side events and sending via normal POST requests

### REST and GraphQL
- GraphQL complicates things. Tread lightly!
- When to use REST: Small apps, widely use public API, requests are readonly, limited budget/team, isomorphic (closely shared) data types, bundle size matters
- When to use GraphQL: Large and complex apps, large teams, complex API model (expect to return different variations of the same object)
- Engineers can use one subscription service across all protocols that all trace back to GraphQL as the subscription provider

### Performance Optimization
__Largest Contentful Paint__ should be <2.5 seconds for good user experience
__Interaction to Next Pain__ should be <200ms for optimal user experience
__Cumulative Layout Shift__ should be 0.1 or less for visual stability

Performance Optimization Options:
- http1.1 has loading inefficiencies, constant data overhead (metadata), costs more in the long run in maintenance & migration
- http2 and http3 supports multiplexing over a single TCP connection, provides 98% greater efficiency

__Javascript Bundling__
- Most clients have >89% modern ES standards compatibility
- You can use polyfills to detect the browser and return optimized assets to reduce payload sizes (ES6/7, ES2025, etc)
- Bundle splitting is considered best practice now to load modules dynamically and reduce bundle sizes and load times
- Advanced Prefetching: Tells browser to fetch the asset and cache it in the browser
  - Preload is for high priority
  - Prefetch is for background loading/lower priority
- Code minification and compression with compacting can reduce memory by 20%
- Use `defer` to load after other assets if you have things that are lower priority, e.g. Analytics, Telemetry scripts

__Styles and Images__
- Optimize by loading critical styles when our GET /index.html happens
  - Do this with the `<style>` tag in the head of Index to load critical styles immediately
  - `<link rel=>` tags are for non-critical styles that can load later, good practice is using the `preload` marker
- For images, start with choosing the right format (MP4, SVG, webp, PNG, JPG, AVIF)
- Browsers wait 3 seconds for custom fonts to load first
- Using `font-display: fallback` or `font-display: optional` will render unstyle text immediately, fallback will switch the font, optional will only on refresh

### CAP Theorem
Consistency, Availability, and Partition Tolerance
Pick two
- __Consistency:__ Every read receives the most recent write or an error
- __Availability:__ Every request receives a response, without guarantee that it contains the most recent version of the information
- __Partition Tolerance:__ The system continues to operate despite arbitrary partitioning due to network failures


### How to Rock a System Design Interview
- Stay framework agnostic, e.g. do not lean exclusively on React Hooks or Svelte Slots or any other libraries
- Gather requirements and maybe put together a visual mockup
  - Functional requirements
  - Nonfunctional requirements
  - Technical context
  - Accessibility considerations
- Go over state management and structure
  - What properties do we want to store in state
  - Plan to optimize how we access data
  - How do we make sure we are accessing the correct data (e.g. by ID)
- API Structure
  - What endpoint(s) do we need?
  - What data schema do we need?
  - How do we want to send and receive data?
    - Short poll, web sockets, server-sent events
    - When do we use regular GET and POST
    - Web sockets feels like the easy answer, but is it?
  - What downstream services do you need?
- Optimizations
  - Network optimizations, which protocol?
  - Package splitting and compression
  - Polyfills or no?
  - What does your rendering tree look like from the DOM down(images, fonts, styles, etc)?
  - How to handle placeholders?
  - What triggers reflows and how can we focus on GPU over CPU?
  - Make sure your UI thread stays open! Be as async as possible!
  - Implement service workers that can cache assets in Indexed DB or other session/local storage
  - Find ways to reduce JS payload with bundle splitting and deferments
  - What fallbacks do you need?
  - A11y workarounds, tradeoffs, other approaches