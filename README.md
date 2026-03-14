# Facundo's programming style guide

This is a brain dump of my current coding preferences. I try to adhere to these in personal projects and I try to subtly move team projects in this direction (when it's not disruptive).

At the bottom is a section with the source material where I got most of my ideas from, which I quoted and paraphrased liberally to support my specific style choices.

This document is a work in progress, so far focused on capturing anything that comes to mind. It will likely require better organization to be used effectively.

## Introduction

I use LLMs a lot for coding these days, as most devs I know, but still feel a bit sick about crafting elaborate prompts and weird `AGENTS.md` files, as if I could somehow imbue these tools with personality or good sense. I've occasionally found myself sending Claude to read one of my blog posts before starting a task (e.g. for testing conventions or to provide context about a project), so I thought why not focus on documenting more of my habits and preferences, for a human audience, for my own sake? If it's human-readable enough then surely Claude should be able to handle it.

Why even bother with sytle if it looks like we won't be writing much code anymore? Maybe we won't, but at least for now we still have to read a lot of it, so I'd rather have LLMs produce it according to my preferences. I also posit that better style makes better design, and better design leads to better software, LLM or not. And, if it does turn out that [we won't need](https://olano.dev/blog/dangerously-skip/) to even read the code anymore, then indulge me on this, my own personal farewell to that side of the craft.

You'll notice that much of this document is spent on software design. I think that programming style is a spectrum that goes from trivial formatting choices (tabs vs spaces, max line length, etc) to software design, even architecture. The automatable aspects should be just automated away and need no discussion. It's the bits that spill into design that are worth being opinionated about: how we distribute code across files directly maps to system modularization; how we name things strongly influeces how we understand, communicate about, and modify the system; and so on.

More important than any presctriptive opinion is the ability to trace it back to some agreed upon principle. Opinions are made to be changed, and only by going back to common ground can we evolve our understanding and prevent dogmatism.

## Design principles and assumptions

1. The right style is the preexisting, agreed-upon style of the project, if there's one. If there isn't, refer to this document.
1. Code is a liability, not an asset.
    - Other things being equal (readability, complexity, economic incentives), the less code the better.
1. The primary concern of the code writer needs to be the experience of future code readers.
    - Never sacrifice reader understanding for writer convenience.
1. For any piece of code, there should be enough context in the codebase for a reader to answer "what is this?" and "why is it here?".
      - If these questions are hard to answer it could mean that: the names should be improved, docstrings or comments are missing, the component should be absorbed by another one, or it shouldn't exist at all.
1. Simplicity is the most important consideration in a design[WiB][APoSD].
    - A system is simple when its design is easy to understand and change.
    - Accidental complexity needs to be minimized, essential complexity needs to be managed.
    - Essential complexity [may be removed](https://olano.dev/blog/a-note-on-essential-complexity/) by redefining the problem.
    - Worse may be better[WiB][grug].
1. Modularity is the primary tool to manage complexity.
    - Modules are fractal: project, file, namespace, class, function can be reasoned about as modules with interface and implementation.
1. Modules should be deep <sup>[APoSD]</sup>.
    - Interface complexity costs more than implementation complexity.
    - Reducing local complexity at the expense of global complexity is usually a bade trade off.
    - Breaking modules apart frequently increases global complexity by adding more interfaces and separating things that depend on each other.
    - Shallow and pass-through modules are a red flag.
1. [Locality of Behavior](https://htmx.org/essays/locality-of-behaviour/) trumps Separation of Concerns <sup>[grug]</sup>
   - Things that need to be understood together should be closer together.
1. Abstractions help with modularity, but each new abstraction (each new "concept") increases cognitive load on code readers [CD].

## Project and directory structure

- The project codebase is a module, with its top level files (README, project.json, etc.) acting as the interface.

- The internal directory structure of a project should be deliberate and meaningful.
  - Things that need to be frequently read together should be spacially close together.
  - In most cases it's better to group components by domain relevance than by its technical attributes
    - e.g. to put a form class close to the endpoint where its used rather than close to other unrelated form classes
    - e.g. put the employee type enum next to the employee model, not next to other project enums

- A README should at the very least answer: what is this project and why is this necessary.
- A README should ideally also answer:
  - how do I build it
  - how do I run it
  - how do I test it
  - how do I deploy it
- The preferred answers to those questions are: `make build`, `make run`, `make test` and `make deploy`
  - Makefiles are preferable to literal instructions, language-specific build tool commands, ad hoc scripts.

## File contents

- File contents should be readable from top to bottom. This means that important (higher level) stuff should come first
    - Put primary exported things at the top, internal helpers at the bottom.
    - If a helper is used by one or two functions only, and for some reason shouldn't be inlined into the function, put it next to those functions instead of at the bottom of the file.
- When opening a file, I should be able to quickly answer "what is this?" and "why is it here?". This can be achieved in several ways:
    - Inferred from the containing package and filename: if this is `app/routes/employees.py` I know these will be API endpoints related to employees, no need for comments.
    - If there's a single or primary struct or class at the top of the file, its docstring may suffice to provide explain the module's purpose.
    - Otherwise a top-level docstring should supply those answers.
- The ordering of things of equal importance within a file is another opportunity to convey meaning
    - e.g. implement the create operation before the delete operation.

## Interface

### Docstrings (i.e. public interface comments)
- Docstrings are part of the interface of a module.
- Docstrings should not reiterate the rest of the interface but complement it.
  - If arguments are self-evident, don't list them.
- Docstrings should not reveal implementation details.
- Docstrings are the best means to capture domain knowledge of the project.

  TODO: good/bad example

### Names
- when naming functions and methods, consider how they will called.
    - Methods should assume the class/instance name as its namespace, i.e. `employee.create` not `employee.create_employee`.
    - Similarly, if the language supports it, functions should preferably assume the module name as its namespace.
- Names should be meaningful and chosen deliberately.
    - Short names are not necessarily bad.
    - Excessively long names are distracting.
    - Natural names allow to reason by analogy and synthetic names prevent ambiguity, choose accordingly [EoC].

## Implementation

- Code duplication is not necessarily a problem, knowledge duplication probably is.
  - Removing duplicated code is not enough reason to introduce an abstraction.
  - Encapsulating knowledge (aka hiding information) may be a good reason to introduce an abstraction.

- Don't extract short, single use helper functions [APoSD][CD].
  - A short code block leaded by a clarifying comment does a better job.
- Only extract single use helper functions if they can be understood on their own and by extracting them the readability of the calling function improves.
  - if understanding the _how_ of the helper is as relevant to the caller function as understanding the _what_, keep it inline.
  - The same rationale applies to extracting other types of modules (e.g. moving functions to separate files).

- Use whitespace deliberately, for instance to separate blocks of code within a large function  [APoSD].

### Comments
- Comments are not a smell, they are an aid for communication.
- Comments should complement the code, not repeat it (they should capture implementor intent---the why, not the what)
- TODO and FIXME comments are an extremely valuable tool to capture current understanding and intent.
  - It's not always convenient to prusue the best implementation, "the right thing", but its useful and cheap to document what we currently understand a better implementation would be, or what we perceive as weaknesses or opportunities for improvement. This helps future maintainers to get the context of previous work, decide if they are still relevant concerns and maybe execute the suggested improvements as part of other related work.
  - In evnironments where FIXME and TODO notes are considered a smell, a compromise can be to file a ticket and include the ticket number in the comment (but this should not replace the comment text!)

### Constants
- The presence of magic numbers and other literal values is not enough reason to externalize constants, especially not globally shared constants.
    - A constant declaration at the top of the function implementation that uses it may suffice
    - If it needs to be accessed from multiple places within a module, put it at the top of that module.
- A constants module is a red flag, and should only house things that genuinely need to to be used at multiple places in an app and are not expected to change.
    - Ask yourself if the modules couldn't be rearrange to remove the constant.
    - Ask yourself it the constant shouldn't be a configurable app setting instead.

### Type specificications
- Don't sacrifice reader understanding for writer convenience (type linting and autocomplete)
    - Input and return type specs improve readability and can reduce the need of clarifying docstring.
    - Awkward twists of inheritance chains and framework internals to satisfy the checker hurt readability and thuse are worse than no type specs at all.
    - Prefer to loosen checker strictness to sprinkling the codebase with checker rule ignores

## Tests

- Prefer integration tests to unit tests[CD][grug].
- There should be one integration test for each meaningful business/domain rule in the project [UTP].
  - The test name and its docstring should capture that domain knowledge, so one can ignore the implementation and get an aproximated use case specification:
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
- Unit tests should be left for pieces of code that need to be exercised in all of its variations, are complicated to setup, or would be too distracting if exercised as integration tests: complex algorithms, state machine transitions, etc.

- Not all code needs to have a test [UTP].
  - Trivial code should not be tested.
  - Layers covered by integration tests should not be unit tested.
  - External dependencies should not be tested, except as part of overarching integration tests.
- Code coverage should only be used to highlight detect untested areas, not as a target or indicator of quality [UTP].
- Test code shouldn't have conditionals (`if`). Split into separate tests.

- The system needs to be manually tested by a human, ideally in a production-like environment. No amount of automated test is an excuse to skip [some form](https://olano.dev/blog/verified-in-production/) of production verification.
- e2e tests are a nice to have in apps are stable and/or there's high risk in broken integrations going unnoticed
  - they take time to implement and brittle, they are no exuse from postponing manual e2e testing
  - even if they are passing, occassional e2e testing are still necessary

### Mocks
- Mocks of intermediate layers of the code are a red flag: they make the tests brittle, coupled to implementation details, with low resistance to refactoring, and low protection against regressions[UTP].
- Only mock the system's observable behavior, i.e. its inputs and outputs.
  - Leverage mock libraries for this purpose, e.g. responses for requests, respx for httpx.
    - i.e. a refactor from `requests.request(url, method='GET')` to `requests.get(url)` should not break a test.
    - if there is no such library, try to test at the lowest possible layer to preserve resistance to refactoring.

### Database
- Don't mock the database.
- Use the same db setup as in the production application.
- Start each test with an empty database (unless that becomes unbearably slow).
- Prefer to build the test data through the public interface of the app, instead of taking shortcuts.
  - E.g. in an HTTP API use the API to create the data, don't update the databse directly.

### Test helpers

- Don't use imported constants and enums in the tests, those are internals. The literals are the system observable behavior, use that.
- When the payloads are short, it's better to inline and repeat them in each test for readability.
- When payloads are big enough to distract from the purpose of the test, extract them to a helper at the bottom of the file.
- Helpers should be extracted to a separate module only if they are generic enough to be relevant to multiple test scenarios.
  - If the helper is needed in a single module, keep it there.
  - Don't overcomplicate the helper to satisfy multiple testing scenarios; keep separate versions on the corresponding modules.
- Craft the helper function interface to improve the readability of test
  - e.g. use sane defaults arguments and override for specific test cases:
    - `create_user()` is good if I don't care about the specific attributes,
    - `create_user(country='ar')` and `create_user(country='br')` is better if the test exercises a business rule around the user country
- Don't use a helper that includes the operation being tested.
  - e.g. don't rely on `create_user` for the user creation tests.
- Only use opaque helpers (e.g. pytest fixtures) for inftrastructure that doesn't need to be introspected, like test clients, db sessions, and request library mocks. Don't use them to build payloads.
## Sources

[NSB] [No silver bullet](https://worrydream.com/refs/Brooks_1986_-_No_Silver_Bullet.pdf)
  - Essential complexity is that inherent to the problem being solved.
  - Accidental complexity is that incurred, necessarily or not, to implement a concrete solution of the problem.
  - The most radical possible solution for constructing software is not to construct it at all.

[TB] [Programming as Theory Building](https://pages.cs.wisc.edu/~remzi/Naur.pdf)
  - The product of software building is not source code but a certain insight, a mental model (a theory), that enables programmers to understand, modfiy, explain, and answer questions about the system.
  - The system dies if no one possesses that mental model anymore.

[APoSD] [A Philosophy of Software Design](https://github.com/johnousterhout/aposd-vs-clean-code/blob/main/README.md)
  - Complexity is anything related to the structure of a software system that makes it hard to understand and modify the system.
      - Symptoms: change amplification, cognitive load, unknown unknowns.
      - Causes: dependencies an obscurity.
  - Reducing complexity is the most important element of software design.
      - The first approach is to eliminate complexity by making code simpler and more obvious.
      - The second approach is to encapsulate it, so that programmers can work on a system without being exposed to all of its complexity at once. This approach is called modular design.
  - Modules have two parts: interface and implementation.
      - The interface is everything that a developer working in a different module must know in order to use the given module. The interface describes _what_ the module does but not _how_ it does it.
        - The formal parts of an interface are specified explicitly in the code.
        - The informal parts of an interface can only be described using comments.
      - The implementation is the code that carries out the promises made by the interface. A developer should not need to understand the implementations of modules other than the one they are working in.
  - An abstraction is a simplified view of an entity, which omits unimportant details.
      - An abstraction that includes unimportant details increases cognitive load.
      - An abstraction that omits important details results in obscurity.
      - The key to designing abstractions is to understand what is important, and to look for designs that minimize the amount of information that is important.
  - It is more important for a module to have a simpler interface than a simple implementation.
      - The benefit provided by a module is its functionality. The cost (in terms of system complexity) is its interface.
      - The best modules are those whose interfaces are much simpler than their implementations. These are called "deep" modules.
      - A deep module is a good abstraction because only a small fraction of its internal complexity is visible to its users.
     - Shallow modules are a red flag.
     - Pass-through methods are a red flag.
     - Information hiding can often be improved by making a class slightly larger.
  - When deciding whether to combine or separate, the goal is to reduce the complexity of the system as a whole and improve its modularity.
    - Subdivision usually results in more interfaces, and every new interface adds complexity.
    - If components are truly independent, then separation is good.
    - Bringing pieces of code together is most beneficial if they are closely related: they share information, they are used together, they overlap conceptually, it's hard to understand one without looking at the other.
    - Length by itself is rarely a good reason for splitting up a method. You shouldn't break up a method unless it makes the overall system simpler.
      - Large methods are fine if they have a simple signature and are easy to read: these methods are deep.
  - The exceptions thrown by a class are part of its interface.
    - Classes with lots of exceptions (declared or not) have complex interfaces.
    - The best way to reduce the complexity damage caused by exception handling is to reduce the number of places where exceptions have to be handled.
  - Good comments reduce complexity.
     - Comments should capture information that was in the mind of the designer but couldn't be represented in the code
     - Comments reduce cognitive load by providing information needed to change the code and by making it easier to ignore what is irrelevant.
     - Comments can remove unknown unknowns, clarify dependencies, and fill in the gaps to eliminate obscurity.
  - Comments should describe things that are not obvious from the code.
     - Comments augment the code by providing information at a different level of detail.
     - Developers should be able to understand the abstraction provided by a module without reading any code other than its external declarations. This is done by supplementing declarations with comments.
     - If interface comments must also describe the implementation, then the class or method is shallow.
     - Implementation comments should help readers understand _what_ the code is doing (not _how_ it does it).
  - Good names reduce complexity.
    - Names should be precise and consistent. A vague name is a red flag.
    - If it's hard to pick a name for a component, it may not have a clean design.
  - Code should be obvious.
  - Things that matter should be emphasized and made more obvious; things don't matter should be hidden as much as possible.
    - What matters can be emphasized through prominence, repetition, and centrality.

[WiB] [Worse is Better](https://dreamsongs.com/RiseOfWorseIsBetter.html)
  - The design must be simple, both in implementation and interface. Simplicity is the most important consideration in a design.
  - The design must be correct in all observable aspects. It is slightly better to be simple than correct.
  - The design must cover as many important situations as is practical. Completeness can be sacrificed in favor of any other quality. In  fact, completeness must be sacrificed whenever simplicity is jeopardized.
  - The design must not be overly inconsistent. Consistency can be sacrificed for simplicity in some cases, but it is better to drop those parts of the design that deal with less common circumstances than to introduce either complexity or inconsistency.

[grug] [The Grug Brained Developer](https://grugbrain.dev/)
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

[CD] [Codin' dirty](https://htmx.org/essays/codin-dirty/)
   - (Some) big functions are good, actually
   - Prefer integration tests to unit tests
   - Keep your class/interface/concept count down

[UTP] [Unit Testing Principles](https://olano.dev/blog/unit-testing-principles/)
  - Some tests are valuable and contribute a lot to overall software quality. Others don’t. They raise false alarms, don’t help you catch regression errors, and are slow and difficult to maintain.
  - Coverage metrics are a good negative indicator (low coverage means you’re not testing enough) but a bad positive one (high coverage doesn’t guarantee good testing quality). Targeting a specific coverage number creates a perverse incentive that goes against the goal of unit testing.
  - Tests shouldn’t verify units of code. Rather, they should verify units of behavior, something that is meaningful for the problem domain and, ideally, something that a business person can recognize as useful.
  - Choose black-box testing over white-box testing by default.
      - If you can’t trace a test back to a business requirement, it’s an indication of the test’s brittleness. Either restructure or delete this test.
      - You need to make sure the test verifies the end result the system under test delivers: its observable behavior, not the steps it takes to do that.
  - The ubiquitous use of mocks produces tests that couple too tightly to implementation details.
      - The use of mocks is beneficial when verifying the communication pattern between your system and external applications.
      - Using mocks to verify communications between classes inside your system results in tests that couple to implementation details and therefore fall short of the resistance-to-refactoring metric.
  - Code can be either deep (complex or important) or wide (work with many collaborators), but not both.
    - Trivial code (low complexity/significance, few collaborators): this code shouldn’t be tested at all.
    - Domain model and algorithms (high complexity/significance, few collaborators): this code should be unit tested. The resulting unit tests are highly valuable and cheap.
    - Controllers (low complexity/significance, many collaborators): controllers should be tested as part of overarching integration tests.
    - Overcomplicated code (high complexity/significance, many collaborators): this code is hard to test, and as such it’s better to split it into domain/algorithms and controllers.

[BC] [Balancing Coupling in Software Design](https://olano.dev/blog/balancing-coupling/)
  - There's local complexity when unrelated components are collocated
  - There's global complexity when significant knowledge travels long distances
  - Complexity has lower impact if the complex component doesn't change frequently
  - Coupling should not be eliminated, it should be "balanced". There is balance when volatile components are modular and complex components don't change.

[EoC] Names chapter from [Elements of Clojure](https://elementsofclojure.com/manuscript/elements_of_clojure.pdf#page=8)
  - Names should be narrow and consistent.
      - A narrow name clearly excludes things it cannot represent.
      - A consistent name is easily understood by someone familiar with the surrounding code, the problem domain, and the broader language ecosystem.
  - The same name can have different senses depending on context.
    - Keeping contexts separate requires continuous effort by the reader, and failing to keep them separate creates subtle misunderstandings.
    - If we avoid separate contexts, our datatype can only be as narrow as its most general case.
    - The only way to be fully consistent is to have a one-to-one relationship between signs and senses. This
means that we must invent a sign for each sense, but also that readers must agree on their sense.
  - Most natural names havea rich, varied collection of senses. To avoid ambiguity we must use synthetic names, which have no intuitive sense in the context of our code.
  - Natural names allow every reader, novice or expert, to reason by analogy. Reasoning by analogy is a powerful tool, especially when our software models and interacts with the real world. Synthetic names defy analogies, and prevent novices from understanding even the basic intent behind your code. choose accordingly.
