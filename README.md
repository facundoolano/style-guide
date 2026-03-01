# Facundo's highly controversial programming style guide

## Introduction

This is a brain dump of my current coding preferences. I try to adhere to these in personal projects and I try to subtly move team projects in this direction when it's not disruptive.

I occassionally find myself sending LLMs to read one of my blog posts before starting some task (e.g. for testing conventions), so rather than crafting weird `AGENTS.md` files I thought why not make a human readable guide to refer to. If it's human-readable enough surely Claude can handle it.

Why even bother with sytle if it looks like we won't be writing much code anymore? Maybe we won't, but it's likely that we still have to read a lot of it, so I'd rather have the LLMs follow my preferences. I also posit that better style makes better design, and better design leads to better quality LLM output. And if it does turn out that [we won't need](https://olano.dev/blog/dangerously-skip/) to even read code anymore, then indulge me on this, my own personal farewell to that side of the craft.

Why so much discussion about software design in a coding style guide? I think that style is a spectrum that goes from trivial formatting decisions (tabs vs spaces, max line length, etc) to software design. The automatable aspects should be just automated away and are not worth discussing. It's the bits that spill into design that are worth being opinionated about: how we distribute code across files directly maps to system modularization; how we name thing strongly influeces how we understand, communicate about, and modify the system, and so on.

More important than any presctriptive opinion is the ability to trace it back to some agreed upon principle. Opinions are made to be changed, and only by going back to common ground can we evolve our understanding and prevent dogmatism.

## Sources

- A Philosophy of Software Design
- A Philosophy of Software Design vs Clean Code
- Worse is Better
- The Grug Brained Developer
- Codin' dirty
- Unit Testing Principles
- Balancing Coupling in Software Design

FIXME add links

## Design Principles and Assumptions

- The primary concern of the code writer needs to be the experience of future code readers.
  - Code needs to work, obviously, and we have means to ensure that (tests). But no one is in a better position that the one writing the code to make it easier to read.
  - The operation of the code [may be more important](https://olano.dev/blog/code-is-run-more-than-read) than its reading, but that rarely needs to be a primary concern when deciding on internal coding style details.
  - Even if the LLMs also need to read the code, the assumption is that the more readable code is for humans, the more effective LLMs will be at processing it. I won't sacrifize human readability for alleged LLM optimizations.
- For any piece of code, there should be enough context in the codebase for a reader to answer "what is this?" and "why is this here?".
- Accidental complexity needs to be removed, essential complexity needs to be managed.
  - there's local complexity when unrelated components are collocated
  - there's global complexity when significant knowledge travels long distances
  - complexity has lower impact if the complex component doesn't change frequently
- Modularity is the primary tool to manage complexity.
  - Modules are fractal: project, file, namespace, class, function can be reasoned about as modules with interface and implementation.
- Modules should be deep.
  - Interface complexity costs more than implementation complexity.
  - Shallow and pass-through modules are a red flag.
  - Breaking modules apart typically increases interface complexity.
  - Reducing local complexity at the expense of global complexity is usually not a good trade off.

- Abstractions help with modularity, but each new abstraction (each new "concept") increases cognitive load on code readers.
- Locality of Behavior often trumps Separation of Concerns
- Code duplication is not necessarily a problem, knowledge duplication probably is.

- The right style is the preexisting style in the project, if there's one. If there isn't, refer to this document.

## Project and directory structure

- The project codebase is a module, with its top level files (README, project.json, etc.) acting as the interface.

- The internal directory structure of a project should be deliberate and meaningful.

- In most cases it's better to group components by domain relevance than by its technical attributes
  - e.g. to put a form class close to the endpoint where its used rather than close to other unrelated form classes
  - e.g. put the employee type enum next to the employee demol, not next to other project enums

- A README should at the very least answer: what is this and why is this necessary.
- A README should ideally also answer:
  - how do I build it
  - how do I run it
  - how do I test it
  - how do I deploy it
- The preferred answers to those questions are: `make build`, `make run`, `make test` and `make deploy`
  - Makefiles are preferable to literal instructions, language-specific build tool commands, ad hoc scripts.

## File contents

- File contents should be readable from top to bottom.
  - This means that important (higher level) stuff should come first
      - Put primary exported things at the top, internal helpers at the bottom
      - If a helper is used by one or two functions only, and for some reason shouldn't be inlined into the function, put it next to those functions instead of at the bottom of the file.
  - When opening a file, I should be able to quickly answer what is this and why its here. This can be achieved in many ways:
      - Inferred from the containing package and filename: if this is `app/routes/employees.py` I know these will be API endpoints related to employees, no need for comments.
      - If there's a single or primary struct or class at the top of the file, its docstring may suffice to provide explain the module's purpose.
      - Otherwise a top-level docstring should supply those answers.
  - The ordering of things of equal importance within a file is another opportunity to convey meaning.
    - e.g. implement the create operation first, the delete operation later

## Module Interface

- Docstrings:
    - Docstrings are part of the interface of a module.
    - Docstrings should not reiterate the rest of the interface but complement it.
      - If arguments are self-evident, don't list them.
    - Docstrings should not reveal implementation details.

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

- Don't extract short, single use helper functions.
  - a short code block leaded by a clarifying comment does a better job.

- Use whitespace deliberately, for instance to separate blocks of code within a large function.

- Constants:
  - Avoding magic numbers is not enough reason to introduce constants, and especially not globally shared constants.
    - A constant declaration at the top of the function that needs it may suffice
    - If it needs to be accessed from multiple places within a module, put it at the top of that module.
    - A constants module is a red flag, and should only house things that genuinely need to to be used at multiple places in an app and are not expected to change.
      - Ask yourself if the modules couldn't be rearrange to remove the constant.
      - Ask yourself it the constant shouldn't be a configurable app setting instead.

TODO: good/bad

- Type specifications
  - If you care about fully fledged type spefications, maybe you shouldn't use a dynamic language.
  - If you use a dynamic language, the readability rule applies: don't sacrifice reader understanding for writer convenience (type linting and autocomplete)
    - If you have to sprinkle your codebase with cheker rule ignores, maybe you shouldn't enforce those checks so strictly.
    - Input and return type specs improve readability and can reduce the need of clarifying docstring.
    - Awkward twists of inheritance chains and framework internals to satisfy the checker hurt readability and thuse are worse than no type specs at all.

## Tests

- TODO https://olano.dev/blog/unit-testing-principles/
- TODO https://olano.dev/blog/what-i-think-i-know-about-testing/
