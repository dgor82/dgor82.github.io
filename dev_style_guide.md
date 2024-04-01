In my .NET / C# projects, I generally follow the set of approaches and paradigms described below, to guide the development process towards excellence and a maintainable codebase - thereby attempting to avoid the complexity trap that besets so many software projects.

Also check the [detailed guide](https://github.com/dgor82/devtools#continuous-integration-ci) to my CI Workflow / Setup supported by shell scripts and GitHub Action workflows.

# I) Dev & Team-Work Methodology

Inspirations/sources:
- [The Agile Manifesto](https://agilemanifesto.org)
    - As fully explained in: [Clean Agile](https://www.goodreads.com/book/show/45280021-clean-agile)
- [Manifesto for Software Craftsmanship](http://manifesto.softwarecraftsmanship.org)
    - As fully explained in: [The Software Craftsman](https://www.goodreads.com/book/show/23215733-software-craftsman-the)
- [Extreme Programming (XP)](https://www.goodreads.com/book/show/67833.Extreme_Programming_Explained)
- [The Pragmatic Programmer](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)
- [Domain-Driven Design](https://www.goodreads.com/book/show/179133.Domain_Driven_Design)
- [Effective Software Testing: A developer's guide](https://www.goodreads.com/book/show/59796908-effective-software-testing)

## Test-Driven Development (TDD)

TDD involves writing tests before production code, ensuring every new feature starts with a failing test. This leads to a test coverage by default, encouraging simpler, more modular code and allowing for easier refactoring and assurance that new changes do not break existing functionality. Experience shows that the initial, additional effort pays off after just a few months, after which the development speed increases dramatically compared to a non-TDD approach. Under TDD, test code is a first-class citizen i.e. the same standards are applied as to the production code. 

However, as usual, pragmatism rules supreme: let us not end up with absurdities like unit tests for simple getters and setters. By default, I am more oriented towards the 'classic school' than the 'mockist school', leading to larger unit tests and soft boundaries between unit-, integration- and functional-tests - with a very selective reliance on mocking. The following sections define my understanding of the different kind of tests involved and the (soft) boundaries between them.

### **1. Integration Tests:** 

Few, end-to-end tests covering the most common use-cases to establish correct interplay of my business logic with its external dependencies like database or external APIs.

They rely on the main code's D.I. container for resolution of all dependencies (with occasional exception, where mocking might facilitate e.g. throwing an exception from an object deep down).   
  
### **2. Functional Tests:**

The tests can traverse multiple architectural layers and their purpose is to test the end-to-end business logic, covering _all_ of the application's supported use-cases. 

They also rely on the main code’s D.I. container for resolution of internal dependencies and use mocks for external dependencies, frameworks and mechanisms (including the DB) where needed.   
  
### 3. Unit Tests:

Their purpose is to test and document interesting/critical/error-prone units intensely. Here, thorough analysis of boundary cases etc. comes into play. An example would be the factories and/or constructors for all central Domain POCOs.   

These would NOT rely on the D.I. container, and use a mix of manually resolved real dependencies (for closely related classes) and mocks. 

### **4. Acceptance Tests**

These tests will ensure the application meets end-user requirements, simulating real-world usage to validate the complete functionality and integration of all features. Unlike other tests, they run against the full system, including its external dependencies, without mocking. This level of testing aims to detect any issues in the user experience before the software goes live, using high-level scenarios often described in a language understandable by both technical and non-technical stakeholders. 

Using e.g. [SpecFlow](https://specflow.org) to help articulate these tests in .NET

### **5. External Tests**

These exist in their own dedicated test module which doesn't have any visibility of any of my other modules. These are therefore isolated unit tests to learn/document/assert the behaviour of external libraries and frameworks, which fulfil any of the below criteria: 

- they are not-obvious and so benefit from executable documentation
- I had to 'learn' to use them (--> learning via unit testing)
- the library has only medium or even low popularity (thus, asserting its functionality that I rely on across my code is important, in case future updates break it - which is far less likely for mega-popular libraries)

## Vertical Slicing

By adopting Vertical Slicing, I ensure any development efforts are focused on delivering small, incremental pieces of functionality that span all the architectural layers from the UI to the backend and all components and platforms. This facilitates quicker iterations and feedback loops, directly aligning development with user feedback and business requirements, reducing the risk of misaligned features.

## Domain-Driven Design (DDD)
(replaces 'Metaphor' from XP)

DDD emphasises a deep understanding of the domain to inform our software design. The central concept here is developing a 'ubiquitous language' which domain experts and coders share. This language is reflected in the naming and choice of abstractions in the code, making it partially comprehensible to non-technical stakeholders (eventually with a goal of moving towards an internal Domain-Specific Language (DSL)). DDD ensures our code stays closely aligned with business needs and the real-world subtleties of the domain. It also facilitates a common language and thus clearer communication across all stakeholders. 

This also implies that the code is the documentation. I'm strictly against extensive technical documentation outside of the code-base (except for high-level requirements or architectural considerations) or extensive explanatory comments inside the code-base. Such comments, in most cases, just mask bad naming and/or badly designed code. 

## Continuous Refactoring & Simple Design

As described in detail by Uncle Bob, Martin Fowler, Kent Beck etc.

# II) Coding Style

Inspirations from
- [Clean Code](https://www.goodreads.com/book/show/3735293-clean-code)
- Clean Architecture
    - [Summary](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
    - [Full Book](https://www.goodreads.com/book/show/18043011-clean-architecture)
- [The Pragmatic Programmer](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)
- [Functional Programming in C#](https://www.goodreads.com/book/show/31550964-functional-programming-in-c)
- [FP vs. OO (by Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2018/04/13/FPvsOO.html)
- [John Carmack on Functional Programming in C++](http://sevangelatos.com/john-carmack-on/)

## SOLID Principles

Adherence to the SOLID Principles, esp. where OO design is at play.

- S = Single Responsibility Principle
- O = Open/Closed Principle
- L = Liskov Substitution Principle
- I = Interface Segregation Principle
- D = Dependency Inversion Principle

## Mixed Paradigm

I follow a mixed paradigm approach to coding, following the mixed-paradigm nature of C# itself. This means blending object-oriented (OO) principles (for system organisation at the larger scale, reliance on Dependency Injection Frameworks etc.) with a functional programming style (FP) (for most of actual code at the smaller scale i.e. avoiding imperative-style coding, mutability and stateful operations whenever feasible). 

This approach reduces side effects, making the code more predictable, easier to test and more suitable for concurrency and parallelism. To achieve this it helps to draw on Lambdas, LINQ, pattern matching, switch-expressions etc. (all natively supported by C#) and, mainly for backend code, depending on circumstances and team skills, consider the cautious introduction of [Language-Ext](https://github.com/louthy/language-ext) (a popular 3rd-party library to move C# even closer to FP, e.g. by allowing monadic composition for elevated types beyond just IEnumerables as natively supported by LINQ).

## Design by Contract

DbC, inspired by Bertrand Meyer, the inventor of the Eiffel language, ensures that software components interact based on clearly defined specifications. This formal agreement on expected inputs, outputs, and side effects between components leads to more reliable and robust system behaviour, facilitating easier debugging and validation of software correctness. This complements the TDD approach described further above. I believe in a pragmatic application of DbC by limiting its use to the outer edges of each component, i.e. where it interfaces with other components or third-party libraries. 

## Nullable Reference Types (C# specific)

`From [8SPHE]`

Rely on consistent and disciplined usage of NRTs (nullable reference types) and the default of enabled Nullability, instead of on FP's custom `Option<T>` type - with the same goal of keeping functions honest and my code free of clutter. 

Augment the compiler's ability to do advanced static analysis of null-states via [# Attributes for null-state static analysis](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis?source=docs&ns-enrollment-type=Collection&ns-enrollment-id=bookmarks)

Over time, possibly as part of gradual refactoring towards deeper use of FP, e.g. with Language-Ext and esp. in backend code, explore introducing Option<> after all in certain vertical slices...
