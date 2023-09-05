---
title: "Integration testing on Spring Boot GraphQL Starter with HttpGraphQlTester"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Java
  - GraphQL
  - HttpGraphQlTester
  - Spring Boot
  - Spring Boot GraphQL Starter
---

Following my [previous article on getting started with spring and graphql](https://www.thecodinganalyst.com/tutorial/Spring-boot-application-getting-started/), let's look at how we can do integration testing on the graphql spring boot application.

You might wonder if it is necessary to do integration testing, can I just leave it to the unit tests? Well, for a simple controller like the one below, there is really nothing much to unit test on. 

```
@QueryMapping
public Flux<Product> getAllProducts(){
    return productService.getAllProducts();
}
```

We will definitely need to have unit tests for our service layer, but having integration test on our controller helps us to ensure we have implemented our `schema.graphqls` correctly, and we have passed on our error message to the graphql, so that it will not just give an "Internal Error". We shall discuss more on that in the next article.

To do integration testing on all the layers (controller, service, repository), we need the full setup to run the test, hence we need to enable `@SpringBootTest`. Spring for GraphQL provides a [GraphQlTester](https://docs.spring.io/spring-graphql/docs/current/reference/html/#testing.graphqltester) interface for testing the graphql queries. And it has different implementations for different ways how we use it, via Http, Websockets, or RSockets. Since we are creating http api, we will implement the `HttpGraphQlTester`. To autowire the `HttpGraphQlTester` into our test class, we can simply add the `@AutoConfigureHttpGraphQlTester` annotation to our class, and declare the `HttpGraphQlTester` by annotating it with the `@Autowired` annotation. 

```
@SpringBootTest
@AutoConfigureHttpGraphQlTester
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@Testcontainers
@ContextConfiguration(classes = TestIntroGraphQlApplication.class)
class ProductControllerIntegrationTest {

    @Autowired
    HttpGraphQlTester httpGraphQlTester;

}
```

> The `@TestInstance` is added because in my case, I want to create a single instance to run all the test functions in the class instead of the default one instance per function, so that I can make use of the data created in my first few functions for my subsequent tests. And the `TestMethodOrder` allows me to specify the order of the tests to run. 

> The `@Testcontainers` indicates that this function will start the docker service to create the database for our tests, and the `@ContextConfiguration` tells spring where to get the configurations, which in my case, contains the configurations of the testcontainer.

The [GraphQlTester](https://github.com/spring-projects/spring-graphql/blob/main/spring-graphql-test/src/main/java/org/springframework/graphql/test/tester/GraphQlTester.java) contains a list of interfaces to help with testing. 

![graphqltester interfaces](/assets/images/2023/09/graphqltester.png)

For a mutation mapping function in the controller, like the function below, we are passing in a `Product` in the parameter, and it should return a Mono of the created `Product`.

```
@MutationMapping
public Mono<Product> createProduct(@Argument Product product){
    log.info("createProduct: " + product.toString());
    return productService.createProduct(product);
}
```

A simple integration test on implementing graphql on the controller will look something like this.

```
@Test
@Order(1)
void createProduct() {
    Product product = this.httpGraphQlTester
            .document("""
                        mutation {
                            createProduct(product: {
                                name: "Pilot Pen",
                                description: "Gel-based ball point pen",
                                price: 1.50,
                                category: "Stationery"
                            }){
                                id
                                name
                                description
                                price
                                category
                            }
                        }
                    """)
            .execute()
            .errors()
            .verify()
            .path("createProduct")
            .entity(Product.class)
            .get();
    assertThat(product.getId()).isNotNull();
    assertThat(product.getName()).isEqualTo("Pilot Pen");
    assertThat(product.getDescription()).isEqualTo("Gel-based ball point pen");
    assertThat(product.getPrice()).isEqualTo(BigDecimal.valueOf(1.50));
    assertThat(product.getCategory()).isEqualTo("Stationery");
    savedProduct = product;
}
```

Running it in our `graphiql` should return the following result.

```
{
  "data": {
    "createProduct": {
      "id": "64f68c13aca30273b0f87534",
      "name": "Pilot Pen",
      "description": "Gel-based ball point pen",
      "price": 1.5,
      "category": "Stationery"
    }
  }
}
```

In the function, we pass in our graphql syntax to our `httpGraphQlTester` with the `document()` function, and pipe it to `execute()` to get it executed. After executing the graphql, we can pipe it to `errors()` and `verify()` to verify that there are no errors in the execution. So if there are any errors from the execution, the function will fail. Then we pipe it to `path("createProduct")` to navigate to the `createProduct` json node in the result, which should be returning us a `Product`, and we use the `get()` function to get it mapped our `Product` class, and we assigned it to the `product` variable in our function. Then we can use our usual assertion functions to verify the return values against what we expect.

A working copy of the above code is available on [https://github.com/thecodinganalyst/graphql/blob/main/src/test/java/com/hevlar/intro/graphql/controller/ProductControllerIntegrationTest.java](https://github.com/thecodinganalyst/graphql/blob/main/src/test/java/com/hevlar/intro/graphql/controller/ProductControllerIntegrationTest.java).

