---
title: "How to unit test rest controller in a Spring boot application"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Boot
  - Spring Testing
  - Mockito
---

Spring provides the [`@WebMvcTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html) annotation for testing Spring MVC components, providing only the configuration relevant to the MVC tests and not the full auto-configuration, so that we just have what we need to test the code within the controller only. This annotation will auto configure Spring Security and MockMvc as well, so that the security configurations are included and we can mock the services and repositories we need for the testing.

```
@ExtendWith(SpringExtension.class)
@WebMvcTest(TopicController.class)
public class TopicControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    TopicService topicService;

    @Autowired
    ObjectMapper objectMapper;
  }
```

A typical unit test on the controller will extend the `SpringExtension.class` to run the test, with the `@WebMvcTest` specifying the controller class to test. The `MockMvc` will be autowired, and the service used by the controller will be injected via the `@MockBean` annotation. As usually the rest controller will accept and output data in json format, we usually also autowire the [`ObjectMapper`](https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/2.3.1/com/fasterxml/jackson/databind/ObjectMapper.html) to convert our model class to json and back.

To stub the return value of the service, we use the `given` and `willReturn`.

```
List<Topic> topicList = List.of(
        new Topic("first topic"),
        new Topic("second topic")
);
given(topicService.list()).willReturn(topicList);
```

And to call the api and test the result, we use the [`MockMvc`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html) to perform the api call, and it will return the result as a [`ResultActions`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultActions.html). We can then chain the `ResultActions` with the `andExpect` method with [`ResultMatcher`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultMatcher.html) to test outcome.

```
mockMvc.perform(get(api))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.length()").value(2));
```

The `perform` method accepts a `RequestBuilder` as an argument, and we have the list of common http methods available as static methods in [`MockMvcRequestBuilders`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockMvcRequestBuilders.html) to use. The above example uses the get method, another example using the post method is illustrated below.

```
@Test
public void whenCreateTopic_thenReturn301CreatedAndTopic() throws Exception {
    Topic topic = Topic.builder()
            .topicId(1L)
            .title("topic")
            .build();
    given(topicService.create(any())).willReturn(topic);

    RequestBuilder request = post(api)
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(topic));
    mockMvc.perform(request)
            .andExpect(status().isCreated())
            .andExpect(content().contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
            .andExpectAll(
                    jsonPath("$.topicId").value("1"),
                    jsonPath("$.title").value("topic")
            );
}
```

The `jsonPath` ResultMatcher allows us to verify the json results using the JsonPath syntax (illustrated in the [README.md](https://github.com/json-path/JsonPath)).

The example above can be found in my [github repo](https://github.com/thecodinganalyst/forum/blob/initial-sample/src/test/java/com/hevlar/forum/controller/TopicControllerTest.java). 

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