Last Update: 17/04/2025

# Dev & Team-Work Practices

For long-lived .NET projects, I generally follow the set of approaches and paradigms described below, to help me guide the development process towards elegant and maintainable code. I'm hoping/attempting to thereby avoid the complexity trap that so easily besets software projects.

# ToC
- [Dev \& Team-Work Practices](#dev--team-work-practices)
- [ToC](#toc)
- [Inspirations/sources:](#inspirationssources)
- [Testing](#testing)
  - [1. Unit Tests](#1-unit-tests)
  - [2. Integration Tests](#2-integration-tests)
  - [3. Acceptance Tests](#3-acceptance-tests)
  - [4. External Tests](#4-external-tests)
  - [Why no test-coverage tools?](#why-no-test-coverage-tools)
  - [Why no TDD?](#why-no-tdd)
- [Vertical Slicing](#vertical-slicing)
- [Domain-Driven Design (DDD)](#domain-driven-design-ddd)
- [Comments](#comments)
- [Continuous Refactoring \& Simple Design](#continuous-refactoring--simple-design)
- [Continuous Integration (CI)](#continuous-integration-ci)
  - [CI Workflow Summary](#ci-workflow-summary)

# Inspirations/sources:

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
- [A Philosophy of Software Design vs Clean Code](https://github.com/johnousterhout/aposd-vs-clean-code/blob/main/README.md?utm_source=substack&utm_medium=email)
- [Public vs. Published Interfaces (Martin Fowler)](https://martinfowler.com/ieeeSoftware/published.pdf)

 
# Testing

My overriding goal is fearless continuous deployment and fearless continuous refactoring.
  
To achieve this goal, I take testing very seriously, which also leads to simpler design, and more modular, decoupled code. 

I have come to the view that the test code (a first-class citizen in terms of coding standards) becomes the *full* specification, *executable* documentation and even a fairly good proxy for *the user*!

The following sections describe how I interpret and apply the 'test pyramid' - with the different kinds of tests in it, and the (sometimes soft) boundaries between them. I end with a few words on why I don't use test coverage tools.

## 1. Unit Tests

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

## 2. Integration Tests

These are few, end-to-end tests covering the most common use-cases to establish the correct interplay of my business logic with its external dependencies like the database or external APIs. This is more about testing data flow and orchestration than about covering a large percentage of logic / code branches / edge cases / exotic flows.

For my integration tests, I rely on the main app's D.I. services container for resolution of all dependencies.
  
## 3. Acceptance Tests

The main purpose of these tests is to ensure the application meets end-user requirements, simulating real-world usage to validate the complete functionality and integration of all features. They run against the full system, including its external dependencies, without any mocking. 

While important, these tests would be far lower in number (in line with the typical test pyramid) and have been lower priority for me. In some cases, like projects based on a Telegram.Bot, they have been manual. For future automation of acceptance tests for GUI-based apps, high-level scenarios should be described in a language understandable by both technical and non-technical stakeholders - I will consider introducing [SpecFlow](https://specflow.org).

## 4. External Tests

These may exist in their own dedicated test module which doesn't have any visibility of any of the actual app. These are therefore isolated unit tests merely to learn/document/assert the behaviour of external libraries and frameworks, which fulfil any of the below criteria: 

- They are not-obvious and so benefit from executable documentation
- I had to 'learn' to use them (--> learning via unit testing)
- The library has only medium or even low popularity (thus, asserting its functionality that I rely on across my code is important, in case future updates break it - which is far less likely for mega-popular libraries)

## Why no test-coverage tools?

On a project created by 'test-infected' developers, where coverage will by default already be large, they don't add much value but create considerable overhead and distraction.
  
The crux is that '100%' test coverage is a totally meaningless measurement target!

1 - When sticking to a coarse measurement like 'statement coverage', then the '100%' is easy to attain but doesn't actually lead to sufficient coverage of branches / conditions / decisions! This is the case, for instance, with JetBrain's dotCover.   
  
Consider this C# statement:
```
var userId = telegramInputMessage.From?.Id 
              ?? throw new ArgumentNullException(nameof(telegramInputMessage),
                  "From.Id in the input message must not be null");        
```

A happy-path test that never touches the `throw` branch of this statement still causes dotCover to report a 100% coverage. This is misleading and perversely incentivises use of a less terse `if` statement syntax (which works as expected). Tail wagging the dog!

2 - When using real coverage (i.e. path-coverage), 100% is unattainable anyway due to the combinatorial explosion (for any code with a fair amount of complexity). 

3 - One thus ends up obsessing about and fiddling with:
- code coverage methodologies
- coverage statistics (invariably aiming for the meaningless 100% for lack of any other sensible target)
- coverage filter settings
- etc.
  
... time and attention that would be better spent thinking deeply about important boundary cases!

## Why no TDD?

Notice that I am not following 'Test-Driven Development' despite advocacy by many of my programming heroes. This is one of the points where I side with John Ousterhout in his [epic debate](https://github.com/johnousterhout/aposd-vs-clean-code/blob/main/README.md?utm_source=substack&utm_medium=email) with Uncle Bob. In short, I found that the very tactical back- and forth between test code and production code, in seconds-long cycles as per true TDD, indeed distracted me from higher-level, design-oriented thinking in larger chunks. This style guide is neutral on the use of TDD as long as you otherwise take tests as seriously as I do. 

# Vertical Slicing

My motivation for adopting Vertical Slicing is to ensure that any development efforts are focused on delivering small, incremental pieces of functionality that span all the architectural layers from the UI to the backend and/or all components and platforms. This translates to quicker iterations and feedback loops, which in turn guarantees tighter alignment of development with user feedback and  requirements.

# Domain-Driven Design (DDD)
(replaces 'Metaphor' from XP)

DDD emphasises a deep understanding of the domain to inform our software design. The central concept here is developing a 'ubiquitous language' which domain experts and coders share. This language is reflected in the naming and choice of abstractions in the code, making it partially comprehensible to non-technical stakeholders (eventually with a goal of moving towards an internal Domain-Specific Language (DSL)). DDD ensures our code stays closely aligned with business needs and the real-world subtleties of the domain. It also facilitates a common language and thus improved communication with non-technical domain experts. 

# Comments

The code is the primary source of project documentation (with documentation related to DevOps and architecture a possible exception). I avoid explanatory comments inside the code-base. In most cases when I catch myself feeling the need to add a comment, it turns out there was an underlying naming or design issue. On this issue I side with Uncle Bob in his [epic debate](https://github.com/johnousterhout/aposd-vs-clean-code/blob/main/README.md?utm_source=substack&utm_medium=email) with John Ousterhout. 

However, there are important exceptions:
- Cases where non-obvious or unusual externalities are involved (e.g. in config-related code)
- Explanatory `XML doc comments` on deep *public* methods (and even more important on *published* methods, see distinction in [Fowler](https://martinfowler.com/ieeeSoftware/published.pdf)) i.e. those that are non-obvious and hide a good amount of complexity. I usually stick to the `summary` tag: it's redundant to specify the `params` or `return` values, they are already visible in the signature.

# Continuous Refactoring & Simple Design

As described in great detail by authors like Robert C. Martin (Uncle Bob), Martin Fowler, Kent Beck etc.

# Continuous Integration (CI)

For my solo or small-team projects I like to set up the following CI workflow, supported by local shell scripts for productivity / automation.

## CI Workflow Summary

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
Full test-coverage should ensure well-enough that no breaking changes are introduced into main. The entire workflow should be automated with project-specific shell scripts designed to run on dev's machines.
