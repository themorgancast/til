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