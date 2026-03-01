# style-guide

## Sources
- A Philosophy of Software Design
- A Philosophy of Software Design vs Clean Code
- Worse is Better
- The Grug Brained Developer
- Codin' dirty


## Principles and Assumptions

- The primary concern of the code writer needs to be the experience of future code readers.
  - Code needs to work, obviously, and we have means to ensure that (automated and user testing). But no one is in a better position that the one writing the code to make it easier to read.
  - The future operation of the code [may be more important](https://olano.dev/blog/code-is-run-more-than-read) than its reading, but that's the focus of design and other activties, it's not something that needs to be a primary focus in the structuring of the core.s
  - Maybe it's LLMs that end up reading the code, not humans, but the assumption here is that the more readable code is for humans, the more effective LLMs will be at processing. I won't sacrifize human readability for alleged LLM optimizations.
- For any piece of code, there should be enough context in the codebase for a reader to answer "what is this?" and "why is this here?".
- Accidental complexity needs to be removed, essential complexity needs to be managed.
- Modularity is the primary tool to manage complexity.
- Modules should be deep.
  - Interface complexity costs more than implementation complexity.
  - Breaking modules apart typically increases interface complexity.
  - Reducing local complexity at the expense of global complexity is usually not a good trade off
- Abstractions help with modularity, but each new abstraction (each new "concept") has a cognitive cost on code readers.
- Code duplication is not necessarily a problem, knowledge duplication always is.

## Project and directory structure

- The directory structure should be intentional and meaninful

- In most cases it's better to group things by domain than by its technical features
  - eg put the employee enum next to the employee data structure, not next to other enums
      - <in reader terms: I will want to more frequently look at the employee together with the enum than to all project enums as a group>

## File structure

- The files should be readable from top to bottom
  - this means that important (higher level) stuff should come first
      - primary exported things at the top, private helpers at the bottom
  - When opening a file, I should be able to quickly answer what this and why here. this can be done in multiple ways:
    - the containing package and the filename. if this is `app/routes/employees.py` I know these will be API endpoints related to employees, no need extra commenting on the file.
    - If there's a single or primary struct or class at the top of the file, its docstring may provide the answers.
    - Otherwise a top-level docstring should provide the answers.

## Docstrings
- docstrings are part of the interface of a module.
- docstrings should not reiterate the rest of the interface.
- docstrings should not reveal implementation details.

## Comments

- comments are not a smell, are a tool for communication
- comments should complement the code, not repeat it (they provide the why, not the what)

### TODO and FIXME comments



## Names

- method should assume the classname as its namespace
- if the language supports it, function should preferably assume the classname as its namespace
- names should be meaningful
    - short names are not necessarily bad
    - excessively long names are distracting

## Tests
