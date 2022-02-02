---
title: "How to create generic JPA repository"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Kotlin
  - Generics
  - JPA
---

Generics makes it possible to reuse code, which is great. But when it needs to apply on a JPA repository, it can get messy. As any small mistake is going to generate error messages which doesn't really tell you what is the root cause. The thing with JPA repositories, is that it has to be applied on actual concrete classes that are marked with the `@Entity` annotation. But you shouldn't be making your abstract classes and interfaces as entities. 

A few rules i have gathered to make this possible.

1. Make any class that uses a generic field to be a generic that extends the class.
2. Keep your package that describes the interfaces out of the the implementation package.


For example, if I have a generic class - `Item<I>` where `I` is going to be the data type for the `@Id` of `Item`, then my repository has to be declared with 2 generics - `I` and `T: Item<I>`. So my signature for the `ItemRepository` will be `interface ItemRepository<I: Any, T: Item<I>>: JpaRepository<T, I>`. There shouldn't be any `Item<I>` type in either the repository or service, except for the generics declaration in the above mentioned line. 

Next, keep your generic models, repositories, and services in a different package from your implementation, and have your `@SpringBoot` class in the implementation package only. Not doing so will result in Spring trying to initialize the `ItemRepository`, which will result in an error as `ItemRepository` has `Item` as the entity, which is not a managed type. The error message will be `Error creating bean with name 'itemRepository' defined in com.example.genericjpa.logic.repository.ItemRepository defined in @EnableJpaRepositories declared on JpaRepositoriesRegistrar.EnableJpaRepositoriesConfiguration: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: Not a managed type: interface com.example.genericjpa.logic.model.Item`

A working example is available at [https://github.com/thecodinganalyst/GenericJPA](https://github.com/thecodinganalyst/GenericJPA).