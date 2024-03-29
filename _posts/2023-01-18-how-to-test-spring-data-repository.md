---
title: "How to test Spring Data repository"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Data
  - Spring Testing
---

Usually we do not need to write tests for the functionalities provided by the framework, but we can do so when certain functionalities we use from the framework is not so clearcut. An example is when we are using the [custom query creation by function method](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation) in Spring Data. Using back the example from my previous post - [Getting started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/), we have the following query by function name - `findAllByTopic_TopicId`.

```
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findAllByTopic_TopicId(Long topicId);
}
```

If we are not that familiar with the function name conventions, it will be good to just write a test to ensure it works as intended. 

To write a new test class for the repository, we need to use the `@DataJpaTest` annotation.

```
@ExtendWith(SpringExtension.class)
@DataJpaTest
public class PostRepositoryTest {

}
```

This annotation will help setup the environment you need to test the repository, like performing entity scan, so that you can use dependency injection, as well as turning on logging. With this annotation, the tests in the class will be transactional and rolled back at the end of each test, and it will use the embedded in-memory database. Do include h2 database in the dependencies of your gradle or maven file. 

```
testImplementation 'com.h2database:h2:2.1.214'
```

Then we autowire a `TestEntityManager` and the repository we want to test into the class.

```
@Autowired
private TestEntityManager entityManager;

@Autowired
private PostRepositoryTest repository;
```

The `TestEntityManager` is a subset of `EntityManager`, with methods useful for tests. We'll use it to insert data to our in memory database. 

The context in our example is such that a `Topic` can have multiple `Post`s, joined by the foreign key TopicId. And our method is retrieving all the posts based on the TopicId. 

We are using the [GivenWhenThen](https://martinfowler.com/bliki/GivenWhenThen.html) style to write our test. Our precondition is to first create the data - a Topic and 2 Posts, then we persist and flush to save it the the database. 

The action - denoted by the `when`, is where we execute the method that we want to test. 

Lastly, the `then` section is where we specify the expected outcome with our assertion statements. 

```
@Test
public void givenTopicHasPosts_whenFindAllByTopic_TopicId_thenReturnTopics(){
    //given
    Topic topic = new Topic("First Topic");
    entityManager.persist(topic);
    entityManager.flush();

    Post post1 = new Post("First Post", topic);
    Post post2 = new Post("Second Post", topic);
    entityManager.persist(post1);
    entityManager.persist(post2);
    entityManager.flush();

    //when
    List<Post> postList = postRepository.findAllByTopic_TopicId(topic.getTopicId());

    //then
    assertThat(postList.size(), is(2));
    assertThat(postList, containsInAnyOrder(post1, post2));
}
```

Lastly, we can add the following settings to the `application.properties` in the `src/test/resources` folder to show and format the sql that are generated by the framework in our log.

```
spring.jpa.show-sql = true
spring.jpa.properties.hibernate.format_sql=true
```

A working copy of the code is available on my [github repository](https://github.com/thecodinganalyst/forum/blob/initial-sample/src/test/java/com/hevlar/forum/persistence/PostRepositoryTest.java).

The source code for the above application is available on my github tagged as the [initial-sample](https://github.com/thecodinganalyst/forum/tree/initial-sample) release, as the code will be updated subsequently as I introduced new articles on how to improve the solution.

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