---
title: "How to handle internal error in Spring for GraphQL"
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

This is an extension to the articles - [`Getting started with Spring and GraphQl`](https://www.thecodinganalyst.com/tutorial/Getting-started-with-spring-and-graphql/) and [`Integration testing on Spring Boot GraphQL Starter with HttpGraphQlTester`](https://www.thecodinganalyst.com/tutorial/Integration-testing-on-Spring-Boot-GraphQL-Starter-with-HttpGraphQlTester/). 

GraphQL pretty much handles validation issues for us, like the screenshot below. We missed out a required field, and graphql provides the appropriate message for us automatically.

![validation error](/assets/images/2023/09/graphiql-validation-error.png)

However, if the exception is due to the internal execution to do the mutation or query of the data, graphql will only give an `Internal Error`, without much information. 

![internal error](/assets/images/2023/09/graphiql-internal-error.png)

For example, in the update function below, the program will first get the repository to find the product, before updating the product with the fields from the `updatedProduct` and save it back in the repository. 

```
public Mono<Product> updateProduct(String id, Product updatedProduct){
    return productRepository.findById(id)
            .switchIfEmpty(Mono.error(new IllegalArgumentException("Product not found")))
            .flatMap(product -> {
                product.setName(updatedProduct.getName());
                product.setDescription(updatedProduct.getDescription());
                product.setPrice(updatedProduct.getPrice());
                product.setCategory(updatedProduct.getCategory());
                return productRepository.save(product);
            });
}
```

If the product id is not found in the repository, it should return a Mono error, encapsulating an IllegalArgumentException with the appropriate error message - `Product not found`. 

However, this information is not passed on to the graphql, like what we saw in the screenshot earlier. The message is intentionally opaque to avoid leaking implementation details. Nevertheless, we can handle this error such that it can return the appropriate message to the client. 

To handle the exceptions for only with the specific controller itself, we can add the handle function with the `@GraphQlExceptionHandler`, as described in the [documentation](https://docs.spring.io/spring-graphql/docs/current/reference/html/#controllers.exception-handler).

```
@Controller
@Slf4j
public class ProductController {

    ...

    @GraphQlExceptionHandler
    public GraphQLError handle(@NonNull Throwable ex, @NonNull DataFetchingEnvironment environment){
        return GraphQLError
                .newError()
                .errorType(ErrorType.BAD_REQUEST)
                .message(ex.getMessage())
                .path(environment.getExecutionStepInfo().getPath())
                .location(environment.getField().getSourceLocation())
                .build();
    }
}
```

In the above function, we got the Throwable and DataFetchingEnvironment to return the error message and the path and location of where the exception occurred, returning a `GraphQLError` to the client. 

![graphiql error message](/assets/images/2023/09/graphiql-error-message.png)

Running the same function again, now we get a more meaningful message. Do note that for the above method, it only handles exceptions from the `ProductController` class. 

Alternatively, we can also create an extension of the `DataFetcherExceptionResolverAdapter` as a fallback to handle exceptions not caught by the respective controllers, as described in the [documentation](https://docs.spring.io/spring-graphql/docs/current/reference/html/#execution.exceptions).

```
@Component
public class CustomExceptionResolver extends DataFetcherExceptionResolverAdapter {
    @Override
    protected GraphQLError resolveToSingleError(@NonNull Throwable ex, @NonNull DataFetchingEnvironment env){
        return GraphqlErrorBuilder.newError()
                .errorType(ErrorType.BAD_REQUEST)
                .message(ex.getMessage())
                .path(env.getExecutionStepInfo().getPath())
                .location(env.getField().getSourceLocation())
                .build();
    }
}
```

We can test that it works in the following integration test. Here, after executing the graphql, we call the `errors()` and `expect()` to check on the error message. Without the above implementation, the test will fail because it would get in Internal Error instead.

```
@Test
@Order(5)
void updateProduct_invalidProduct(){
    this.httpGraphQlTester
            .document("""
                    mutation {
                        updateProduct(
                            id: "123",
                            updatedProduct: {
                                name: "Pilot G1 Pen",
                                description: "Best selling gel-based ball point pen",
                                price: 1.80
                                category: "Stationery"
                            }
                        ){
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
            .expect(error -> Objects.equals(error.getMessage(), "Product not found"));
}
```

A working copy of the above code is available on [https://github.com/thecodinganalyst/graphql/blob/main/src/main/java/com/hevlar/intro/graphql/controller/ProductController.java](https://github.com/thecodinganalyst/graphql/blob/main/src/main/java/com/hevlar/intro/graphql/controller/ProductController.java).
