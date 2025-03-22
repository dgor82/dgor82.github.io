Last Update: 22/03/2025

# My Dev Style Guide

For long-lived .NET projects, I generally follow the set of approaches and paradigms described below, to help me guide the development process towards elegant and maintainable code. I'm hoping/attempting to thereby avoid the complexity trap that so easily besets software projects.

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
- [II) Coding Style](#ii-coding-style)
  - [SOLID Principles](#solid-principles)
  - [Design by Contract](#design-by-contract)
  - [Composition over Inheritance](#composition-over-inheritance)
    - [Problems with implementation-inheritance](#problems-with-implementation-inheritance)
    - [Solution](#solution)
  - [Mixed Paradigm (OOP ⋃ FP)](#mixed-paradigm-oop--fp)
    - [Extending C# with Monadic Wrappers](#extending-c-with-monadic-wrappers)
      - [1) Instead of nullable reference types: `Option<T>`](#1-instead-of-nullable-reference-types-optiont)
      - [2) For potentially throwing operations: `Attempt<T>`](#2-for-potentially-throwing-operations-attemptt)
      - [3) For potentially failing operations:  `Result<T>`](#3-for-potentially-failing-operations--resultt)
      - [4) For validation error collections: `Validation<T>`](#4-for-validation-error-collections-validationt)
    - [Monadic Composition in C#](#monadic-composition-in-c)
    - [Immutable Collections in C##](#immutable-collections-in-c)
      - [1. What does immutability actually mean in C#?](#1-what-does-immutability-actually-mean-in-c)
      - [2. What are the performance implications of using immutable collections and how to deal with them?](#2-what-are-the-performance-implications-of-using-immutable-collections-and-how-to-deal-with-them)
      - [3. To avoid any potential for dogma or Cargo Cult around this: when do we actually benefit from the use of immutable collections?](#3-to-avoid-any-potential-for-dogma-or-cargo-cult-around-this-when-do-we-actually-benefit-from-the-use-of-immutable-collections)

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

My unit tests make use of a modified D.I. services container that inherits from the app's main container but replaces external dependencies (e.g. repositories with database access) with their mocks or stubs. Overall it would seem then, that I intuitively have subscribed to the classic (rather than mockist) [school of thought](https://martinfowler.com/articles/mocksArentStubs.html#ClassicalAndMockistTesting). I feel this makes my unit tests more realistic and less brittle in the face of continuous refactoring, thus better supporting the two overriding goals stated at the outset. 

In terms of terminology, I make a clear distinction when naming variables and types between mocks and stubs (more on that, see [Mocks Aren't Stubs (Martin Fowler)](https://martinfowler.com/articles/mocksArentStubs.html)):  

- **Mocks** allow the setting up of expected return values, verification of behaviour etc. via the [Moq](https://github.com/devlooped/moq) mocking framework  
- **Stubs** are the actual objects used by the code under test, with reduced/modified behaviour, and are often obtained by calling `.Object` on mocks.

### 2. Integration Tests

These are few, end-to-end tests covering the most common use-cases to establish the correct interplay of my business logic with its external dependencies like the database or external APIs. This is more about testing data flow and orchestration than about covering a large percentage of logic / code branches / edge cases / exotic flows.

For my integration tests, I rely on the main app's D.I. services container for resolution of all dependencies.
  
### 3. Acceptance Tests

The main purpose of these tests is to ensure the application meets end-user requirements, simulating real-world usage to validate the complete functionality and integration of all features. They run against the full system, including its external dependencies, without any mocking. 

While important, these tests would be far lower in number (in line with the typical test pyramid) and have been lower priority for me. In some cases, like projects based on a Telegram.Bot, they have been manual. For future automation of acceptance tests for GUI-based apps, high-level scenarios should be described in a language understandable by both technical and non-technical stakeholders - I will consider introducing [SpecFlow](https://specflow.org).

### 4. External Tests

These may exist in their own dedicated test module which doesn't have any visibility of any of the actual app. These are therefore isolated unit tests merely to learn/document/assert the behaviour of external libraries and frameworks, which fulfil any of the below criteria: 

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

My motivation for adopting Vertical Slicing is to ensure that any development efforts are focused on delivering small, incremental pieces of functionality that span all the architectural layers from the UI to the backend and/or all components and platforms. This translates to quicker iterations and feedback loops, which in turn guarantees tighter alignment of development with user feedback and  requirements.

## Domain-Driven Design (DDD)
(replaces 'Metaphor' from XP)

DDD emphasises a deep understanding of the domain to inform our software design. The central concept here is developing a 'ubiquitous language' which domain experts and coders share. This language is reflected in the naming and choice of abstractions in the code, making it partially comprehensible to non-technical stakeholders (eventually with a goal of moving towards an internal Domain-Specific Language (DSL)). DDD ensures our code stays closely aligned with business needs and the real-world subtleties of the domain. It also facilitates a common language and thus improved communication with non-technical domain experts. 

For me, this also implies that the code becomes the primary source of project documentation (with documentation related to DevOps and architecture a possible exception). I avoid explanatory comments inside the code-base except for cases where non-obvious or unusual externalities are involved (e.g. in config-related code). In most cases when I catch myself feeling the need to add a comment, it turns out there was an underlying naming or design issue. 

## Continuous Refactoring & Simple Design

As described in great detail by authors like Robert C. Martin (Uncle Bob), Martin Fowler, Kent Beck etc.

## Continuous Integration (CI)

For my solo or small-team projects I like to set up the following CI workflow, supported by local shell scripts for productivity / automation.

### CI Workflow Summary

Inspired and informed, among others, by [Trunk Based Development](https://trunkbaseddevelopment.com/5-min-overview/)

1. The main branch is protected from direct mergers, it only accepts mergers through PRs.

2. Developers work locally on short-lived feature branches.

3. When ready for merging...  
a) They run build & test locally for the entire solution for the relevant `Debug_*` configuration(s).  
b) On pass, they push their working branch to GitHub which triggers automated PR-creation.  
c) The subsequent merger into main then triggers a build, test & deploy run for the Release configuration of each **assembly** marked for deployment.  
d) In case of conflicts, these need to be resolved manually, followed by a renewed PR merger attempt.

1. This means, the PRs are merged into main **before** reviews: reviews shall be conducted post-merger and a corresponding GitHub project-task is generated automatically. 

This approach to CI supports a truly _continuous_ integration without delays from waiting for manual PR reviews.
Full test-coverage / TDD should ensure well-enough that no breaking changes are introduced into main. The entire workflow should be automated with project-specific shell scripts designed to run on dev's machines.

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

The SOLID principles guide overall system design & orchestration:

- S = Single Responsibility Principle
- O = Open/Closed Principle
- L = Liskov Substitution Principle
- I = Interface Segregation Principle
- D = Dependency Inversion Principle

## Design by Contract

DbC, inspired by Bertrand Meyer, the inventor of the Eiffel language, ensures that software components interact based on clearly defined specifications. This formal agreement on expected inputs, outputs, and side effects between components leads to more reliable and robust system behaviour, facilitating easier debugging and validation of software correctness. This complements the TDD approach described further above. 

**I believe in a pragmatic application of DbC by limiting its use to the outer edges of each component, i.e. where it interfaces with other components or third-party libraries.** 

## Composition over Inheritance

In OOP design, I prefer composition over inheritance. 

### Problems with implementation-inheritance
1. OOP Languages are designed with the assumption that sub-typing (for polymorphic use) and implementation sharing (to avoid duplication) go hand-in-hand. That's often true but not always, which is where things break down. 
2. When starting to build class hierarchies, I don't usually have enough foresight to get it right. The deeper the hierarchies grow and the more other modules come to depend on its specifics, the harder it is to change.
3. Sub classes come to depend on specific ways base classes further up the hierarchy implement things, in a way this breaks encapsulation.

### Solution
I avoid conflating sub-typing for polymorphism with implementation sharing for DRY!  
-> For polymorphism, I use interfaces (and avoid using default implementations)  
-> To achieve DRY, I compose objects that offer specific behaviour into the class requiring it.

Exceptions in the name of pragmatism are frequent though, especially in lower level code that other modules won't come to depend on, or when a very flat inheritance hierarchy (e.g. 1 level) is virtually guaranteed. 


## Mixed Paradigm (OOP ⋃ FP)

The .NET ecosystem offers the unique luxury to include C# and F# assemblies in a single solution/repository, with lower barriers for interoperability than for any other OOP/FP language pair. Here is how I'd ideally like to take full advantage of the relative strengths of both languages: 

1) C# as the solution's main language:  
Great for UI/application development, frictionless integration with third-party libraries and other mainstream OOP tasks

2) F# as a supplemental language (for naturally well-isolated, pure logic modules):  
Great for functional (sub-)domain modelling, elegant and resilient data transformations/algorithms...

Commercial realities may dictate sticking to C# exclusively though so F# is, in my case, probably reserved for pet projects. But even within my C# assemblies, I follow a mixed paradigm approach, following the mixed-paradigm nature of C# itself. This roughly means blending OOP principles for system organisation at the larger scale (SOLID, Dependency Injection, etc.) with a functional programming style (FP) for most of the actual code construction. 

More specifically, it means avoiding imperative code, mutability and stateful operations whenever feasible and carefully demarcating those classes that require statefulness. To reinforce this pattern I follow the convention of using the immutable `record` as my default type and `class` only when statefulness is required. 

This approach reduces side effects, and has made my code more predictable, easier to test and more suitable for concurrency and parallelism. As John Carmack argued so well in [this article](http://sevangelatos.com/john-carmack-on/), there are incremental benefits to be gained from moving towards functional style coding even within a traditional OOP language.

Luckily C# has evolved to include lots of great FP-related facilities and I draw heavily on them: Lambdas, LINQ, pattern matching, switch-expressions etc. To make up for a few still missing facilities in C# - mainly to enable [Railway Oriented Programming](https://fsharpforfunandprofit.com/rop/) (ROP) - I have created a library of light-weight monadic wrappers (see section below). 

This mixed-paradigm approach requires recognising where to draw the line, i.e. finding the most natural cleavage plane to resolve the inevitable tension between OOP and FP. Currently, some of the more advanced FP concepts like partial application or monadic transformation fall by the wayside in my C# code. At some point I thus even considered the use of [Language-Ext](https://github.com/louthy/language-ext) to move C# even closer to FP but distanced myself from that idea after further deliberation to ...:

a) **avoid** the extreme dependency on such a heavy-weight but only medium-popular library 

b) **avoid** further reduced readability of my C# code for most mainstream .NET devs

c) **avoid** pushing the limits of the paradigm too far, i.e. going too much against the grain of C#  

### Extending C# with Monadic Wrappers

#### 1) Instead of nullable reference types: `Option<T>`

`From [8SPHE]`

I apply three simple rules:
  
1. Enable nullable types consistently via `<Nullable>enable</Nullable>` in Directory.Build.props saved in solution root.
2. Very rarely make any types under my control nullable (with the exception of using them for the fields in my monadic wrappers and a few other low-level/technical code exceptions).
3. Instead, use `Option<T>` for any optional T, especially for Ts representing domain concepts.

#### 2) For potentially throwing operations: `Attempt<T>`

I use `Attempt<T>` to encapsulate the outcome of an operation that might throw an exception. I steer clear of the name `Try<T>` (idiomatic in the functional world) due to its clash with the specific semantics of the 'Try' prefix for methods in C# (i.e. returning a bool and writing the result into an out variable). I also like `Attempt<T>` more because it's a noun, thereby aligning better than `Try<T>` with the noun-names of the other monadic wrappers. 
  
#### 3) For potentially failing operations:  `Result<T>`

Encapsulates the return value of an operation that might either succeed or result in a user-facing error message.

#### 4) For validation error collections: `Validation<T>`

Encapsulates a collection of validation results/errors (e.g. to show a user everything that was wrong with their input). Only relevant in some projects.
  
### Monadic Composition in C#

Combined with .NET's `Task<T>` and `IEnumerable<T>`, these custom elevated types lend themselves for elegant monadic compositions and Railway Oriented Programming with the LINQ comprehension/query syntax (yes, I had to extend them with SelectMany() ('Bind' in FP-speak) overloads) - leading to workflows like the one below, which are declarative, fault-tolerant and, I find, so much more expressive compared to traditional, imperative style coding!

![Monadic Workflow / ROP](assets/images/ROP_Monadic_Workflow_Example.png)

### Immutable Collections in C##

In line with FP, I design for many of my collections (Lists, Sets, Dictionaries...) to be immutable. Immediately, three issues/questions arise though:

#### 1. What does immutability actually mean in C#?

In my mind, there are three aspects of immutability in a collection that are completely orthogonal and thus need to be treated separately. Let's explore by starting out with the obvious choice of `ImmutableList<T>`...

**Aspect-1: Immutability of Items** 

The items of our ImmutableList are not necessarily immutable themselves. If their properties can freely be mutated, would you still call the list immutable? At the least that is misleading. To ensure immutability of the items themselves, their properties' `set` access modifier should be `init` or at least `private set`. In C# a `record` created with positional properties by default use `init` and is therefore immutable (not considering non-destructive mutation via the `with` keyword). I therefore use records by default. 

**Aspect-2: Immutability of the Collection Object**

Developers can still call `.Add()` or `.Remove()` on our ImmutableList for non-destructive mutation. This protects the underlying (original) object which helps e.g. with thread safety and often one of the main objectives.

**Aspect-3: Conceptual Immutability of the Collection**

What if the intended immutability is about protecting against other developers' ability to perform  non-destructive mutation? That might be called for in case the collection represents some immutable concept whose integrity must be preserved throughout an application's lifetime (e.g. a fixed menu of operations). In this case having `ImmutableList<T>` as the underlying type is not sufficient and the API needs to, at least, expose / return it via the `IReadOnlyList<T>` interface, which doesn't offer developers mutation methods and thus carries a strong signal. It offers no guarantee, however, because a downcast e.g. to IList<T> is possible, making mutation methods available again. In rare cases, when a real guarantee is needed, the underlying collection would have to be wrapped in a `ReadOnlyCollection` type which can't be downcast. In case of Sets and Dictionaries, a `FrozenSet` and `FrozenDictionary` can be used since .NET 8 which also don't offer mutation methods. 


#### 2. What are the performance implications of using immutable collections and how to deal with them?

Writing to and reading from an `ImmutableList<T>` is one to two orders of magnitudes slower than `List<T>`, for large collections this can be significant. If we know that no (or hardly no) writes are needed, an `ImmutableArray<T>` offers full, mutable read performance (and much more terrible write performance). 

If we need a Set or Dictionary instead of a List and no writing is needed then `FrozenSet` and `FrozenDictionary` (from .NET 8) offer highly optimised read performance. And when batch mutating any immutable type (e.g. in a loop), simply use the `Builder` before converting back to the immutable type. 

In conclusion, I carefully choose the appropriate type/pattern in the spirit of avoiding premature pessimisation. 

#### 3. To avoid any potential for dogma or Cargo Cult around this: when do we actually benefit from the use of immutable collections?

I have come to the conclusion that there is no need for using immutability for locally scoped, private collections which are not passed to other modules and where there is no chance of multi-threaded / shared access (e.g. in single threaded code within a object scoped within a single function invocation). This description may well fit a large majority of collections in a code base. 

In all other cases, I use immutable collections (with a suitable selection regarding the three immutability aspects described above). Examples where it's especially relevant are:
- when passing collections across module/class boundaries and Snapshot Semantics are desirable (for easier reasoning about code etc.)
- when there is a chance for multi-threaded access without any synchronisation
- when representing events from an Event Sourcing datastore
- when efficient equality comparisons are needed (computed hash codes can simply be cached)

