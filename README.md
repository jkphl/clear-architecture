# A Clear Architecture

In my experience, development approaches like [Domain-Driven Design] and structural concepts as the [Hexagonal Architecture] or the [Onion Architecture] carry a lot of wisdom but don't necessarily provide practical guidance when it comes to starting off with a new project. After several unsatisfactory experiments, I felt a sort of relief when I first read about the [Clean Architecture], which nicely aggregates some high-level concepts while simplifying things at the same time. However, even the Clean Architecture doesn't provide a simple-to-follow recipe for layouting a project, naming classes, files and directories and deciding where to settle a particular functionality. So I trial-and-errored myself to the point where I had a rather concise, opinionated implementation of the Clean Architecture that prove useful in several projects, even and especially in combination with each other. Le me introduce you to the **Clear Architecture**.

*Note: I won't go into too much detail regarding the various high-level concepts but rather focus on the practical side of things. Please see at the end for a list of readings I recommend for further understanding. Also, while I mainly use the Clear Architecture for PHP projects, it should be easily adoptable to other environments as well. Please let me know if you succeed (or fail) in doing so.*


## Key objectives

* Pragmatic, down-to-earth architecture with a fixed base layout and a concise set of rules and conventions
* Reasonable balance of simplicity, usability, abstraction and high-level concepts
* Independence of frameworks and delivery mechanisms
* Suitable for building libraries (to be included by other packages)


## Three-tier architecture

<img src="https://cdn.rawgit.com/jkphl/generator-cleanphp/3306407b/doc/clear-architecture-domain-application-client-tiers.svg" alt="Clear Architecture tiers" align="right" width="50%"/>


### ① Domain tier

* **Domain** objects & services
* High-level business rules

> In a banking application, the domain layer holds definitions of a bank account, an account holder, a currency etc. as well as their relationships with each other.


### ② Application tier

* **Application** specific business rules & services
* Use cases orchestrating domain objects & services
* Translating between external requests and domain logic (back and forth)

> The application layer provides a currency exchange service, executes different types of bank transactions and so on.


### ③ Client tier (3 sectors)

* Low-level implementation details and mechanisms
* Public **ports** for external agencies (APIs, CLI, MVC components, etc.)
* **Infrastructural** details (persistence, database, frameworks, 3rd party library bindings)
* Unit, functional and integration **tests**

> The client layer implements the persistence infrastructure (e.g. a database), provides a web interface for online banking and a [FinTS] interface to be used by external applications.

#### Ports

The *Ports* sector is the **public interface** of your application. It accept requests from external agencies (e.g. the Web, the command-line, an embedding system etc.), communicates them to the [infrastructure sector](#infrastructure) and the ② application layer and sends back the processed results. It represents your system to the outer world by taking the form of a web or native [GUI], a [CLI], a [REST API] or any other type of interface suitable for accessing your application. Components in this sector may include (but are not limited to):

* [Facades]
* [Interfaces]
* [Factories]
* [Exceptions] that might occur on the client layer
* [Constant definitions] useful for accessing your application
* Controller / Action components of [MVC] / [ADR] architectures

*Note: If your application provides a web interface or similar API, don't directly use the `Ports` directory (or any of its subdirectories) as document root for web server scripts. Instead, create a top-level `public` directory and put your bootstrap script files there. Search for a similar solution when providing a CLI ([see below](#considerations-for-php-implementations) for PHP specific recommendations).*

#### Infrastructure

The *Infrastructure* sector is not strictly private, but it https://github.com/jkphl/generator-cleanphp/blob/master/doc/CLEAR-ARCHITECTURE.md#considerations-for-php-implementationsat are not necessarily of public interest. While the outward *[Ports](#ports)* interface should be stable in the long run and generally not subject to alterations, it's acceptable for infrastructural components to change with thehttps://github.com/jkphl/generator-cleanphp/blob/master/doc/CLEAR-ARCHITECTURE.md#portsrements. Ideally, they can be swapped against equivalent mechanisms without affecting the overall system function. External agencies should avoid direct access to infrastructural components, but there might be exceptions for efficiency's sake. Typically in this sector:

* External [libraries] and [frameworks] (always considered to be part of the infrastructure even if stored elsewhere and / or autoloaded)
* [Persistence] mechanisms (e.g. [database] platforms)
* Model / View / Presenter / Domain / Responder components of [MVC] / [MVP] / [ADR] architectures
* [Template engines] and templates
* Configuration data

#### Tests

The *Tests* sector holds all resources required for [testing your application] on various levels. In general, tests are nothing more than highly specialized clients of your application and must be granted full access to your ③ client and ② application layers. Test resources may also be accessed by external agencies (e.g. by extension) and typically consist of:

* Unit, integration, interface or acceptance test cases
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

* The top level directory `src` separates the actual source files from other package resources, e.g. documentation, configuration files, 3rd party libraries etc.
* `<Module>` must be replaced with a vendor-unique module name in [UpperCamelCase] writing (e.g. `MyApp`).
* The 3rd level is made up of five directories representing the main architectural tiers and sectors (see above).


### Application specifics

Inside the five main directories, your application may add additional structures as needed. However, to keep things consistent, I recommend sticking to these conventions:

* Directory and file names are always to be written in **UpperCamelCase**. I prefer using singular expressions wherever possible (i.e. `Factory` instead of `Factories`).
* If you have **multiple similar components, that are mostly used by external agents** (e.g. on a lower architectural level or by an external package), **keep them at a common central location**. As an example, I typically use directories named `Facade`, `Contract`, `Service` or `Factory` for grouping classes with similar functionality.
* **Keep closely related components together**. If you have, for instance, a class definition that implements an interface as described in [The Dependency Inversion Principle](#the-dependency-inversion-principle), put them into the same directory instead of spreading them across the file system. This rule commonly outweighs the previhttps://github.com/jkphl/generator-cleanphp/blob/master/doc/CLEAR-ARCHITECTURE.md#the-dependency-inversion-principleersonal taste in some situations though.
* If a lower architectural layer "mirrors" and extends the structure of a higher one, e.g. by providing concrete implementations of an interface defined on the higher level, **stick to the same directory and file names** as much as possible. This will help with keeping the cross-boundary relationships in mind.


## Rules & Conventions

<img src="https://cdn.rawgit.com/jkphl/generator-cleanphp/3306407b/doc/clear-architecture-dependency-rule.svg" alt="Clear Architecture tiers" align="right" width="50%"/>


### The Dependency Rule

In the Clear Architecture, source code dependencies may **only ever point to the same or an inward layer**.

> Nothing in an inner circle can know anything at all about something in an outer circle. In particular, the name of something declared in an outer circle must not be mentioned by the code in an inner circle. That includes, functions, classes, variables or any other named software entity. (*[Clean Architecture], Robert C. Martin*)

Valid inward cross-boundary dependencies include:

* Constructing instances of classes defined in enclosed layers
* Implementing interfaces, using traits etc.
* Calling functions and methods
* Using classes, interfaces etc. for [typing]

Strictly adhering to the Dependency Rule makes your application highly testable and very flexible in terms of implementation details (choice of a database platform or persistence strategy, client APIs etc.).


### The Dependency Inversion Principle

In order to not violate the Dependency Rule, the [Dependency Inversion Principle] must be used whenever complex data needs to be passed to an inward layer. Instead of expecting and directly referencing a lower-level component (e.g. as function parameter), a layer only provides and references an interface that needs to be implemented by the caller. This way, the conventional dependency relationship in inverted and the high-level layer doesn't depend on lower-level ones.

![Dependency inversion by using an interface / abstract service class](https://cdn.rawgit.com/jkphl/generator-cleanphp/4b0317a9/doc/clear-architecture-dependency-inversion.svg)


### Naming conventions

The following special components (including their files) must be named after their role:

* Interfaces must use the **`Interface`** suffix (e.g. `MyCustomInterface`)
* Traits must use the **`Trait`** suffix (e.g. `MyCustomTrait`)
* Factories must use the **`Factory`** suffix (e.g. `MyCustomFactory`). Public method names must use the `create` prefix (e.g. `createFromParams`).


## Considerations for PHP implementations

* Stick to commonly agreed coding style rules like [PSR-1] and [PSR-2]. Consider using code quality checkers like [Scrutinizer], [Code Climate] etc. Have a look at my [Clear Architecture Yeoman Generator] to speed up the installation and configuration of these tools.
* Apply [PSR-4] autoloading standards, with `<Module>` being the **base directory** corresponding to the **namespace prefix** (e.g. `Jkphl\MyApp`).
* Use [Composer] as the principal dependency manager for your projects. By default, Composer installs package dependencies into the top-level `vendor` directory and supports [PSR-4] compliant autoloading out of the box.
* When providing a CLI, check out Composer's [vendor binary features].
* For configuration purposes, I highly recommend the use of **environment variables** (respectively the excellent [phpdotenv] library or one of its equivalents).


## Recommendations

* [Clean Architecture in PHP] by Kristopher Wilson
* [Domain-Driven Design in PHP] by Carlos Buenosvinos et al.


[ADR]: http://pmjones.io/adr/
[Clean Architecture in PHP]: https://leanpub.com/cleanphp
[Clean Architecture]: https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html "The Clean Architecture by Bob Martin"
[Clear Architecture Yeoman Generator]: https://github.com/jkphl/generator-cleanphp
[CLI]: https://en.wikipedia.org/wiki/Command-line_interface
[Code Climate]: https://codeclimate.com/
[Composer]: https://getcomposer.org "Composer — Dependency Manager for PHP"
[Constant definitions]: https://en.wikipedia.org/wiki/Constant_%28computer_programming%29
[database]: https://en.wikipedia.org/wiki/Database
[Dependency Inversion Principle]: https://en.wikipedia.org/wiki/Dependency_inversion_principle
[Domain-Driven Design in PHP]: https://leanpub.com/ddd-in-php
[Domain-Driven Design]: https://en.wikipedia.org/wiki/Domain-driven_design
[Exceptions]: https://en.wikipedia.org/wiki/Exception_handling
[Facades]: https://en.wikipedia.org/wiki/Facade_pattern
[Factories]: https://en.wikipedia.org/wiki/Factory_%28object-oriented_programming%29
[FinTS]: https://en.wikipedia.org/wiki/FinTS
[frameworks]: https://en.wikipedia.org/wiki/Software_framework
[GUI]: https://en.wikipedia.org/wiki/Graphical_user_interface
[Hexagonal Architecture]: http://alistair.cockburn.us/Hexagonal+architecture
[Interfaces]: https://en.wikipedia.org/wiki/Protocol_%28object-oriented_programming%29
[libraries]: https://en.wikipedia.org/wiki/Library_%28computing%29
[MVC]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[MVP]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter
[Onion Architecture]: http://jeffreypalermo.com/blog/the-onion-architecture-part-1/
[Persistence]: https://en.wikipedia.org/wiki/Persistence_%28computer_science%29
[phpdotenv]: https://github.com/vlucas/phpdotenv
[PSR-1]: http://www.php-fig.org/psr/psr-1/
[PSR-2]: http://www.php-fig.org/psr/psr-2/
[PSR-4]: http://www.php-fig.org/psr/psr-4/
[REST API]: https://en.wikipedia.org/wiki/Representational_state_transfer
[Scrutinizer]: https://scrutinizer-ci.com/
[Template engines]: https://en.wikipedia.org/wiki/Template_processor
[testing your application]: https://en.wikipedia.org/wiki/Software_testing#Testing_levels
[typing]: https://en.wikipedia.org/wiki/Type_system
[UpperCamelCase]: https://en.wikipedia.org/wiki/Camel_case
[vendor binary features]: https://getcomposer.org/doc/articles/vendor-binaries.md
