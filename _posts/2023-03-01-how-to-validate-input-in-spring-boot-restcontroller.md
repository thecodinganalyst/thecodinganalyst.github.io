---
title: "How to validate input in Spring Boot RestController"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Boot
  - Spring Web
  - Validation
---

Validating user inputs is not a difficult task, but it is rather cumbersome. Although essential, having lots of validation checks in the code makes the code hard to read. An easier way to do it is to use [Jakarta Bean Validation](https://beanvalidation.org/) that is included with Spring Boot Starter Validation.

If it is not already included in the `build.gradle`, add this line to the dependencies.

```
implementation 'org.springframework.boot:spring-boot-starter-validation:3.0.2'
```

Continuing the [forum project](https://github.com/thecodinganalyst/forum) which we have been working on since [the last article on configuring the username and password authentication](https://www.thecodinganalyst.com/tutorial/how-to-configure-access-management-in-spring-security/), we create a new `ForumUserController`  to allow registration of new users. The function `registerUser` is a POST request which will accept a `UserRegistrationDto` containing the information needed to register a new user, and return a `UserRoleDto`.

```
@RestController
@RequestMapping("/api/v1/users")
public class ForumUserController {

    ForumUserService userService;

    public ForumUserController(ForumUserService userService){
        this.userService = userService;
    }

    @PostMapping("/registerUser")
    public UserRoleDto registerUser(@RequestBody @Valid UserRegistrationDto userRegistrationDto) {
        try {
            ForumUser user = this.userService.registerUser(userRegistrationDto);
            return ForumUserToUserRoleMapper.map(user);
        }catch (Exception e){
            throw new ResponseStatusException(HttpStatus.CONFLICT, e.getMessage());
        }
    }
}
```

So as we need to validate the `UserRegistrationDto`, we shall add the `@Valid` annotation to the parameter of the function. Then we need to annotate the class which we want to validate. Jakarta Bean Validation provides a number of predefined validations - @NotNull, @NotEmpty, @Size, etc. The whole list of constraints available are available at [https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary.html](https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary.html).


```
@PasswordMatching(message = "Passwords do not match")
public record UserRegistrationDto(

        @NotNull(message = "User id cannot be null")
        @NotEmpty(message = "User id cannot be empty")
        String userId,

        @NotNull(message = "Given name cannot be null")
        @NotEmpty(message = "Given name cannot be empty")
        String givenName,

        @NotNull(message = "Family name cannot be null")
        @NotEmpty(message = "Family name cannot be empty")
        String familyName,

        @ValidEmail(message = "Email is not valid")
        @NotNull(message = "Email cannot be null")
        @NotEmpty(message = "Email cannot be empty")
        String email,

        @NotNull(message = "Password cannot be null")
        @NotEmpty(message = "Password cannot be empty")
        @Size(min = 8, message = "Password must be at least 8 characters long")
        @ValidPassword
        String password,

        @NotNull(message = "Matching password cannot be null")
        @NotEmpty(message = "Matching password cannot be empty")
        String matchingPassword
) {
}
```

> We can also create our own customized annotations. The `@PasswordMatching`, `@ValidPassword`, and `@ValidEmail` in the below code are our custom validations which I will explain in my next article. 

Do note that the validation exceptions will not be caught by our function in the RestController, as the parameters are filtered before it even reaches our RestController function. In order to handle the validation exceptions, we have to add a function with the `@ExceptionHandler` annotation in our RestController.

```
@RestController
@RequestMapping("/api/v1/users")
public class ForumUserController {

    ...

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public List<ErrorDto> handleValidationExceptions(MethodArgumentNotValidException ex){
        return ex.getBindingResult()
                .getAllErrors()
                .stream()
                .map(ForumUserController::getErrorDto)
                .toList();
    }

    private static ErrorDto getErrorDto(ObjectError error) {
        String field = error instanceof FieldError fieldError ? fieldError.getField() : null;
        String rejectedValue = error instanceof FieldError fieldError ? String.valueOf(fieldError.getRejectedValue()) : null;

        return new ErrorDto(
                error.getObjectName(),
                field,
                rejectedValue,
                error.getDefaultMessage());
    }
}
```

In the above code, the `handleValidationException` method will catch any `MethodArgumentNotValidException`, and return a list of self-defined `ErrorDto`s as the result, with a status of 400 (BAD REQUEST).

The `ErrorDto` is defined as such

```
public record ErrorDto(
        String objectName,
        String fieldName,
        String rejectedValue,
        String errorMessage
) {}
```

The `getAllErrors()` in the `handleValidationExceptions` method will return a list of `ObjectError`. Of which, those validation exceptions for the fields (i.e. the userId, givenName, familyName, email, etc) will be `FieldError`. `FieldError` inherits `ObjectError`, and contains information on the rejectedValue and fieldName. So if it is a field error, our `getErrorDto` will cast the ObjectError to FieldError, so that it can get more detailed information about the field and value that causes the exception.

To test that the validation is working, let's pass an empty string to all the fields, and run it with mockMvc. It should return a status 404 (BAD Request), and the list of ErrorDto as the result.

```
@Test
void whenRegisterUserWithEmptyFields_shouldFail() throws Exception {
    // given
    UserRegistrationDto userRegistrationDto = new UserRegistrationDto(
            "",
            "",
            "",
            "",
            "",
            ""
    );
    String userRegistrationDtoJson = objectMapper.writeValueAsString(userRegistrationDto);

    // when
    MvcResult result = mockMvc.perform(post("/api/v1/users/registerUser")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(userRegistrationDtoJson).with(csrf()))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.length()").value(9))
            .andReturn();

    // then
    String jsonResult = result.getResponse().getContentAsString();
    List<ErrorDto> errorDtos = Arrays.stream(objectMapper.readValue(jsonResult, ErrorDto[].class)).toList();

    List<ErrorDto> expectedErrorDtos = List.of(
            new ErrorDto("userRegistrationDto", "userId", "", "User id cannot be empty"),
            new ErrorDto("userRegistrationDto", "givenName", "", "Given name cannot be empty"),
            new ErrorDto("userRegistrationDto", "familyName", "", "Family name cannot be empty"),
            new ErrorDto("userRegistrationDto", "email", "", "Email is not valid"),
            new ErrorDto("userRegistrationDto", "email", "", "Email cannot be empty"),
            new ErrorDto("userRegistrationDto", "password", "", "Password cannot be empty"),
            new ErrorDto("userRegistrationDto", "matchingPassword", "", "Matching password cannot be empty"),
            new ErrorDto("userRegistrationDto", "password", "", "Password must be at least 8 characters long"),
            new ErrorDto("userRegistrationDto", "password", "", "Password must be at least 8 characters long, contains at least 1 upper case letter, 1 lower case letter, and 1 digit")
    );

    assertThat(errorDtos, containsInAnyOrder(expectedErrorDtos.toArray()));
}
```

A full working example of the above code sample is available on my github repository - [https://github.com/thecodinganalyst/forum](https://github.com/thecodinganalyst/forum)

This is part of a series illustrating how to build a backend Spring boot application.
- [Getting Started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/)
- [Deploying to Docker](https://thecodinganalyst.github.io/tutorial/Deploying-mult-container-application-to-docker/)
- [Spring Data Testing](https://thecodinganalyst.github.io/tutorial/how-to-test-spring-data-repository/)
- [Testing Services](https://thecodinganalyst.github.io/tutorial/how-to-test-services-in-a-spring-boot-application/)
- [Unit Testing of Controller](https://thecodinganalyst.github.io/tutorial/how-to-unit-test-rest-controller-in-a-spring-boot-application/)
- [Integration Testing](https://thecodinganalyst.github.io/knowledgebase/how-to-do-integration-testing-in-spring-boot-rest-application/)
- [Code quality review with Sonarqube](https://www.thecodinganalyst.com/tutorial/integrate-code-quality-review-with-sonarqube/)
- [Configure Spring Security CSRF for testing on Swagger](https://www.thecodinganalyst.com/tutorial/Configure-spring-security-csrf-for-testing-on-swagger/)
- [Configure Access Management in Spring Security](https://www.thecodinganalyst.com/tutorial/how-to-configure-access-management-in-spring-security/)
- [Validate inputs in Spring Boot RestController](https://www.thecodinganalyst.com/tutorial/how-to-validate-input-in-spring-boot-restcontroller/)