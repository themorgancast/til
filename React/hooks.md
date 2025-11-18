# React Hooks

## Introduces interactivity and side effects to components

### Hook Rules
- They can NEVER be inside of conditionals or loops e.g. for, while, do
- Declare at the top level of your component
- Will be called in the same order every time (Linting will save you from yourself on this one)

## Hooks in React

### useState
Allows functional components to declare and manage variables. Returns an array with the current state and a setter to update to a new value.

_Note: You can set event listeners on the parent but this does not fulfill a11y standards especailly on form input fields, so best to put listeners on individual elements_

### useReducer

### useEffect
Enables side effects and rendering inside of functional components. Runs after every render by default, but can be controlled with a dependency array that will trigger function re-run. 2nd argument can be blank which tells it to only render on initial component load and never again.

Use cases: Trigger animation on form submission, fetching data on component initial load outside of the render cycle, canceling a request/timeout (in the return/cleanup)

### useContext
Subscribes to React Context, shares data across/down component tree easily. Prevents losing information and state when components get blown away. Avoids prop drilling.

Use cases: Themes, a11y, user information, shopping cart contents, login information, libraries like react-router or TanStack

### useRef

### useMemo
Caches result of expensive calculations to prevent expensive re-renders

### useCallback
Caches a function's definition, prevents re-creation of function on every render

Use cases: Passing functions as props to child components

### useOptimistic
_Note: Only in React 19_

### useActionState
_Note: Only in React 19_

## Custom Hooks
Use cases: Share logic between components

Examples in the wild: debouncing, local storage, media queries, data fetching, themes, auth, etc