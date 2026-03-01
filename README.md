# Facundo's highly controversial programming style guide

## Introduction

This is a brain dump of my current coding preferences. I try to adhere to these in personal projects and I try to subtly nudge team projects in this direction when it's not disruptive.

I occassionally found myself sending LLMs to read one of my blog posts before starting some task (e.g. for testing conventions), so rather than crafting weird `AGENTS.md` files I though why not make a human readable guide to refer to. If it's human-readable enough surely Claude can handle it.

Why even bother with sytle if it looks like we won't be writing much code anymore? Maybe we won't, but it's likely that we still have to read a lot of it, so I'd rather have the LLMs follow my preferences. I also posit that better style makes better design, and better design leads to better quality output from LLMs. And [if it does turn out](https://olano.dev/blog/dangerously-skip/) that we won't need to even read code anymore, then indulge me on this, as my own personal farewell to that aspect of the craft.

Why so much discussion about software design in a style guide: I think that style is a spectrum that goes from trivial formatting decisions (tabs vs spaces, max line length, etc) to software design decisions. The automatable aspects should be just automated and are not worth discussing. It's the bits that spill into design that are worth being opinionated about: how we distribute code across files directly maps to system modularization; how we name thing strongly influeces how we understand, communicate about, and modify the system, and so on.

More important than any presctriptive opinion is the ability to trace it back to some agreed upon principle. Opinions are made to be changed, and only by going back to common ground can we evolve our understanding and prevent dogmatism.

## Sources

- A Philosophy of Software Design
- A Philosophy of Software Design vs Clean Code
- Worse is Better
- The Grug Brained Developer
- Codin' dirty
- Unit Testing Principles
- Balancing Coupling in Software Design


## Design Principles and Assumptions

- The primary concern of the code writer needs to be the experience of future code readers.
  - Code needs to work, obviously, and we have means to ensure that (tests). But no one is in a better position that the one writing the code to make it easier to read.
  - The future operation of the code [may be more important](https://olano.dev/blog/code-is-run-more-than-read) than its reading, but that's the focus of design and other activties, it's not something that needs to be a primary focus in the structuring of the core.s
  - Maybe it's LLMs that end up reading the code, not humans, but the assumption here is that the more readable code is for humans, the more effective LLMs will be at processing. I won't sacrifize human readability for alleged LLM optimizations.
- For any piece of code, there should be enough context in the codebase for a reader to answer "what is this?" and "why is this here?".
- Accidental complexity needs to be removed, essential complexity needs to be managed.
  - there's local complexity when unrelated components are collocated
  - there's global complexity when significant knowledge travels long distances
  - complexity has lower impact if the complex component doesn't change frequently
- Modularity is the primary tool to manage complexity.
  - Modules are fractal: project, file, namespace, class, function can be reasoned about as modules with an interface and an implementation.
- Modules should be deep.
  - Shallow and pass-through modules are a smell.
  - Interface complexity costs more than implementation complexity.
  - Breaking modules apart typically increases interface complexity.
  - Reducing local complexity at the expense of global complexity is usually not a good trade off.

- Abstractions help with modularity, but each new abstraction (each new "concept") has a cognitive cost on code readers.
- Locality of Behavior often beats Separation of Concerns
- Code duplication is not necessarily a problem, knowledge duplication always is.

- The right style is the preexisting style in the project, if there's one. If there isn't, refer to this document.

## Project and directory structure

- The project codebase is a module, with its top level files (README, project.json, etc.) acting as the interface.

- The directory structure of a project should be intentional and meaninful.

- In most cases it's better to group components by their domain than by its technical features
  - eg put the employee enum next to the employee data structure, not next to other enums
      - [in reader terms: I will want to more frequently look at the employee together with the enum than to all project enums as a group]
  - e.g. put django forms declarations and fastapi request models next to their corresponding endpoints, as they constitute the same HTTP interface, instead of keeping separate forms/ and schemas/ packages.

## File contents

- File contents should be readable from top to bottom
  - this means that important (higher level) stuff should come first
      - primary exported things at the top, private helpers at the bottom
      - if a helper is used by one or two functions only, and for some reason shouldn't be inlined, it may make sense to put it next to those functions instead of at the bottom
  - When opening a file, I should be able to quickly answer what this and why here. this can be done in multiple ways:
      - the containing package and the filename. if this is `app/routes/employees.py` I know these will be API endpoints related to employees, no need extra commenting on the file.
      - If there's a single or primary struct or class at the top of the file, its docstring may provide the answers.
      - Otherwise a top-level docstring should provide the answers.
  - The order of things of the same importance within a file is an opportunity to convey meaning.
    - e.g. a create operation likely should come before a delete operation

## Module Interface

- Docstrings:
    - docstrings are part of the interface of a module.
    - docstrings should not reiterate the rest of the interface.
    - docstrings should not reveal implementation details.

TODO: good/bad

- Names:
  - when naming functions and methods, take into accout how they will be used.
    - method should assume the classname as its namespace
    - if the language supports it, function should preferably assume the classname as its namespace
      - names should be meaningful
        - short names are not necessarily bad
        - excessively long names are distracting

TODO: good/bad

## Module Implementation

- Comments:
    - comments are not a smell, they are a tool for communication
    - comments should complement the code, not repeat it (they provide the why, not the what)
    - TODO and FIXME comments are an extremely valuable tool to capture current understanding and intent.
      - it's not always convenient to prusue the best implementation, "the right thing", but its useful and cheap to document what we currently understand a better implementation would be, or what we perceive as weaknesses or opportunities for improvement. This helps future maintainers to get the context of previous work, decide if they are still relevant concerns and maybe execute the suggested improvements as part of other related work.
      - in evnironments where FIXME and TODO notes are considered a smell, a compromise can be to file a ticket and include the ticket number in the comment (but this should not replace the comment text!)

TODO: good/bad

- whitespace

- Constants:
  - Avoding magic numbers is not enough reason to introduce constants, and especially not globally exposed constants.
    - a constant declaration at the top of a function definition may suffice
    - if it needs to be accessed from multiple places in a module, put it at the top of the module
    - a constants module is a smell, and should only house things that genuinely need to to be used at multiple places in an app.
      - ask yourself if the modules couldn't be rearrange to remove the constant.
      - ask yourself it the constant shouldn't be a configurable app setting instead.

TODO: good/bad

- Type specifications
  - if you care about a thorough type system, don't use a dynamic language
  - if you use a dynamic language, the readability rule applies: don't sacrifice reader understanding for writer convenience
    - if you have to sprinkle your code with type check rule ignores, maybe you shouldn't enforce those checks
    - input and return types can improve readability a lot. awkward twists of inheritance chains and framework internals hurt it.

## Tests
