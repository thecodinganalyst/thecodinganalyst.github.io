---
title: "How to create custom validation annotation"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Boot
  - Spring Web
  - Validation
---

When we have an application that collects inputs from users, we usually need to have some validation checks to ensure user inputs are within what is expected. Example like it should hava some minimum length, it should be a future date, it should not be blank or empty, etc. Writing such code is one of the most mundane tasks one can get, but we can get away with it by using the constraint annotations provided by [jakarta validation](https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary.html). For more details on how to use the constraints provided, do refer to my previous article - [How to validate input in Spring Boot RestController](https://www.thecodinganalyst.com/tutorial/how-to-validate-input-in-spring-boot-restcontroller/).

If the readily available constraints doesn't have what we need, we can create custom validations and use it like how we use the constraints available.

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

In the above example, you can see the @ValidPassword is the custom constraint that we created. Let's have a look at the code.

```
@Documented
@Target({TYPE, FIELD, PARAMETER, RECORD_COMPONENT})
@Constraint(validatedBy = CustomPasswordValidator.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidPassword {
    String message() default "Password must be at least 8 characters long, contains at least 1 upper case letter, 1 lower case letter, and 1 digit";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

We create an annotation by using `@interface` instead of the usual `class`. The `Target` annotation specifies that this annotation can be used for types (i.e. classes), fields, parameters, and record components. The `@Retention` annotation specifies that the annotation will be retained during runtime. We also annotate it with `@Constraint` with the parameter `validatedBy` specifying our validation class - CustomPasswordValidator, which we are going to create.

```
public class CustomPasswordValidator implements ConstraintValidator<ValidPassword, String> {

    private static final String PASSWORD_PATTERN = "^(?=.*[A-Z])(?=.*[a-z])(?=.*[0-9])(?=.*[^a-zA-Z0-9]).{8,}$";

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return Pattern.compile(PASSWORD_PATTERN)
                .matcher(value)
                .matches();
    }
}
```

This class needs to implement `ConstraintValidator` interface, with 2 generic paramters. The first one is the annotation class that will use this, and the second one is the type of input it will be validating. The `isValid` function is the one which we can customize to return a boolean value indicating if the validation will pass. Here we are using regular expression to validate the string. 

