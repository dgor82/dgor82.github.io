Last Update: 26/05/2024

# My Dev Style Guide

In my .NET / C# projects, I generally follow the set of approaches and paradigms described below, to guide the development process towards excellence and a maintainable codebase - thereby attempting to avoid the complexity trap that besets so many software projects.

# ToC

- [My Dev Style Guide](#my-dev-style-guide)
- [ToC](#toc)
- [I) Dev \& Team-Work Practices](#i-dev--team-work-practices)
  - [Test-Driven Development (TDD)](#test-driven-development-tdd)
    - [1. Unit Tests](#1-unit-tests)
    - [2. Integration Tests](#2-integration-tests)
    - [3. Acceptance Tests](#3-acceptance-tests)
    - [4. External Tests](#4-external-tests)
    - [Why no test-coverage tools?](#why-no-test-coverage-tools)
  - [Vertical Slicing](#vertical-slicing)
  - [Domain-Driven Design (DDD)](#domain-driven-design-ddd)
  - [Continuous Refactoring \& Simple Design](#continuous-refactoring--simple-design)
  - [Continuous Integration (CI)](#continuous-integration-ci)
    - [CI Workflow Summary](#ci-workflow-summary)
  - [Commit Messages](#commit-messages)
- [II) Coding Style](#ii-coding-style)
  - [SOLID Principles](#solid-principles)
  - [Design by Contract](#design-by-contract)
  - [Mixed Paradigm (OO ∩ FP)](#mixed-paradigm-oo--fp)
    - [Use of Monadic Wrappers](#use-of-monadic-wrappers)
      - [1) Instead of Nullable Reference Types: `Option<T>`](#1-instead-of-nullable-reference-types-optiont)
      - [2) For Failing Operations: `Attempt<T>`](#2-for-failing-operations-attemptt)
      - [3) For Validation Error Collections: `Validation<T>`](#3-for-validation-error-collections-validationt)
    - [Monadic Composition](#monadic-composition)

# I) Dev & Team-Work Practices

Inspirations/sources:
- [The Agile Manifesto](https://agilemanifesto.org)
    - As fully explained in: [Clean Agile](https://www.goodreads.com/book/show/45280021-clean-agile)
- [Manifesto for Software Craftsmanship](http://manifesto.softwarecraftsmanship.org)
    - As fully explained in: [The Software Craftsman](https://www.goodreads.com/book/show/23215733-software-craftsman-the)
- [Extreme Programming (XP)](https://www.goodreads.com/book/show/67833.Extreme_Programming_Explained)
- [The Pragmatic Programmer](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)
- [Domain-Driven Design](https://www.goodreads.com/book/show/179133.Domain_Driven_Design)
- [Effective Software Testing: A developer's guide](https://www.goodreads.com/book/show/59796908-effective-software-testing)
- [Trunk Based Development](https://trunkbaseddevelopment.com/5-min-overview/)
- [Mocks Aren't Stubs (Martin Fowler)](https://martinfowler.com/articles/mocksArentStubs.html)

## Test-Driven Development (TDD)

My overriding goal is fearless continuous deployment and fearless continuous refactoring.
  
To achieve this goal, I follow TDD, which involves writing tests and production code hand-in-hand, ensuring every new feature starts with a failing test. This leads to test coverage by default, simpler design, and more modular, decoupled code. 

I have discovered that, with true TDD, the test code (a first-class citizen in terms of coding standards) becomes the *full* specification, *executable* documentation and even a fairly good representation of *the user*!

The following sections describe how I interpret and apply the 'test pyramid' - with the different kinds of tests in it, and the (sometimes soft) boundaries between them. I end with a few words on why I don't use test coverage tools.

### 1. Unit Tests

I try to decouple unit tests from implementation details: the ideal 'unit' under test thus is NOT a single method/function but rather a cluster of closely interrelated methods (recursively including all or most internal dependencies) that, taken together, represent some logical 'chunk' of end-to-end functionality (henceforth just 'unit'). I have given up artificially distinguishing between unit-tests and functional tests (doing so drove me to units that are too small).

Any unit will typically be represented by a single test class. The various test cases in that class aim to thoroughly test all the boundary cases - how thoroughly depends on the unit's complexity and importance. The various forms of code coverage criteria can be sources of inspiration/ideas:

- line/statement coverage vs.
- branch coverage vs.
- condition + branch coverage (a good default?) vs.
- path coverage (beware of the combinatorial explosion!) vs. 
- [MC/DC](https://en.wikipedia.org/wiki/Modified_condition/decision_coverage) (thorough but doable - the smart/optimised choice for high-stakes / mission-critical code)

While the emphasis of my unit tests is on testing interfaces / return values, I feel it is justified to selectively also use verifications of the behaviour of mocks of important external dependencies with visible side effects. In this context, one of my [tech mentors](tech_advisory_board.html), [Paul Butcher](https://www.linkedin.com/in/paulbutcher/), likes to jokingly bring up the  "launching of ICBMs" as an example for a side-effect, for which the verification of behaviour might be worthwhile.

My unit tests make use of a modified D.I. services container that inherits from the app's main container but replaces external dependencies (e.g. repositories with database access) with their mocks. Overall it would seem then, that I intuitively have subscribed to the classic (rather than mockist) [school of thought](https://martinfowler.com/articles/mocksArentStubs.html#ClassicalAndMockistTesting). I feel this makes my unit tests more realistic and less brittle in the face of continuous refactoring, thus better supporting the two overriding goals stated at the outset. 

### 2. Integration Tests

These are few, end-to-end tests covering the most common use-cases to establish the correct interplay of my business logic with its external dependencies like the database or external APIs. This is more about testing data flow and orchestration than about covering a large percentage of logic / code branches / edge cases / exotic flows.

For my integration tests, I rely on the main app's D.I. services container for resolution of all dependencies.
  
### 3. Acceptance Tests

The main purpose of these tests is to ensure the application meets end-user requirements, simulating real-world usage to validate the complete functionality and integration of all features. They run against the full system, including its external dependencies, without any mocking. 

While important, these tests would be far lower in number (in line with the typical test pyramid) and have been lower priority for me. In some cases, like projects based on a Telegram.Bot, they have been manual. For future automation of acceptance tests for GUI-based apps, high-level scenarios should be described in a language understandable by both technical and non-technical stakeholders - I will consider introducing [SpecFlow](https://specflow.org).

### 4. External Tests

These exist in their own dedicated test module which doesn't have any visibility of any of the actual app. These are therefore isolated unit tests to learn/document/assert the behaviour of external libraries and frameworks, which fulfil any of the below criteria: 

- They are not-obvious and so benefit from executable documentation
- I had to 'learn' to use them (--> learning via unit testing)
- The library has only medium or even low popularity (thus, asserting its functionality that I rely on across my code is important, in case future updates break it - which is far less likely for mega-popular libraries)

### Why no test-coverage tools?

Especially on a greenfield project, where the developer(s) are 'test-infected' (think 'test first', follow TDD...) they don't add much value but they do create considerable overhead and distraction.
  
The crux is that '100%' test coverage is a totally meaningless measurement target!

1) When sticking to a coarse measurement like 'statement coverage', then the '100%' is easy to attain but doesn't actually lead to sufficient coverage of branches / conditions / decisions! This is the case, for instance, with JetBrain's dotCover.   
  
Consider this C# statement:
```
var userId = telegramInputMessage.From?.Id 
              ?? throw new ArgumentNullException(nameof(telegramInputMessage),
                  "From.Id in the input message must not be null");        
```

A happy-path test that never touches the `throw` branch of this statement still causes dotCover to report a 100% coverage. This is misleading and perversely incentivises use of a less terse `if` statement syntax (which works as expected). Tail wagging the dog!

2) When using real coverage (i.e. path-coverage), 100% is unattainable anyway due to the combinatorial explosion (for any code with a fair amount of complexity). 

3) One thus ends up obsessing about and fiddling with:
- code coverage methodologies
- coverage statistics (invariably aiming for the meaningless 100% for lack of any other sensible target)
- coverage filter settings
- etc.
  
... time and attention that would be better spent thinking deeply about important boundary cases!


## Vertical Slicing

By adopting Vertical Slicing, I ensure any development efforts are focused on delivering small, incremental pieces of functionality that span all the architectural layers from the UI to the backend and all components and platforms. This facilitates quicker iterations and feedback loops, directly aligning development with user feedback and business requirements, reducing the risk of misaligned features.

## Domain-Driven Design (DDD)
(replaces 'Metaphor' from XP)

DDD emphasises a deep understanding of the domain to inform our software design. The central concept here is developing a 'ubiquitous language' which domain experts and coders share. This language is reflected in the naming and choice of abstractions in the code, making it partially comprehensible to non-technical stakeholders (eventually with a goal of moving towards an internal Domain-Specific Language (DSL)). DDD ensures our code stays closely aligned with business needs and the real-world subtleties of the domain. It also facilitates a common language and thus clearer communication across all stakeholders. 

This also implies that the code base becomes the best kind of project documentation (only devOps & high-level architecture may require  documentation separate from code). I totally avoid explanatory comments inside the code-base except for cases where non-obvious or unusual externalities are involved (e.g. config-related code). In most cases when I catch myself feeling the need to add a comment, it turns out there was an underlying naming or design issue. 

## Continuous Refactoring & Simple Design

As described in detail by Uncle Bob, Martin Fowler, Kent Beck etc.

## Continuous Integration (CI)

For my solo or small-team projects I like to set up the following CI workflow, supported by local shell scripts for productivity / automation.

### CI Workflow Summary

Inspired and informed, among others, by [Trunk Based Development](https://trunkbaseddevelopment.com/5-min-overview/)

1. The main branch is protected from direct mergers, it only accepts mergers through PRs.

2. Developers work locally on short-lived feature branches.

3. When ready for merging...  
a) They run build & test locally for the entire solution for every `Debug_*` configuration.  
b) On pass, they push their working branch to GitHub which triggers automated PR-creation and merger into main.  
c) This in turn should trigger a build, test & deploy run for the Release configuration of each **project** (!) marked for deployment (i.e. not solution-wide).  
d) In case of conflicts, these need to be resolved manually, followed by a renewed PR merger attempt.

4. This means, the PRs are merged into main **before** reviews: reviews shall be conducted post-merger and a corresponding GitHub project-task is generated automatically. 

This approach to CI supports a truly _continuous_ integration without delays from waiting for manual PR reviews.
Full test-coverage / TDD should ensure well-enough that no breaking changes are introduced into main. Usually, the entire workflow should be automated with project-specific shell scripts designed to run on dev's machines.

I have created this [Custom GPT](https://chatgpt.com/g/g-0KRoUTOLM-generate-short-pr-feature-branch-title) to generate meaningful names for feature branches / PR titles to support the above workflow.

## Commit Messages

I use the A.I. plugin of JetBrains Rider to auto-generate my commit messages, with the following default prompt (in the A.I. Plugin's settings) having proven to generate accurate and useful summaries on the right level of detail. Feel free to copy and modify to your delight!

> Avoid overly verbose descriptions or unnecessary details.
Start with a short sentence in imperative form, no more than 50 characters long.
Then leave an empty line and optionally continue with more details in bullet point form.
Only add details if the changes made are not very minor or cosmetic e.g. a single rename or clean-up. In that case, no further bullet points / details are needed.
Just in case details are needed, for more substantial commits, follow these guidelines:
Add between one and three bullet-point style entries, one for each 'cluster' of change you identified.
These should not be complete, grammatically correct sentences!
Each bullet point starts with a new line.
Avoid adding bullet points about obvious changes e.g. the fact that an interface was updated to include newly added functionality that was already covered in another bullet point.
Also, do not try to infer the intention behind any changes, or the possible benefits associated with them. 
For each cluster of change, summarise the actual changes to a level of abstraction so that you don't actually need to mention specific names of files / classes / methods that were changed (since I can see that anyway in the actual commit's change history). 


# II) Coding Style

Inspirations from
- [Clean Code](https://www.goodreads.com/book/show/3735293-clean-code)
- Clean Architecture
    - [Summary](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
    - [Full Book](https://www.goodreads.com/book/show/18043011-clean-architecture)
- [The Pragmatic Programmer](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)
- [Functional Programming in C#](https://www.goodreads.com/book/show/31550964-functional-programming-in-c)
- [Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#](https://www.goodreads.com/book/show/34921689-domain-modeling-made-functional)
- [FP vs. OO (by Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2018/04/13/FPvsOO.html)
- [John Carmack on Functional Programming in C++](http://sevangelatos.com/john-carmack-on/)
- [Railway Oriented Programming (by Scott Wlaschin)](https://fsharpforfunandprofit.com/rop/)

## SOLID Principles

Adherence to the SOLID Principles, esp. where OO design is at play.

- S = Single Responsibility Principle
- O = Open/Closed Principle
- L = Liskov Substitution Principle
- I = Interface Segregation Principle
- D = Dependency Inversion Principle

## Design by Contract

DbC, inspired by Bertrand Meyer, the inventor of the Eiffel language, ensures that software components interact based on clearly defined specifications. This formal agreement on expected inputs, outputs, and side effects between components leads to more reliable and robust system behaviour, facilitating easier debugging and validation of software correctness. This complements the TDD approach described further above. I believe in a pragmatic application of DbC by limiting its use to the outer edges of each component, i.e. where it interfaces with other components or third-party libraries. 

## Mixed Paradigm (OO ∩ FP)

I follow a mixed paradigm approach to coding, following the mixed-paradigm nature of C# itself. This means blending object-oriented (OO) principles for system organisation at the larger scale (SOLID etc.) with a functional programming style (FP) for most of the actual code construction. This means avoiding imperative code, mutability and stateful operations whenever feasible and carefully demarcating the group of classes that require statefulness. 

This approach reduces side effects, making my code more predictable, easier to test and more suitable for concurrency and parallelism. To achieve this, I draw on Lambdas, LINQ, pattern matching, switch-expressions etc. (all natively supported by C#). Despite my mantra for a single language across the entire system, I'd consider mixing a F# project into the .NET solution for backend code in experimental or pet projects, and in commercial projects when team skills allow for it. After all, it is one of the small luxuries of the .NET ecosystem that C# and F# both run on the Common Language Runtime (CLR) and can easily be mixed in a single solution without InterOp-related friction.

Previously I was considering the cautious introduction of [Language-Ext](https://github.com/louthy/language-ext) into commercial projects to move C# even closer to FP. After further deliberation I have distanced myself from that idea in favour of 'paradigmatic integrity', i.e. to avoid:  
a) going too much against the grain of C#  
b) extreme dependency on a heavy-weight but only medium-popular library  
c) reduced readability of my code for most C# devs out there

Instead, I have created my own library of light-weight monadic wrappers, i.e. the bare minimum to enable [Railway Oriented Programming](https://fsharpforfunandprofit.com/rop/) (ROP) in C#.

### Use of Monadic Wrappers 

#### 1) Instead of Nullable Reference Types: `Option<T>`

`From [8SPHE]`

I apply three simple rules:
  
1. Enable nullable types consistently via `<Nullable>enable</Nullable>` in Directory.Build.props saved in solution root.
2. Never actually make any types under my control nullable (with the exception of using them for the fields in my monadic wrappers).
3. Instead, use `Option<T>` for any optional T.

#### 2) For Failing Operations: `Attempt<T>`

In my default setup, I also have `Attempt<T>` to encapsulate the outcome of an operation that might result in an error message or throw an exception. I steer clear of the name `Try<T>` (idiomatic in the functional world) due to its clash with the specific semantics of the 'Try' prefix for methods in C# (i.e. returning a bool and writing the result into an out variable). I also like `Attempt<T>` more because it's a noun, thereby aligning better than `Try<T>` with the noun-names of the other monadic wrappers. Finally, per my own convention, a function that returns an `Attempt<T>` shall have the 'Safely' prefix in its name, if it wraps potential Exceptions. Example for a signature: 

```
static async Task<Attempt<string>> SafelyProcessInputRequestAsync(string input) {...}
```

Note that my `Attempt<T>` combines what would typically be `Result<T>` and `Try<T>` into a single type, thanks to my custom `Failure` record:

```
public record Failure(Exception? Exception = null, UiString? Error = null); 

public record Attempt<T>
{
    internal T? Value { get; }
    internal Failure? Failure { get; }
    
    public bool IsSuccess => Failure == null;
    public bool IsFailure => !IsSuccess;

    etc...
}
```
  
#### 3) For Validation Error Collections: `Validation<T>`

Encapsulates a collection of validation results/errors (e.g. to show a user everything that was wrong with their input).

  
### Monadic Composition

Combined with .NET's `Task<T>` and `IEnumerable<T>`, these custom elevated types lend themselves for elegant monadic compositions and Railway Oriented Programming with the LINQ comprehension/query syntax - leading to workflows like the one below, which are declarative, fault-tolerant and so much more expressive compared to traditional, imperative style coding!

![Monadic Workflow / ROP](assets/images/ROP_Monadic_Workflow_Example.png)
