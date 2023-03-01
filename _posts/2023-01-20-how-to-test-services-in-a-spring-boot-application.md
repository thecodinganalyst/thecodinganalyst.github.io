---
title: "How to test services in a Spring boot application"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Boot
  - Spring Testing
  - Mockito
---

Unit testing services is easy, cause we don't need to set up any application context in Spring, since the purpose of unit testing is just to isolate testing of the code that is written in the function we want to test. We shouldn't be testing any other code that are in other classes or functions. 

To run the tests, we can extend the class with just the `MockitoExtension`. The only dependent in our services are the repository, and as we shouldn't be testing the functionalities of the repository class, we will just mock it by adding the `@Mock` annotation to the declaration of our repository.

In order to have the service in our test, and get the mocked repository into our service, we annotate our service with the `@InjectMocks`, and Mockito will settle that for us.

```
@ExtendWith(MockitoExtension.class)
public class PostServiceTest {

    @Mock
    PostRepository postRepository;

    @InjectMocks
    PostService postService;
}
```

Suppose we want to test this function, which uses the `findById` of the topicRepository.

```
public Topic update(Topic topic){
    LocalDateTime created = topicRepository.findById(topic.getTopicId())
            .orElseThrow()
            .getCreated();
    topic.setCreated(created);
    return topicRepository.save(topic);
}
```

To stub the function of the repository class we are using, we use the `given` method of Mockito to return a stub of the expected result of the function we will use in our service. Here, we tells Mockito to return an Optional of `Post` when the `findById` function of our repository is called. 

```
//given
LocalDateTime createdDateTime = LocalDateTime.now().minusHours(1);
Topic topic = Topic.builder()
        .topicId(1L)
        .title("First Topic")
        .created(createdDateTime)
        .updated(createdDateTime)
        .build();
given(postRepository.findById(1L)).willReturn(Optional.of(post));
```

Then we can call the function we need to test and assert the expected results.

```
//when
Topic updatedTopic = Topic.builder().topicId(1L).title("Updated Topic").build();
topicService.update(updatedTopic);

//then
assertThat(updatedTopic.getCreated(), is(createdDateTime));
```

To test for exception, we can use the assertThrows method.

```
@Test()
public void givenTopicDoesNotExist_whenDelete_thenThrowException(){
    //given
    given(topicRepository.existsById(1L)).willReturn(false);

    //when
    Throwable throwable = assertThrows(NoSuchElementException.class, () -> topicService.delete(1L));

    //then
    assertThat(throwable, is(notNullValue()));
}
```

The code for the above example is available on [https://github.com/thecodinganalyst/forum/blob/master/src/test/java/com/hevlar/forum/service/TopicServiceTest.java](https://github.com/thecodinganalyst/forum/blob/initial-sample/src/test/java/com/hevlar/forum/service/TopicServiceTest.java).

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