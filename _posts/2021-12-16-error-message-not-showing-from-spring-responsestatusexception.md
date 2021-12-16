---
title: "Error message not showing from Spring ResponseStatusException"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Java
  - Spring
---

A little discovery I found recently. My error messages are not showing as intended. 

```
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
fun createAccount(@RequestBody account: Account): Account {
    if(accountService.existAccount(account.id))
        throw ResponseStatusException(HttpStatus.CONFLICT, "Account - ${account.id} already exists")

    return try{
        accountService.saveAccount(account)

    } catch (e: Exception) {
        throw ResponseStatusException(HttpStatus.BAD_REQUEST, "Invalid account - ${account.id}")
    }
}
```

The result is

```
{
    "timestamp": "2021-12-16T01:52:23.099+00:00",
    "status": 409,
    "error": "Conflict",
    "path": "/accounts"
}
```

My error message - `Account - ${account.id} already exists` doesn't show up as intended.


So turns out that since Spring 2.3, error messages will not show by default, in order to reduce risks of leaking client information. (Source)[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes#changes-to-the-default-error-pages-content].

So in order to make the error message appear again, the following line is to be included in the `application.properties` file - 
`server.error.include-message=always`. In case the application.properties file isn't created yet, do create it in `src\main\resources`. 

So after restarting, I will get the error message I wanted.

```
{
    "timestamp": "2021-12-16T01:58:27.722+00:00",
    "status": 409,
    "error": "Conflict",
    "message": "Account - CASH already exists",
    "path": "/accounts"
}
```



