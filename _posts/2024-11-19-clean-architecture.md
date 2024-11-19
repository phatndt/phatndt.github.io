---
title: Clean architecture
date: 2024-11-19 20:30:00
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [architecture]     # TAG names should always be lowercase
---

# Clean Architecture Basic

![img-description](https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

## 1. What is Clean Architecture?

Clean Architecture is software development philosophy that provides developers with a way to organize codes base on key princiles.

## 2. Key principles of Clean Architecture

- **Seperate of concerns**: Each layer has its responsibility, reducing dependencies between parts of code.
- **Dependency rule**: This rule says that source code dependencies can only points inwards. Nothing in an inner circle can know anything all about something in an outer circle. In particular, the name of something declared in an outer circle must not be mentioned by the code in the an inner circle. That includes, functions, classes. variables, or any other named software entity.
- **Testability**: The separation allows for independent unit testing of each layer.
- **Maintainability**: By isolating layers, code changes are less likely to introduce bugs in unrelated parts of the application.

## 3. Key layers in Clean Architecture

- **Entities**: encapsulate business logic. This can be an objects, or a set of data structures and fucntions. The entities could be used by many different applications.

- **Use case**: contain application specific business rules. It encapsulates and implements all of the use cases of the system.

- **Interface adapter**: set of adapters that convert data from the format most convenient for the use cases and entities, to the format most convenient for some external agency, such as Database or UI. It also contains any other adapter necessary to convert data from some external form, such as an external service, to the internal form used by the use cases and entities.

- **Frameworks and Drivers**: compose of frameworks and tools such as Database, Web, Android,...

## 4. Pros and Cons of Clean Architecture

### 4.1 Pros:

1. **Separation of Concerns, single of responsibility**.

2. **Testability**: each layer is decouples, allow isolating testing

3. **Reusability**: core business logic can be reused across platforms.

4. **Maintainable and Scalability**: each layer is responsible for specific tasks, making it easier to locate and fix issues or add features, so support large-scales projects.

### 4.2 Cons:

1. **Increased Complexity**
Drawback: For small projects, the additional layers and abstractions may feel excessive.
Example: Setting up Use Cases, interfaces, and multiple layers can seem over-engineered for a simple app.
2. **Steeper Learning Curve**
Drawback: Developers new to Clean Architecture may find it challenging to grasp and implement correctly.
Example: Understanding dependency inversion and layer boundaries requires time and practice.
3. **Boilerplate Code**
Drawback: Writing interfaces, abstractions, and data mapping can lead to repetitive code.
Example: Creating repository interfaces and their implementations for every data source can feel redundant.
4. **Slower Initial Development**
Drawback: Setting up the architecture takes time, which can slow down initial development.
Example: Simple features require coordination between layers, adding overhead.
5. **Overhead for Small Projects**
Drawback: Clean Architecture is often overkill for small or one-off applications.
Example: A simple to-do app might not need this level of modularity and abstraction.
6. **Requires Discipline**
Drawback: Maintaining separation of concerns and adhering to architectural rules requires consistent effort.
Example: Developers may unintentionally bypass layers, leading to tightly coupled code.

## 5. When using Clean Architecture

- Large-scale, long-term projects with complex requirements.

- Applications requiring frequent updates and scaling.

- Projects needing high test coverage and modularity.
