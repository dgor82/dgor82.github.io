# My Dev Style Guide

In my .NET / C# projects, I generally follow the set of approaches and paradigms described below, to guide the development process towards excellence and a maintainable codebase - thereby attempting to avoid the complexity trap that besets so many software projects.

Also check the [detailed guide](https://github.com/dgor82/devtools#continuous-integration-ci) to my CI Workflow / Setup supported by shell scripts and GitHub Action workflows.

# ToC

- [My Dev Style Guide](#my-dev-style-guide)
- [ToC](#toc)
- [I) Dev \& Team-Work Practices](#i-dev--team-work-practices)
  - [Test-Driven Development (TDD)](#test-driven-development-tdd)
    - [1. Integration Tests](#1-integration-tests)
    - [2. Functional Tests](#2-functional-tests)
    - [3. Unit Tests](#3-unit-tests)
    - [4. Acceptance Tests](#4-acceptance-tests)
    - [5. External Tests](#5-external-tests)
  - [Vertical Slicing](#vertical-slicing)
  - [Domain-Driven Design (DDD)](#domain-driven-design-ddd)
  - [Continuous Refactoring \& Simple Design](#continuous-refactoring--simple-design)
  - [Continuous Integration (CI)](#continuous-integration-ci)
    - [CI Workflow Summary](#ci-workflow-summary)
    - [Scripts supporting the CI Workflow](#scripts-supporting-the-ci-workflow)
      - [Start Work](#start-work)
      - [Finish Work](#finish-work)
      - [The triggered fb/\* Workflow](#the-triggered-fb-workflow)
      - [The triggered main Workflow](#the-triggered-main-workflow)
      - [Clean-Up Branches](#clean-up-branches)
- [II) Coding Style](#ii-coding-style)
  - [SOLID Principles](#solid-principles)
  - [Mixed Paradigm](#mixed-paradigm)
  - [Design by Contract](#design-by-contract)
  - [Nullable Reference Types (C# specific)](#nullable-reference-types-c-specific)

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

## Test-Driven Development (TDD)

TDD involves writing tests before production code, ensuring every new feature starts with a failing test. This leads to a test coverage by default, encouraging simpler, more modular code and allowing for easier refactoring and assurance that new changes do not break existing functionality. Experience shows that the initial, additional effort pays off after just a few months, after which the development speed increases dramatically compared to a non-TDD approach. Under TDD, test code is a first-class citizen i.e. the same standards are applied as to the production code. 

However, as usual, pragmatism rules supreme: let us not end up with absurdities like unit tests for simple getters and setters. By default, I am more oriented towards the 'classic school' than the 'mockist school', leading to larger unit tests and soft boundaries between unit-, integration- and functional-tests - with a very selective reliance on mocking. The following sections define my understanding of the different kind of tests involved and the (soft) boundaries between them.

### 1. Integration Tests

Few, end-to-end tests covering the most common use-cases to establish correct interplay of my business logic with its external dependencies like database or external APIs.

They rely on the main code's D.I. container for resolution of all dependencies (with occasional exception, where mocking might facilitate e.g. throwing an exception from an object deep down).   
  
### 2. Functional Tests

The tests can traverse multiple architectural layers and their purpose is to test the end-to-end business logic, covering _all_ of the application's supported use-cases. 

They also rely on the main code’s D.I. container for resolution of internal dependencies and use mocks for external dependencies, frameworks and mechanisms (including the DB) where needed.   
  
### 3. Unit Tests

Their purpose is to test and document interesting/critical/error-prone units intensely. Here, thorough analysis of boundary cases etc. comes into play. An example would be the factories and/or constructors for all central Domain POCOs.   

These would NOT rely on the D.I. container, and use a mix of manually resolved real dependencies (for closely related classes) and mocks. 

### 4. Acceptance Tests

These tests will ensure the application meets end-user requirements, simulating real-world usage to validate the complete functionality and integration of all features. Unlike other tests, they run against the full system, including its external dependencies, without mocking. This level of testing aims to detect any issues in the user experience before the software goes live, using high-level scenarios often described in a language understandable by both technical and non-technical stakeholders. 

Using e.g. [SpecFlow](https://specflow.org) to help articulate these tests in .NET

### 5. External Tests

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

This process supports a truly _continuous_ integration without delays from waiting for manual PR reviews.
Full test-coverage / TDD should ensure well-enough that no breaking changes are introduced into main.

Note: `rollForward` property in global.json should be explicitly set to "disable", because any active rollForwarding strategy would mislead the sdk caching procedure in GitHub Actions which rely on a hash of global.json. For a detailed discussion, see the end of: https://chat.openai.com/share/084972a7-e536-4f72-8a30-3c1e1e361481

### Scripts supporting the CI Workflow

The above CI process can be handled manually but certain shell (bash) scripts (for UNIX-based machines), for example, automate many of the manual/repetetive tasks associated with it:
- start_work.sh
- finish_work.sh
- clean_up_local_branches.sh

#### Start Work
Do daily dev work on a `tmp/*` branch to allow for regular pushing to GitHub (for backup) without triggering the CI/CD workflow every time (to save GH Runner resources). To start dev, run the 'start_work.sh' script which does this:
* Checkouts to and updates the local `main` branch
* From `main`, checkouts into a new branch by name of `tmp/[randomly-generated-alphanumeric-string]` (the branch will get a meaningful, summarising name in the next step, when work on it was finished)

#### Finish Work
When done developing on the current `tmp/*` working branch, run the 'finish_work.sh' script which does this:
* Updates local tracking branch `origin/main` with any new commits (e.g. those made by other devs) since you branched off from it in step-1.
* Checks whether your working branch still is a direct ancestor of `origin/main` 
    * if that's not the case, **rebases** (!) your working branch on the newly updated `main` branch. This essentially rewrites history, pretending that we have made our current local developments on top of the very latest developments of others.
* Extracts all `Debug_*` configurations from the .sln file in the working directory. 
    * In case the script was run with option `--noios` ignores release configurations that contain the string 'ios'
    * In case the script was run with option `--nomob` ignores release configurations that contain either 'ios' or 'android'
    * These options were added because mobile builds can be very slow, and if we are sure we don't need to build/test them, this greatly accelerates execution of the script. Use with caution!
* Restores dependencies, builds solution and runs tests for each of the relevant Debug configurations. This step ensures that everything still builds and passes all tests - and is especially important in case your working branch was rebased on other devs' changes in the previous step. 
* Shows current (semantic) version and asks you for a new one. Then:
    * Updates version.txt in the solution root
    * Updates any relevant version tags in .csproj files 
    * Increments by one the `<ApplicationVersion>` tag in any 'android' .csproj - this special version for Google Play needs to be an integer and doesn't follow the semantic versioning pattern.
* Checks out to a new branch, and asks you for its name, which should represent a meaningful summary of all your commits and will be used as the title for a new PR. 
    * The script automatically prefixes the branch name with `fb/` which, by convention, is the required pattern to trigger my typical CI workflows in GitHub Actions. 
* Pushes the new branch to `origin` and thus triggers the CI workflow

#### The triggered fb/* Workflow
GitHub Actions Workflow on the `fb/*` branch is triggered by previous step.
* Creates a corresponding PR from your feature branch
* Attempts merging this PR into `main` (using 'squash' to avoid cluttering the commit history of `main` with details!)
* Deletes the `fb/*` branch again if successful
* Triggers the CI workflow on the `main` branch

**Note-1:**
A rare reason for failure could be a merging conflict between the `fb/*` branch and `main`. This should be rare because, if following the finish_work script described above, the latest changes from `origin/main` would have been pulled and rebased locally. Any conflicts should have shown up at that point, for you to resolve locally i.e. before the scripts completes and triggers the CI workflow on GitHub Actions.

**Note-2:**
No build & test run here yet - this was just run on your local dev machine as part of the Finish Work step, see above. The full build & test cycle will run on the `main` workflow in the next step though for each project marked for deployment. I.e. this avoids a triple running of the cycle, which can be overkill and slow down the CI workflow. The combination of running it twice (once locally before merger, solution-wide, in Debug_* configurations, and once on `main` post merger in Release configuration) should be sufficient for most normal projects.

**Note-3:**
This should also automatically generate a GitHub Project Todo for post-merger review. This can be set up in the 'default workflow' settings in the GitHub Project U.I. found under: https://github.com/orgs/YOUR_ORG/projects/YOUR_PROJECT_ID/workflows

#### The triggered main Workflow
GitHub Action Workflow on `main` branch is triggered by previous step
* Based on version.txt, creates and pushes a new version tag to the latest commit
* Uses a matrix strategy to run build & test jobs for all Non-iOS deployment projects (iOS builds require 10 x more expensive, separate macOS GitHub Action Runners or local mac build machines)
* Uses caching to avoid repeated downloads of SDKs, workloads and dependencies (relies on `rollForward` in global.json being set to "disable", see explanation above.)
* CD: Deploys any non mobile app code (e.g. cloud-based backend services or browser-based client software etc.).

#### Clean-Up Branches
After several iterations of the above CI flow, multiple outdated `tmp/*` and `fb/*` branches will have accumulated in the local git repository. The 'clean_up_local_branches.sh' script iterates through all of them and allows you quick & easy force deletion. Normally, there shouldn't be any branches left lingering on origin thanks to the configured auto branch deletion after PR mergers. 

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

## SOLID Principles

Adherence to the SOLID Principles, esp. where OO design is at play.

- S = Single Responsibility Principle
- O = Open/Closed Principle
- L = Liskov Substitution Principle
- I = Interface Segregation Principle
- D = Dependency Inversion Principle

## Mixed Paradigm

I follow a mixed paradigm approach to coding, following the mixed-paradigm nature of C# itself. This means blending object-oriented (OO) principles for system organisation at the larger scale according to SOLID principles with a functional programming style (FP) for most of the actual code construction at the smaller scale (i.e. avoiding imperative code, mutability and stateful operations whenever feasible and carefully demarcating the group of classes that require statefulness). 

This approach reduces side effects, making the code more predictable, easier to test and more suitable for concurrency and parallelism. To achieve this it helps to draw on Lambdas, LINQ, pattern matching, switch-expressions etc. (all natively supported by C#). Despite my mantra for a single language across the entire system, I'd consider mixing a F# project into the .NET solution for backend code in experimental / pet projects only (unless circumstances and team skills allow it for a commercial project). After all, it is one of the small luxurires of the .NET ecosystem that C# and F# both run on the Common Language Runtime (CLR) and can easily be mixed in a single solution.

Previously I was considering the cautios introduction of [Language-Ext](https://github.com/louthy/language-ext) into commercial projects. It's a popular 3rd-party library to move C# even closer to FP, e.g. by allowing monadic composition for elevated types beyond just IEnumerables (which C# already supports natively via LINQ) like `Option<>`, `Task<>` or `Either<>`. After further delibaration I have distanced myself from that idea in favour of 'paradigmatic integrity' and the more crystallised approach described above. 

## Design by Contract

DbC, inspired by Bertrand Meyer, the inventor of the Eiffel language, ensures that software components interact based on clearly defined specifications. This formal agreement on expected inputs, outputs, and side effects between components leads to more reliable and robust system behaviour, facilitating easier debugging and validation of software correctness. This complements the TDD approach described further above. I believe in a pragmatic application of DbC by limiting its use to the outer edges of each component, i.e. where it interfaces with other components or third-party libraries. 

## Nullable Reference Types (C# specific)

`From [8SPHE]`

Rely on consistent and disciplined usage of NRTs (nullable reference types) and the default of enabled Nullability, instead of on FP's custom `Option<T>` type - with the same goal of keeping functions honest and my code free of clutter. 

Augment the compiler's ability to do advanced static analysis of null-states via [# Attributes for null-state static analysis](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis?source=docs&ns-enrollment-type=Collection&ns-enrollment-id=bookmarks)

Over time, possibly as part of gradual refactoring towards deeper use of FP, e.g. with Language-Ext and esp. in backend code, explore introducing Option<> after all in certain vertical slices...
