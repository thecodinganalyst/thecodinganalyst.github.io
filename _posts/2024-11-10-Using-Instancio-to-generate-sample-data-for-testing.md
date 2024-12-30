---
title: "Using Instancio to generate sample date for testing"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Spring Boot
  - Instancio
  - Testing
---

Testing is a critical component of software development, ensuring that applications behave as expected. In Spring Boot projects, creating robust and maintainable tests often requires generating realistic test data. Manually crafting this data can be time-consuming and error-prone. This is where Instancio shines. Instancio is a Java library designed to simplify the process of generating test data with minimal configuration, enabling developers to focus on writing effective tests.

Instancio is particularly a life-saver when dealing with objects that have a large number of fields. For example, consider a Customer class with over 20 fields, including nested objects. Manually mocking such an object can quickly become cumbersome and error-prone. With Instancio, you can generate such objects with ease:

```
@Test
public void generateComplexObject() {
    Customer customer = Instancio.create(Customer.class);
    System.out.println(customer);
}
```

This simple code snippet creates an instance of Customer with all its fields populated, including nested objects, eliminating the need for manual setup.

<!--more-->

## What is Instancio?

Instancio is a flexible and powerful library that automates the creation of Java objects with populated fields. It supports complex object graphs and works seamlessly with both primitive and custom types. You can generate realistic test data with just a few lines of code, making it an excellent tool for unit and integration testing in Spring Boot.

## Key Features of Instancio

- Automatic Object Creation: Instancio automatically creates objects and populates their fields.

- Customizable Rules: You can specify generation rules for specific fields or types.

- Support for Nested Objects: Generate complex object graphs effortlessly.

- Integration with JUnit and TestNG: Easily integrate Instancio into your test framework.

- Fluent API: The library offers a user-friendly API for customizing object creation.

## Setting Up Instancio in a Spring Boot Project

To start using Instancio, add the following dependency to your pom.xml file if you're using Maven:

```
<dependency>
    <groupId>org.instancio</groupId>
    <artifactId>instancio-core</artifactId>
    <version>2.0.0</version>
    <scope>test</scope>
</dependency>
```

For Gradle, add:

```
testImplementation 'org.instancio:instancio-core:2.0.0'
```

## Basic Usage

### Generating a Simple Object

Suppose you have a User class with fields like id, name, and email. You can use Instancio to generate a User instance as follows:

```
import org.instancio.Instancio;

@Test
public void generateSimpleObject() {
    User user = Instancio.create(User.class);
    System.out.println(user);
}
```

Instancio automatically populates the fields of the User class with realistic values, such as random strings for name and email.

### Generating a List of Objects

To generate a collection of objects, use the Instancio.ofList() method:

```
@Test
public void generateListOfObjects() {
    List<User> users = Instancio.ofList(User.class).size(10).create();
    users.forEach(System.out::println);
}
```

This creates a list of 10 User objects with populated fields.

### Customizing Object Generation

Setting Specific Values

You can customize object generation by setting specific values for certain fields:

```
@Test
public void customizeFieldGeneration() {
    User user = Instancio.of(User.class)
            .set(field(User::getName), "John Doe")
            .set(field(User::getEmail), "john.doe@example.com")
            .create();
    System.out.println(user);
}
```

### Ignoring Fields

If you want to exclude certain fields from being populated, you can do so with the ignore() method:

```
@Test
public void ignoreFields() {
    User user = Instancio.of(User.class)
            .ignore(field(User::getPassword))
            .create();
    System.out.println(user);
}
```

### Generating a Past Date

Instancio provides a generate() method that allows for advanced customizations, such as generating dates in the past. For example:

```
@Test
public void generatePastDate() {
    User user = Instancio.of(User.class)
            .generate(field(User::getBirthDate), gen -> gen.temporal().localDate().past(LocalDate.now().minusYears(1)))
            .create();
    System.out.println(user.getBirthDate());
}
```

In this example, the birthDate field is generated as a date from the past year.

### Generating a String in Microsoft JSON Date Format

To generate a string representing a date in the Microsoft JSON date format, you can use the generate() method with a custom generator:

```
@Test
public void generateMicrosoftJsonDateFormat() {
    User user = Instancio.of(User.class)
            .generate(field(User::getDateString), gen -> gen.string().from(() -> {
                Date date = Date.from(LocalDate.now().atStartOfDay(ZoneId.systemDefault()).toInstant());
                return "/Date(" + date.getTime() + ")/";
            }))
            .create();
    System.out.println(user.getDateString());
}
```

In this example, the dateString field is generated as a string in the Microsoft JSON date format (e.g., /Date(1672531200000)/).

## Using Instancio in Spring Boot Tests

### Integration with Spring Boot

Instancio can be used in unit and integration tests in a Spring Boot project. Here is an example using Instancio with Spring Boot’s testing framework:

```
@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void testCreateUser() {
        User user = Instancio.of(User.class)
                .set(field(User::getName), "Jane Doe")
                .set(field(User::getEmail), "jane.doe@example.com")
                .create();

        userService.save(user);

        User retrievedUser = userService.findById(user.getId());
        assertEquals("Jane Doe", retrievedUser.getName());
        assertEquals("jane.doe@example.com", retrievedUser.getEmail());
    }
}
```

### Mocking with Instancio

When writing unit tests with mocking frameworks like Mockito, you can use Instancio to generate mock data:

```
@Test
public void testWithMockedRepository() {
    User user = Instancio.create(User.class);

    when(userRepository.save(any(User.class))).thenReturn(user);

    User savedUser = userService.save(new User());
    assertNotNull(savedUser);
}
```

### Advantages of Using Instancio

- Time-Saving: Automatically generating test data saves time and effort.

- Improved Test Quality: Realistic and diverse data leads to better test coverage.

- Flexibility: Customization options allow you to control the generated data.

- Ease of Use: The fluent API makes Instancio easy to integrate and use.

## Conclusion

Instancio is a powerful tool for generating test data in Spring Boot projects. By automating object creation, it reduces the time and effort needed to write effective tests, allowing developers to focus on building robust applications. Whether you’re writing unit tests or integration tests, Instancio can streamline your testing process and enhance the quality of your tests.