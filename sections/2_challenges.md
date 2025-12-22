## 2. Challenges

Integrating Claude Code into an existing codebase comes with two core challenges.

**Context window limits**

Claude Code has a 200k token context window—less than 5% of this project's codebase. You must carefully select which information to provide: include code that shows your patterns; exclude anything irrelevant. Too little context leads to incorrect implementations; too much leads to context pollution and degraded performance.

**Balancing speed, quality, and oversight**

Implementation with an LLM should be at least as fast as doing it yourself—otherwise, you lose the benefit. This creates a tradeoff:

- **Quality**: Early on, Claude produces inconsistent code that doesn't follow your patterns—forcing refactoring or detailed prompt crafting.
- **Oversight**: Small tasks can run with minimal supervision; architectural changes need close attention.

The challenge is developing intuition for when to step in and when to let Claude work autonomously.
