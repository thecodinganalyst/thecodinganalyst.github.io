---
title: "Configure Spring Security CSRF for testing on Swagger"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring Security
  - CSRF
  - Swagger
---

Today we are going to work on adding [Spring Security](https://docs.spring.io/spring-security/reference/index.html) to our Spring boot application.  

To start, we will first need to add the dependencies for Spring Security into our gradle file.

```
implementation 'org.springframework.boot:spring-boot-starter-security:3.0.2'
testImplementation 'org.springframework.security:spring-security-test:6.0.1'
```

The default configuration in Spring Security will enable [Cross Site Request Forgery](https://docs.spring.io/spring-security/reference/features/exploits/csrf.html) protection by default using the [Synchronizer Token Pattern](https://docs.spring.io/spring-security/reference/features/exploits/csrf.html#csrf-protection-stp). It involves the server to generate a unique token for each http session, and passing it to the client in a secured manner. Then for subsequent http requests that are not `GET`, `HEAD`, `OPTIONS` or `TRACE`, the client will need to include the token in either the form as `_csrf` or in the http header as `X-XSRF-TOKEN`. The server will compare the token with the token generated earlier for the session, and will return a 403 forbidden status if the token does not match.

In Spring Security, the [CsrfConfigurer](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configurers/CsrfConfigurer.html) is used to configure how the CSRF protection should work. The `CsrfConfigurer` uses a [`token repository`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CsrfTokenRepository.html) to generate the token, and a [`request handler`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CsrfTokenRequestHandler.html) to do the token matching. 

In our example, we are going to reuse the Spring Boot application as introduced in my previous article - [Getting Started Spring Boot application](https://www.thecodinganalyst.com/tutorial/Spring-boot-application-getting-started/). It is a simple REST application with Swagger UI created with SpringDoc, so that we can test the api easily. 

However, testing the new spring boot 3 in swagger ui leaves us with quite a few default configurations in spring security that we cannot use. 

The default `token repository` used by spring security 6 to provide the initial token is with the [HttpSessionCsrfTokenRepository](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/HttpSessionCsrfTokenRepository.html), and that provides the token in the http session. It doesn't quite work in the context of the swagger ui as it has individual api for the user to run. The swagger ui's "Try it out" feature will not be able to get the token from the session. To overcome this problem, we need to overwrite the default token repository with the [CookieCsrfTokenRepository.withHttpOnlyFalse](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CookieCsrfTokenRepository.html#withHttpOnlyFalse()) which is less secured. Instead of placing the token in the http session, the `CookieCsrfTokenRepository` will create a cookie named `XSRF-TOKEN` with the token at the client when any `POST`, `PUT`, or `PATCH` request is made to the server. 

To do this, we create a new class for our security configurations so that we can overwrite the default security filter chain. In the past, we need the class to extend from WebSecurityConfigurerAdapter, but this has been [deprecated](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter); so we just need to add the `@Configuration` and `@EnableWebSecurity` annotations to our class.

```
@Configuration
@EnableWebSecurity
public class ForumSecurityConfiguration {
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception{
      httpSecurity
              .csrf()
              .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
              .csrfTokenRequestHandler(new CsrfTokenRequestAttributeHandler());
      return httpSecurity.build();
  }
}
```

Another problem with using CSRF protection with Swagger is that in Spring Security 6, the default `request handler` is the [XorCsrfTokenRequestAttributeHandler](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/XorCsrfTokenRequestAttributeHandler.html). This request handler will mask the token before sending and unmask it upon receiving. However, with our current setup, the token are not masked, and all the matching of the csrf will fail, and 403 forbidden is returned instead. So to overcome this, we overwrite the request handler with the old `CsrfTokenRequestAttributeHandler`. 

Another problem is that the `HttpSessionCsrfTokenRepository` is wrapped in a [LazyCsrfTokenRepository](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/LazyCsrfTokenRepository.html) and that means the CSRF token isn't generated by the server and passed to the client until it is required. So if the client only browse to the page or call get request, there are no CSRF token generated for the client when it needs to call a POST request. So the first POST request will fail. We can easily overcome it by creating a REST controller with a dummy post request like `POST /csrf` without any body, and call this method before calling any of the proper methods so that we have the token ready for use. If not, we can also do without it, and just let the first call fail since this is just for testing in swagger. We will use better CSRF protection method in production.

```
@RestController
@RequestMapping("/api/v1/csrf")
public class CsrfController {

    @PostMapping
    @ResponseStatus(HttpStatus.OK)
    public void CsrfToken(){

    }
}
```

Last but not least, we need to enable csrf in our swagger ui by simply adding the following line to our `application.properties` file, so that swagger will add the `X-XSRF-TOKEN` in the header with the value of the token from the cookie `XSRF-TOKEN`. 

```
springdoc.swagger-ui.csrf.enabled=true
```

To ensure we can overcome the CSRF protection in our test cases, we need to add the `with(csrf())` in our call, like the example below.

```
mvc.perform(post(topicsEndpoint)
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(topic))
                    .with(csrf()))
            .andExpect(status().isCreated())
```

Then in our unit tests annotated with `@WebMvcTest`, we need to add the `@ContextConfiguration` with both our test class and our security configuration class, like the example below.

```
@ExtendWith(SpringExtension.class)
@WebMvcTest(value = PostController.class)
@ContextConfiguration(classes = { ForumSecurityConfiguration.class, PostController.class })
class PostControllerTest {
}
```

A working source code of the above is available in the `csrf-example` tagged release - [https://github.com/thecodinganalyst/forum/tree/csrf-example](https://github.com/thecodinganalyst/forum/tree/csrf-example).

This is part of a series illustrating how to build a backend Spring boot application.
- [Getting Started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/)
- [Deploying to Docker](https://thecodinganalyst.github.io/tutorial/Deploying-mult-container-application-to-docker/)
- [Spring Data Testing](https://thecodinganalyst.github.io/tutorial/how-to-test-spring-data-repository/)
- [Testing Services](https://thecodinganalyst.github.io/tutorial/how-to-test-services-in-a-spring-boot-application/)
- [Unit Testing of Controller](https://thecodinganalyst.github.io/tutorial/how-to-unit-test-rest-controller-in-a-spring-boot-application/)
- [Integration Testing](https://thecodinganalyst.github.io/knowledgebase/how-to-do-integration-testing-in-spring-boot-rest-application/)
- [Code quality review with Sonarqube](https://www.thecodinganalyst.com/tutorial/integrate-code-quality-review-with-sonarqube/)
- [Configure Spring Security CSRF for testing on Swagger](https://www.thecodinganalyst.com/tutorial/Configure-spring-security-csrf-for-testing-on-swagger/)