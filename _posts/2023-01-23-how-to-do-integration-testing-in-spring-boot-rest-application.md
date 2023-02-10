---
title: "How to do integration testing in Spring Boot Rest application"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Boot
  - Spring Testing
  - Integration Testing
---

Unlike unit tests where we should be isolating the test to just the codes in the function we are testing, integration tests is meant to test the full spectrum of dependencies of the function. In the case of testing the rest api of our spring boot application, we are testing the results from the combination of the controller, service, and repository, to ensure that the result is what we are expecting. To do that, we will need to integrate all the components to work together.

We can do that by simply annotating our integration test class with the `@SpringBootTest` annotation. And to be able to autowire a mock MVC, we add the `@AutoConfigureMockMvc` annotation. We also autowire the `ObjectMapper` so that we can create json string of our model class easily.

```
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@DirtiesContext
public class PostControllerIntegrationTest {
    @Autowired
    MockMvc mvc;

    @Autowired
    ObjectMapper objectMapper;
}
```

A dilema with integration testing is that we shouldn't mock any of our components, so that we can have a complete uninterrupted scenario for our test environment, but that will not allow us to add data and run individual test function to test the outcome when initial data is required. For example, in order to test the update function, the data to be updated must already exist. So we can't run the test for the update function without first running the test for the create function in the same test. We can't run just the test function, but we can run the whole test class and that will work. Except that the individual test functions are executed in random order, or usually by alphabetical order. That will not work for us if our update test function is alphabetically before our create test function. So, in order to arrange the order of the functions to execute, we add the `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)` to our class, then add the `@Order(1)` annotation to our function to let Spring Testing the order for our tests.

```
@Test
@Order(1)
public void whenCreatePost_thenReturnStatusCreatedAndPost() throws Exception {

@Test
@Order(2)
public void whenListPost_thenReturnPostList() throws Exception {
```

Next, because we want to use the `@BeforeAll` annotation to create a `Topic` before running all the functions in this `PostControllerIntegrationTest`, so that all the test functions can use it and we don't want to create the `Topic` every time before a function, we have to add the `@TestInstance(TestInstance.Lifecycle.PER_CLASS)` to have 1 instance of the test class to run all the tests instead of creating an instance for each method, which is the default. Without this, having the `@BeforeAll` will give a compile error.

```
@BeforeAll
public void setup() throws Exception {
    topic = Topic.builder().topicId(1L).title("Topic").build();
    MvcResult result = mvc.perform(post("/api/v1/topics").contentType(MediaType.APPLICATION_JSON).content(objectMapper.writeValueAsString(topic)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.topicId").value(1))
            .andReturn();
    topic = objectMapper.readValue(result.getResponse().getContentAsString(), Topic.class);
}
```

The `DirtiesContext` annotation applied to the test class will create a totally new application context after finish running the test class, so that the database will be empty. This is needed because we are creating the same `Topic` instance in the 2 different test classes, we don't want our tests to have data from the results of running other tests. 

Lastly, we can write our test functions just like how we write for our [unit tests](https://thecodinganalyst.github.io/knowledgebase/how-to-unit-test-rest-controller-in-a-spring-boot-application/) by using the MockMvc to run the http request and asset the results, sans the mocking. 

```
@Test
@Order(1)
public void whenPostTopic_thenReturnStatusCreatedAndTopic() throws Exception {
    Topic topic = new Topic("Topic");
    mvc.perform(post(topicsEndpoint).contentType(MediaType.APPLICATION_JSON).content(objectMapper.writeValueAsString(topic)))
            .andExpect(status().isCreated())
            .andExpect(content().contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.title").value("Topic"))
            .andExpect(jsonPath("$.created").isNotEmpty())
            .andExpect(jsonPath("$.updated").isNotEmpty());
}
```

The above working example is available on my [github repo](https://github.com/thecodinganalyst/forum/blob/initial-sample/src/test/java/com/hevlar/forum/controller/PostControllerIntegrationTest.java). 

This is part of a series illustrating how to build a backend Spring boot application.
- [Getting Started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/)
- [Deploying to Docker](https://thecodinganalyst.github.io/tutorial/Deploying-mult-container-application-to-docker/)
- [Spring Data Testing](https://thecodinganalyst.github.io/tutorial/how-to-test-spring-data-repository/)
- [Testing Services](https://thecodinganalyst.github.io/tutorial/how-to-test-services-in-a-spring-boot-application/)
- [Unit Testing of Controller](https://thecodinganalyst.github.io/tutorial/how-to-unit-test-rest-controller-in-a-spring-boot-application/)
- [Integration Testing](https://thecodinganalyst.github.io/knowledgebase/how-to-do-integration-testing-in-spring-boot-rest-application/)
- [Code quality review with Sonarqube](https://www.thecodinganalyst.com/tutorial/integrate-code-quality-review-with-sonarqube/)
- [Configure Spring Security CSRF for testing on Swagger](https://www.thecodinganalyst.com/tutorial/Configure-spring-security-csrf-for-testing-on-swagger/)