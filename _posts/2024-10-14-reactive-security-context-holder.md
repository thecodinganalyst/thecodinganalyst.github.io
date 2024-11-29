---
title: "Getting the security context in reactive spring applications"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Spring Security
  - Webflux
  - Project Reactor
---

In reactive applications, typically when you are using Spring Webflux, handling security contexts, including authentication and authorization, requires a non-blocking approach to integrate with reactive streams effectively. The `ReactiveSecurityContextHolder` class is a vital component in Spring Security's reactive stack, helping manage security contexts across different reactive threads and ensuring consistent access to authentication information throughout the request lifecycle. In this article, we'll explore what `ReactiveSecurityContextHolder` is, when to use it, and how to incorporate it into your reactive Spring applications.

<!--more-->

`ReactiveSecurityContextHolder` is a utility class in Spring Security's reactive stack, designed to work with Project Reactor, a reactive programming library. It provides non-blocking access to the security context, holding the current user's authentication details in a Mono<SecurityContext>. This class is akin to SecurityContextHolder in traditional, blocking Spring Security but tailored for reactive applications where blocking operations are discouraged or even disallowed.
The `ReactiveSecurityContextHolder` operates within the ReactorContext, which carries contextual information in a reactive application. In practice, this enables the retrieval and manipulation of security context information (such as user authentication) seamlessly across different stages of a reactive pipeline.



## Example to get the security context with ReactiveSecurityContextHolder

```
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.ReactiveSecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import reactor.core.publisher.Mono;

public Mono<String> getUserDetails() {
    return ReactiveSecurityContextHolder.getContext()
        .map(securityContext -> {
            Authentication authentication = securityContext.getAuthentication();
            UserDetails userDetails = (UserDetails) authentication.getPrincipal();
            return "User: " + userDetails.getUsername();
        });
}
```

## Example to save the security context with ReactiveSecurityContextHolder

```
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextImpl;
import org.springframework.security.core.context.ReactiveSecurityContextHolder;
import reactor.core.publisher.Mono;

public Mono<Void> setCustomSecurityContext(String username, String role) {
    UsernamePasswordAuthenticationToken authentication = 
        new UsernamePasswordAuthenticationToken(username, null, List.of(new SimpleGrantedAuthority(role)));
    SecurityContextImpl securityContext = new SecurityContextImpl(authentication);

    return Mono.deferContextual(context -> 
        ReactiveSecurityContextHolder.withSecurityContext(Mono.just(securityContext)))
        .then();
}

```

Reactive Web Filters are often used for cross-cutting concerns like logging, authentication, and authorization. In scenarios where you need to intercept and modify the SecurityContext in a filter, you can use ReactiveSecurityContextHolder as shown below:

```
import org.springframework.web.server.WebFilter;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

public class CustomSecurityContextFilter implements WebFilter {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        return ReactiveSecurityContextHolder.getContext()
            .flatMap(securityContext -> {
                // Modify or inspect security context as needed
                return chain.filter(exchange).contextWrite(ReactiveSecurityContextHolder.withSecurityContext(Mono.just(securityContext)));
            });
    }
}

```

## Practical Tips
- **Avoid Blocking Calls:** Ensure that you only use ReactiveSecurityContextHolder within reactive flows, avoiding any blocking operations as they could lead to performance issues in reactive applications.
- **Context Propagation:** Use contextWrite to propagate security context changes within the reactive pipeline, especially if you're creating custom authentication mechanisms or performing role checks.
- **Fallback Mechanisms:** Always handle cases where the security context might be absent, especially in routes that are not secured, to avoid NullPointerExceptions.

`ReactiveSecurityContextHolder` is a powerful tool for managing security contexts in Spring reactive applications, enabling access to authentication details across non-blocking, asynchronous streams. By following the patterns and examples above, you can confidently manage security in your reactive applications, ensuring a seamless and efficient user authentication experience.

Understanding `ReactiveSecurityContextHolder` and its usage will help you harness the full power of Spring Security's reactive capabilities, making it an essential part of any developer's toolkit in reactive programming.
