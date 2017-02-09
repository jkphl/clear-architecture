# A Clear Architecture

In my experience, development approaches like [Domain-Driven Design] and structural concepts as the [Hexagonal Architecture] or the [Onion Architecture] carry a lot of wisdom but don't necessarily provide practical guidance when it comes to starting off with a new project. After several unsatisfactory experiments, I felt a sort of relief when I first read about the [Clean Architecture], which nicely aggregates some high-level concepts while simplifying things at the same time.

However, even the Clean Architecture doesn't provide a simple-to-follow recipe for layouting a project, naming classes, files and directories and deciding where to settle a particular functionality. So I trial-and-errored myself to the point where I had a rather concise, opinionated implementation of the Clean Architecture that prove useful in several projects — even and especially in combination with each other. Let me introduce you to the **Clear Architecture**.  

*Note: I won't go into too much detail about the various high-level concepts involved in the Clear Architecture but rather focus on their practical application. Follow the links to learn more about them.*


## Key objectives

* Pragmatic, down-to-earth architecture with a fixed base layout and a concise set of rules and conventions
* Reasonable balance of simplicity, usability, abstraction and high-level concepts
* Independence of programming environment, frameworks and delivery mechanisms
* Suitable for building libraries (to be used by other packages)


## Three-tier architecture

In the Clear Architecture, source code is structured into 3 tiers that build on one another, most easily illustrated as concentric circles with the outermost one consisting of 3 complementary sectors.

<img src="https://cdn.rawgit.com/jkphl/clear-architecture/b1d9d3f2/images/clear-architecture-domain-application-client-tiers.svg" alt="Clear Architecture tiers" align="right" width="50%"/>


### ① Domain tier

* [**Domain** objects & services]
* High-level business rules

> In a banking application, the domain layer holds the definitions of the bank account, the account holder, the currency etc. as well as their relationships with each other.


### ② Application tier

* **Application** specific business rules & services
* [Use cases] orchestrating domain components
* Translating between external requests from the [③ client tier](#-client-tier-3-sectors) and domain logic (back and forth)

> The application layer executes different types of bank transactions, provides a currency exchange service and so on.  


### ③ Client tier (3 sectors)

* Low-level implementation details and mechanisms
* Public **ports** for external agencies (APIs, CLI, MVC components, etc.)
* **Infrastructural** details (persistence strategy, database platform, frameworks, 3rd party library bindings, etc.)
* Unit, functional and integration **tests**

> The client layer implements the database operations, provides a web interface for online banking and a [FinTS] interface to be used by external applications.

#### Ports

The *Ports* sector is the **public interface** of your application. It accept requests from external agencies (e.g. the Web, the command-line, an embedding system etc.), communicates them to the [infrastructure sector](#infrastructure) as well as the [② application layer](#-application-tier) and responds with the processed results. It represents your system to the outer world by taking the form of a web or native [GUI], a [CLI], a [REST API] or any other type of interface suitable for accessing your application. Components in this sector may include (but are not limited to):

* [Facades]
* [Interfaces]
* [Factories]
* [Exceptions] that might occur on the client layer
* [Constant definitions] useful for accessing your application 
* Controller / Action components of [MVC] / [ADR] architectures

*Note: If your application provides a web interface or similar API, don't directly use the `Ports` directory (or any of its subdirectories) as document root for web server scripts. Instead, create a top-level `public` directory and put your bootstrap script files there. Search for a similar solution when providing a CLI ([see below](#considerations-for-php-implementations) for PHP specific recommendations).*

#### Infrastructure

The *Infrastructure* sector is not strictly private, but it holds the implementation details that are not necessarily of public interest. While the *[Ports](#ports)* interface should be stable in the long run, it's acceptable for infrastructural components to change with the requirements. Ideally, they can be swapped against equivalent mechanisms without affecting the overall system functionality. External agencies should avoid accessing the infrastructure layer directly, but there might be exceptions for efficiency's sake. Typically in this sector:

* Third-party [libraries] and [frameworks] (they're always considered to be part of the infrastructure even if stored elsewhere and / or autoloaded)
* [Persistence] mechanisms (e.g. [database] platforms)
* Model / View / Presenter / Domain / Responder components of [MVC] / [MVP] / [ADR] architectures
* [Template engines] and templates
* Configuration data

#### Tests

The *Tests* sector holds all resources required for [testing your application]. In general, tests are nothing more than highly specialized clients of your application and must be granted full access to your [③ client](#-client-tier-3-sectors) and [② application](#-application-tier) layers. Test resources may also be accessed by external agencies (e.g. by extension) and typically include:

* Unit, integration, interface and / or acceptance test cases
* Test fixtures


## Directory layout

### Base skeleton

```
`-- src
    `-- <Module> 
        |-- Application
        |-- Domain
        |-- Infrastructure
        |-- Ports
        `-- Tests
```

* The top level directory `src` separates the actual source files from other package resources, e.g. documentation, dependency modules, web interface files etc. 
* `<Module>` must be replaced with a vendor-unique [UpperCamelCase] package name (e.g. `MyApp`).
* The 3rd level is made up of five directories representing the main architectural [tiers and sectors](#three-tier-architecture) .


### Application specifics

Inside the five main directories, your application may add additional structures as needed. However, to keep things consistent, I recommend sticking to these conventions:

* Directory and file names are always to be written in **UpperCamelCase**. I prefer using singular expressions wherever possible (i.e. `Factory` instead of `Factories`).
* If you have **multiple similar components**, that are mostly used by external agencies (e.g. on a lower architectural level or by an external package), **keep them at a common central location**. As an example, I typically use directories named `Facade`, `Contract`, `Service` or `Factory` for grouping classes with similar functionality.
* **Keep closely related components together**. If you have, for instance, a class definition that implements an interface as described in [The Dependency Inversion Principle](#the-dependency-inversion-principle), put them into the same directory instead of spreading them across the file system. This rule commonly outweighs the previous one — it might be a matter of taste in some situations though.
* If a lower architectural layer "mirrors" and extends the structure of a higher one, e.g. by providing concrete implementations of a high-level interface, try to use **common file and directory names accross levels**. This will help keeping the cross-boundary relationships in mind.


## Rules & Conventions

<img src="https://cdn.rawgit.com/jkphl/clear-architecture/b1d9d3f2/images/clear-architecture-dependency-rule.svg" alt="Clear Architecture tiers" align="right" width="50%"/>


### The Dependency Rule

In the Clear Architecture, source code dependencies may **only ever point to the same or an inward layer**.

> Nothing in an inner circle can know anything at all about something in an outer circle. In particular, the name of something declared in an outer circle must not be mentioned by the code in an inner circle. That includes functions, classes, variables or any other named software entity. (*[Clean Architecture], Robert C. Martin*)

In other words, it's perfectly fine to reference same or higher-level components e.g. by

* constructing class instances ("[objects]"),
* implementing [interfaces], using [traits] etc.,
* calling functions and methods or
* using classes, interfaces etc. for [typing].

Adhering to the Dependency Rule makes your application testable and flexible in terms of implementation details (persistence strategy, database platform, provided client APIs etc.).


### The Dependency Inversion Principle

In order to not violate the Dependency Rule, the [Dependency Inversion Principle] must be used whenever complex data needs to be passed across a boundary to an inward layer. Instead of expecting and directly referencing a low-level component (e.g. as a function parameter), the high-level layer provides and references an interface that must be implemented and inherited from by the caller. This way, the conventional dependency relationship is inverted and the high-level layer is decoupled from the low-level component.

![Dependency inversion by using an interface / abstract service class](https://cdn.rawgit.com/jkphl/clear-architecture/b1d9d3f2/images/clear-architecture-dependency-inversion.svg)


### Decoupling & Dependency Injection

In general, avoid tight coupling between components and prefer abstractions / [interfaces] over concrete implementations, thus enabling [polymorphism] and making your application a lot better testable. You will always have to deal with object instantiation and globals at some point, but try to limit their occurrences to a minimum and manage them smartly. Some type of [Dependency Injection] mechanism might be of great help, but please consider the [possible drawbacks][Dependency Injection drawbacks] as well. DI configuration should be part of the [*Infrastructure* sector](#infrastructure) or of a general bootstrap process, stored somewhere outside the layer directories altogether.


### Naming conventions

The following special components (and their files) must be named after their role:

* Interfaces must use the **`Interface`** suffix (e.g. `MyCustomInterface`)
* Traits must use the **`Trait`** suffix (e.g. `MyCustomTrait`)
* Factories must use the **`Factory`** suffix (e.g. `MyCustomFactory`). Public factory method names must use the `create` prefix (e.g. `createFromParams`). 


## Considerations for PHP implementations

*Note: I mostly use the Clear Architecture for projects written in PHP but it should be easy to adapt its principles to other environments as well. Please let me know if you succeed (or fail) in doing so.*

* Stick to commonly agreed coding style rules like [PSR-1] and [PSR-2]. Consider using code quality checkers like [Scrutinizer], [Code Climate] etc. Have a look at my [Clear Architecture Yeoman Generator] to speed up the installation and configuration of these tools.
* Apply [PSR-4] autoloading standards with `<Module>` being the **base directory** corresponding to the **namespace prefix** (e.g. `Jkphl\MyApp`). Follow the next point to get autoloading for free.
* Use [Composer] as the principal dependency manager for your projects. By default, Composer installs package dependencies into the top-level `vendor` directory.
* When providing a CLI, check out Composer's [vendor binary features].
* Use **environment variables** for configuration purposes (respectively the [phpdotenv] library or one of its equivalents).


## Recommended readings

* [S.O.L.I.D. Software Development, One Step at a Time][S.O.L.I.D. Software Development] by Derek Bailey
* [Clean Architecture in PHP] by Kristopher Wilson
* [Domain-Driven Design in PHP] by Carlos Buenosvinos et al.

##  Legal

Copyright © 2017 Joschi Kuphal <joschi@kuphal.net> / [@jkphl](https://twitter.com/jkphl). Licensed under the terms of the [MIT license](LICENSE.txt). Originally published at at https://jkphl.is/articles/clear-architecture-php

[ADR]: http://pmjones.io/adr/ "Action-Domain-Responder by Paul M. Jones"
[Clean Architecture in PHP]: https://leanpub.com/cleanphp "Clean Architecture in PHP by Kristopher Wilson"
[Clean Architecture]: https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html "The Clean Architecture by Robert C. Martin"
[Clear Architecture Yeoman Generator]: https://github.com/jkphl/generator-cleanphp "Clear Architecture PHP Project Yeoman Generator by Joschi Kuphal"
[CLI]: https://en.wikipedia.org/wiki/Command-line_interface "Command-line Interface on Wikipedia"
[Code Climate]: https://codeclimate.com/ "Automated code review | Code Climate"
[Composer]: https://getcomposer.org "Composer — Dependency Manager for PHP"
[Constant definitions]: https://en.wikipedia.org/wiki/Constant_(computer_programming) "Constants on Wikipedia"
[database]: https://en.wikipedia.org/wiki/Database "Database on Wikipedia"
[Dependency Injection]: https://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection on Wikipedia"
[Dependency Injection drawbacks]: https://sites.google.com/site/unclebobconsultingllc/blogs-by-robert-martin/dependency-injection-inversion "Dependency Injection Inversion by Robert C. Martin"
[Dependency Inversion Principle]: https://en.wikipedia.org/wiki/Dependency_inversion_principle "Dependency Inversion Principle on Wikipedia"
[Domain-Driven Design in PHP]: https://leanpub.com/ddd-in-php "Domain-Driven Design in PHP by Carlos Buenosvinos et al."
[Domain-Driven Design]: https://en.wikipedia.org/wiki/Domain-driven_design "Domain Driven Design on Wikipedia"
[**Domain** objects & services]: https://en.wikipedia.org/wiki/Domain_model "Domain model on Wikipedia"
[Exceptions]: https://en.wikipedia.org/wiki/Exception_handling "Exception handling on Wikipedia"
[Facades]: https://en.wikipedia.org/wiki/Facade_pattern "Facade pattern on Wikipedia"
[Factories]: https://en.wikipedia.org/wiki/Factory_(object-oriented_programming) "Factories on Wikipedia"
[FinTS]: https://en.wikipedia.org/wiki/FinTS "FinTS on Wikipedia"
[frameworks]: https://en.wikipedia.org/wiki/Software_framework "Software framework on Wikipedia"
[GUI]: https://en.wikipedia.org/wiki/Graphical_user_interface "Graphical User Interface on Wikipedia"
[Hexagonal Architecture]: http://alistair.cockburn.us/Hexagonal+architecture "The Hexagonal Architecture by Alistair Cockburn"
[Interfaces]: https://en.wikipedia.org/wiki/Protocol_(object-oriented_programming) "Interfaces on Wikipedia"
[libraries]: https://en.wikipedia.org/wiki/Library_(computing) "Library on Wikipedia"
[MVC]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller "Model-View-Controller on Wikipedia"
[MVP]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter "Model-View-Presenter on Wikipedia"
[objects]: https://en.wikipedia.org/wiki/Object-oriented_programming#Objects_and_classes "Object oriented programming on Wikipedia"
[polymorphism]: https://en.wikipedia.org/wiki/Polymorphism_(computer_science) "Polymorphism on Wikipedia"
[Onion Architecture]: http://jeffreypalermo.com/blog/the-onion-architecture-part-1/ "The Onion Architecture by Jeffrey Palermo"
[Persistence]: https://en.wikipedia.org/wiki/Persistence_(computer_science) "Persistence on Wikipedia"
[phpdotenv]: https://github.com/vlucas/phpdotenv "vlucas/phpdotenv by Vance Lucas on Github"
[PSR-1]: http://www.php-fig.org/psr/psr-1/ "PSR-1: Basic Coding Standard - PHP-FIG"
[PSR-2]: http://www.php-fig.org/psr/psr-2/ "PSR-2: Coding Style Guide - PHP-FIG"
[PSR-4]: http://www.php-fig.org/psr/psr-4/ "PSR-4: Autoloader - PHP-FIG"
[REST API]: https://en.wikipedia.org/wiki/Representational_state_transfer "Representational state transfer on Wikipedia"
[S.O.L.I.D. Software Development]: http://www.codemag.com/article/1001061 "S.O.L.I.D. Software Development, One Step at a Time by Derek Bailey" 
[Scrutinizer]: https://scrutinizer-ci.com/ "Measure and Improve Code Quality continuously with Scrutinizer"
[Template engines]: https://en.wikipedia.org/wiki/Template_processor "Template processor on Wikipedia"
[testing your application]: https://en.wikipedia.org/wiki/Software_testing#Testing_levels "Testing levels on Wikipedia"
[traits]: https://en.wikipedia.org/wiki/Trait_(computer_programming) "Traits on Wikipedia"
[typing]: https://en.wikipedia.org/wiki/Type_system "Type system on Wikipedia"
[UpperCamelCase]: https://en.wikipedia.org/wiki/Camel_case "Camel case on Wikipedia"
[Use cases]: https://en.wikipedia.org/wiki/Use_case "Use case on Wikipedia"
[vendor binary features]: https://getcomposer.org/doc/articles/vendor-binaries.md "Vendor binaries and the vendor/bin directory - Composer"
