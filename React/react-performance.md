# React Performance

### Some Core Principles

Keep it simple! Less is more.

Don't just memoize everything. Consider the performance costs and trade-offs of your choices.

Sometimes the issue isn't React, it's that you aren't making good decisions

### Anonymous Functions and Rendering Cycles
- React reads top-down
- Don't put your hooks in conditionals, React will yell at you
- Wrapping event handlers in an anonymous function will make sure JS doesn't execute slow code paths or create 1000x executions killing us slowly in our sleep
- Primitives are NOT subject to this

### Fibers
- These are data structures that React uses to keep track of component instances
- This is why having *unique identifiers* matters, otherwise we will die trying to figure out what changed
- React keeps the history of your hooks
- Don't use dumb shit like `math.random()` or array indices or anything else that's going to change

