---
title: "Difference between getById and findById in Spring JPA"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Java
  - Kotlin
  - Spring
  - JPA
  - ImportanceOfDocumentation
  - ImportanceOfGoodNaming
---

Use getById() if you just want to get a reference of the entity, like assigning it to a property of another entity. 

Use findById() if you actually want to get the entity. 

Because I tried to use getById() to retrieve an entity, but as it uses lazy loading, I got a `org.hibernate.LazyInitializationException: could not initialize proxy`. 

```kotlin
fun getAccount(accountId: String): Account{
  return accountRepository.getById(accountId)
}
```

So switching to findById() resolves the issue immediately

```kotlin
fun getAccount(accountId: String): Account{
  return accountRepository.findById(accountId).orElseThrow()
}
```

Initially, I thought it is the same method just that getById() returns an object, but findById() returns an Optional object. But thanks to this [article](https://www.javacodemonk.com/difference-between-getone-and-findbyid-in-spring-data-jpa-3a96c3ff), now I know that getById() is **only useful when access to properties of object is not required**. 