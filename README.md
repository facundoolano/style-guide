# Facundo's highly controversial programming style guide

## Introduction

This is a brain dump of my current coding preferences. I try to adhere to these in personal projects and I try to subtly move team projects in this direction (when it's not disruptive).

I've occassionally found myself sending LLMs to read one of my blog posts before starting a task (e.g. for testing conventions), so rather than crafting weird `AGENTS.md` files I
thought why not make a human readable guide to refer them to. If it's human-readable enough surely Claude can handle it.

Why even bother with sytle if it looks like we won't be writing much code anymore? Maybe we won't, but at least for now we still have to read a lot of it, so I'd rather have the LLMs produce code that follows my preferences. I also posit that better style makes better design, and better design leads to better software, LLM or not. And, if it does turn out that [we won't need](https://olano.dev/blog/dangerously-skip/) to even read the code anymore, then indulge me on this, my own personal farewell to that side of the craft.

Why so much discussion about software design in a coding style guide? I think that style is a spectrum that goes from trivial formatting decisions (tabs vs spaces, max line length, etc) to software design. The automatable aspects should be just automated away and are not worth discussing. It's the bits that spill into design that are worth being opinionated about: how we distribute code across files directly maps to system modularization; how we name thing strongly influeces how we understand, communicate about, and modify the system, and so on.

More important than any presctriptive opinion is the ability to trace it back to some agreed upon principle. Opinions are made to be changed, and only by going back to common ground can we evolve our understanding and prevent dogmatism.

## Design Principles and Assumptions

- Code is a liability, not an asset. Other things being equal (readability, complexity, economic incentives), the less code the better.
- The right style is the preexisting style in the project, if there's one. If there isn't, refer to this document.
- The primary concern of the code writer needs to be the experience of future code readers.
  - Never sacrifice reader understanding for writer convenience.
  - For any piece of code, there should be enough context in the codebase for a reader to answer "what is this?" and "why is this here?".
    - If these questions are hard to answer it could mean that: the names should be improved, docstrings/comments are missing, the component should be absorbed by another component, or it shouldn't exist at all.
- Reducing complexity is the most important element of software design. [APoSD]
  - A system is simple when its design is easy to understand and change.
  - Essential complexity is inherent the problem being solved. Accidental complexity is that incurred, necessarily or not, to implement a concrete solution of the problem. [NSB]
  - Accidental complexity needs to be minimized, essential complexity needs to be managed.
  - Essential complexity [may be removed](https://olano.dev/blog/a-note-on-essential-complexity/) by redefining the problem.
- Modularity is the primary tool to manage complexity.
  - Modules are fractal: project, file, namespace, class, function can be reasoned about as modules with interface and implementation.
  - Modules should be deep <sup>[APoSD]</sup>.
    - Interface complexity costs more than implementation complexity.
    - Reducing local complexity at the expense of global complexity is usually not a good trade off.
    - Breaking modules apart typically increases global complexity by increasing the amount of interfaces.
    - Shallow and pass-through modules are a red flag.
- Abstractions help with modularity, but each new abstraction (each new "concept") increases cognitive load on code readers [CD].
- Worse may be better [WiB]
    - The design must be correct in all observable aspects. It is slightly better to be simple than correct.
    - The design must cover as many important situations as is practical. Completeness can be sacrificed in favor of any other quality. In  fact, completeness must be sacrificed whenever simplicity is jeopardized.
    - The design must not be overly inconsistent. Consistency can be sacrificed for simplicity in some cases, but it is better to drop those parts of the design that deal with less common circumstances than to introduce either complexity or inconsistency.
- [Locality of Behavior](https://htmx.org/essays/locality-of-behaviour/) trumps Separation of Concerns <sup>[grug]</sup>
- Code duplication is not necessarily a problem, knowledge duplication probably is.

## Project and directory structure

- The project codebase is a module, with its top level files (README, project.json, etc.) acting as the interface.

- The internal directory structure of a project should be deliberate and meaningful.
  - Things that need to be frequently read together should be spacially close together.
  - In most cases it's better to group components by domain relevance than by its technical attributes
    - e.g. to put a form class close to the endpoint where its used rather than close to other unrelated form classes
    - e.g. put the employee type enum next to the employee model, not next to other project enums

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

- Docstrings (i.e. public interface comments):
    - Docstrings are part of the interface of a module.
    - Docstrings should not reiterate the rest of the interface but complement it.
      - If arguments are self-evident, don't list them.
    - Docstrings should not reveal implementation details.
    - Docstrings are the best means to capture domain knowledge of the project.

TODO: good/bad

- Names:
  - when naming functions and methods, take into accout how they will be used.
    - method should assume the classname as its namespace
      - if the language supports it, function should preferably assume the module name as its namespace. i.e. `employee.create` not `employee.create_employee`
  - names should be meaningful and chosen deliberately
    - short names are not necessarily bad
    - excessively long names are distracting
    - natural names allow to reason by analogy
    - synthetic names prevent ambiguity

## Module Implementation

- Comments:
    - comments are not a smell, they are a tool for communication
    - comments should complement the code, not repeat it (they should capture implementor intent---the why, not the what)
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
  - Don't sacrifice reader understanding for writer convenience (type linting and autocomplete)
    - Input and return type specs improve readability and can reduce the need of clarifying docstring.
    - Awkward twists of inheritance chains and framework internals to satisfy the checker hurt readability and thuse are worse than no type specs at all.
    - Prefer to loosen checker strictness to sprinkling the codebase with checker rule ignores

## Tests

- Prefer integration tests to unit tests.
- There should be one integration test for each meaningful business/domain rule in the project.
  - the test name and its docstring should reflect this (and not reiterate the test implementation), so one can ignore the implementation and get an aproximated use case specification:
  ```python
  def test_login_no_verified_fails(self, client):
        "Unverified users cannot log in."
        # ...
  def test_login_succeeds(self, client):
        "Users can log in after verifying their email."
        # ...
  def test_login_wrong_password(self, client):
        "Login fails with an error message when entering a wrong password."
        # ...
  ```
  - this also means that tests that aren't meaningful should be removed/refactored
- Unit tests should be left for pieces of code that need to be exercised in all of its variations, are complicated to setup, or would be too distracting if exercised as integration tests: complex algorithms, state machine transitions, etc.

- Not all code needs to have a test.
  - Trivial code should not be tested.
  - Layers covered by integration tests should not be unit tested.
  - External dependencies should not be tested, except as part of overarching integration tests.
- Code coverage should only be used to highlight detect untested areas, not as a target or indicator of quality.
- It's not ok to have conditional code in tests. Split into different tests.

### Mocks
- never mock intermediate layers of the code, only observable behavior, e.g. system inputs and outputs.
  - leverage mock libraries for this purpose, e.g. responses for requests, respx for httpx.
    - i.e. a refactor from `requests.request(url, method='GET')` to `requests.get(url)` should not break a test.
    - if there is no such library, try to test at the lowest possible layer to preserve resistance to refactoring.

### Database
- Don't mock the database.
- use the same db setup as in the production application.
- start each test with an empty database (unless that becomes unbearably slow).
- prefer to build the test data through the public interface of the app, as opposed to injecting necessary values directly in the database through other means.

### Test data and helper functions

- Don't use imported constants and enums in the tests, those are internals. The literals are the system observable behavior, use that.

- when the payloads are short, it's better to inline and repeat them in each test for readability.
- when payloads are big enough to distract from the purpose of the test, extract them to a helper at the bottom of the file.
- helpers should be extracted to a separate module only if they are generic enough to be relevant to multiple test scenarios.
  - if the helper is needed in a single module, keep it there
  - don't overcomplicate the helper to satisfy multiple testing scenarios; keep separate versions on the corresponding modules.

- take advantage of defaults to improve readability of the tests through the helpers.
  - `create_user()` is good if I don't care about the specific attributes
  - `create_user(country='ar')` and `create_user(country='br')` is better if the test exercises a business rule around the user country
- don't use helpers to test the operation being encapsulated.
  - don't use `create_user` when the test targets user creation.

- Only use opaque helpers (e.g. pytest fixtures) for inftrastructure that doesn't need to be introspected, like test clients, db sessions, and request library mocks. Don't use them to build payloads.
## Sources

TODO: finish inlining key takeaways
TODO: move take aways to their corresponding sections, with the reference codes.
TODO: add anchors

- [ ] [No silver bullet](https://worrydream.com/refs/Brooks_1986_-_No_Silver_Bullet.pdf)
- [ ] [A Philosophy of Software Design](https://github.com/johnousterhout/aposd-vs-clean-code/blob/main/README.md)
- [x] [Programming as Theory Building](https://pages.cs.wisc.edu/~remzi/Naur.pdf)
  - The product of software building is not source code but a certain insight, a mental model (a theory), that enables programmers to understand, modfiy, explain, and answer questions about the system.
  - The system dies if no one possesses that mental model anymore.
- [x] [Worse is Better](https://dreamsongs.com/RiseOfWorseIsBetter.html) [WiB]
- [x] [The Grug Brained Developer](https://grugbrain.dev/) [grug]
  - "No" is the best tool against complexity. Say it to unnecessary features, abstractions, and over-engineering.
  - When you can't say no, deliver 80% of the value with 20% of the code.
    - Project managers often forget requirements or move on — the 80/20 usually serves their real interests anyway.
  - Don't factor code too early. Wait for the system's shape to emerge.
  - Don't write tests before you understand the problem or domain. Write them after prototyping, when the code has proven itself.
  - Integration tests are the sweet spot: high-level enough to verify correctness, low-level enough to diagnose failures.
  - Mocking provides limited value. If you must mock, use only coarse-grained mocks.
  - Don't remove or drastically change code without understanding why it exists.
  - Respect working systems even when imperfect.
  - About microservices: introducing a network call between subsystems adds tremendous complexity without necessarily solving the underlying factoring problem.
  - Over-abstraction in type systems makes simple tasks unnecessarily difficult.
  - Don't minimize lines of code at the expense of readability.
  - Good APIs reduce cognitive load. Bad APIs force users to think about implementation details.
  - Design for usage, not implementation.
  - Place methods on the objects they operate on.
  - Common operations should not require unnecessary intermediate steps (e.g., don't force stream conversion just to filter a list).
  - Return types that match user expectations (a filtered list should return a list, not a stream).
- [x] [Codin' dirty](https://htmx.org/essays/codin-dirty/)
   - (Some) big functions are good, actually
   - Prefer integration tests to unit tests
   - Keep your class/interface/concept count down

- [x] [Unit Testing Principles](https://olano.dev/blog/unit-testing-principles/)
  - Some tests are valuable and contribute a lot to overall software quality. Others don’t. They raise false alarms, don’t help you catch regression errors, and are slow and difficult to maintain.
  - Tests shouldn’t verify units of code. Rather, they should verify units of behavior, something that is meaningful for the problem domain and, ideally, something that a business person can recognize as useful.
  - Coverage metrics are a good negative indicator (low coverage means you’re not testing enough) but a bad positive one (high coverage doesn’t guarantee good testing quality). Targeting a specific coverage number creates a perverse incentive that goes against the goal of unit testing.
  - The ubiquitous use of mocks produces tests that couple too tightly to the implementation.
  - The more the test is coupled to the implementation details of the system under test (SUT), the more false alarms it generates. You need to make sure the test verifies the end result the SUT delivers: its observable behavior, not the steps it takes to do that.
  - Choose black-box testing over white-box testing by default. If you can’t trace a test back to a business requirement, it’s an indication of the test’s brittleness. Either restructure or delete this test.
  - The use of mocks is beneficial when verifying the communication pattern between your system and external applications. Conversely, using mocks to verify communications between classes inside your system results in tests that couple to implementation details and therefore fall short of the resistance-to-refactoring metric.
  - Code can be either deep (complex or important) or wide (work with many collaborators), but not both.
    - Trivial code (low complexity/significance, few collaborators): this code shouldn’t be tested at all.
    - Domain model and algorithms (high complexity/significance, few collaborators): this code should be unit tested. The resulting unit tests are highly valuable and cheap.
    - Controllers (low complexity/significance, many collaborators): controllers should be tested as part of overarching integration tests.
    - Overcomplicated code (high complexity/significance, many collaborators): this code is hard to test, and as such it’s better to split it into domain/algorithms and controllers.

- [x] [Balancing Coupling in Software Design](https://olano.dev/blog/balancing-coupling/)
  - There's local complexity when unrelated components are collocated
  - There's global complexity when significant knowledge travels long distances
  - Complexity has lower impact if the complex component doesn't change frequently
  - Coupling should not be eliminated, it should be "balanced". There is balance when volatile components are modular and complex components don't change.
- [ ] 'Names' Chapter from [Elements of Clojure](https://elementsofclojure.com/manuscript/elements_of_clojure.pdf)
