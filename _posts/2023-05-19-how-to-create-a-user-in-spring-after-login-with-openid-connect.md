---
title: "How to create a user in Spring after login with OpenID Connect"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Spring
  - OAuth2
  - OpenID Connect
  - Spring Security
  - MongoDB
---

Following my [last article on creating OpenID Connect login with Spring Boot](https://www.thecodinganalyst.com/tutorial/OAuth2-login-with-spring-boot/), we shall now enhance the application to create a user in our MongoDB database for new user logins. I am using a local installation of [MongoDB Community Edition](https://www.mongodb.com/try/download/community) for my database. 

Firstly, we add the `implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'` dependency to our `build.gradle` file, so that we can use MongoDB in our spring boot project. Then we add the database configurations in our `application.properties`. 

```
spring.data.mongodb.database=OAuth2Sample
spring.data.mongodb.port=27017
spring.data.mongodb.host=localhost
spring.data.mongodb.auto-index-creation=true
```

We need the `auto-index-creation` property so that the indexes created in our classes below can be created automatically. [https://docs.spring.io/spring-data/mongodb/docs/current-SNAPSHOT/reference/html/#mapping.index-creation](https://docs.spring.io/spring-data/mongodb/docs/current-SNAPSHOT/reference/html/#mapping.index-creation)

Next, we create our user class, which we shall name it `AppUser`. This class should implement the [`UserDetails`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetails.html) interface, as `UserDetails` is the representation of the user returned by Spring Security. 

```
@Document(collection = "AppUser")
@NoArgsConstructor
@AllArgsConstructor
@Data
public class AppUser implements UserDetails {

    @Indexed(unique = true)
    String username;
    String password;
    boolean accountNonExpired;
    boolean accountNonLocked;
    boolean credentialsNonExpired;
    boolean enabled;
    Collection<? extends GrantedAuthority> grantedAuthorities;

    public AppUser(String username, String password){
        this.username = username;
        this.password = password;
        this.accountNonExpired = true;
        this.accountNonLocked = true;
        this.credentialsNonExpired = true;
        this.enabled = true;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return grantedAuthorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

The `@Document(collection = "AppUser")` annotation specifies that this is the entity class that represents a mongodb document, under the collection named `AppUser`. In terms of RDBMS, a collection is like a table, whereas a document is like a row in the table. 

The `@Indexed(unique = true)` annotation over the `username` attribute specifies that the field should be a unique index. Mongodb will create a default `_id` as the primary key, and we are not going to overwrite it, so we just have to make the username unique.

Next, we can create the repository class for this entity.

```
@Repository
public interface AppUserRepository extends MongoRepository<AppUser, String>{
    Optional<AppUser> findByUsername(String username);
}
```

Next again, we shall create a class to implement `UserDetailsService`. This is the service used by spring security to retrieve the user. What we are doing here is to override the `loadUserByUsername` method, so that whenever a call to retrieve the user, it will get the user from our `AppUserRepository`.

```
@Service
public class AppUserDetailsService implements UserDetailsService {

    AppUserRepository appUserRepository;

    public AppUserDetailsService(AppUserRepository appUserRepository){
        this.appUserRepository = appUserRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return appUserRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User " + username + " not found"));
    }

}
```

Lastly, in order to only add the user after it is authenticated, we shall add an EventListener to listen for the [`AuthenticationSuccessEvent`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/event/AuthenticationSuccessEvent.html).

```
@EventListener
public void onSuccess(AuthenticationSuccessEvent successEvent){
    Authentication authentication = successEvent.getAuthentication();

    OAuth2LoginAuthenticationToken oAuth2Token = getOAuth2Token(authentication);
    OAuth2User oAuth2User = getOAuth2User(authentication);

    if(oAuth2Token != null && oAuth2User != null){
        String email = oAuth2User.getAttribute("email");
        if(email == null) return;
        try{
            userDetailsService.loadUserByUsername(email);
        }catch (UsernameNotFoundException exception){
            AppUser appUser = new AppUser(email, null);
            appUserRepository.save(appUser);
        }
    }
}

private OAuth2User getOAuth2User(Authentication authentication){
    return authentication.getPrincipal() instanceof OAuth2User user ?
            user :
            null;
}

private OAuth2LoginAuthenticationToken getOAuth2Token(Authentication authentication){
    return authentication instanceof OAuth2LoginAuthenticationToken token ?
            token :
            null;
}
```

The event will contain the `Authentication`, and to determine if it is an OIDC or OAuth2 login, and not other authentication types like UsernamePassword, we only proceed if the Authentication is of type `OAuth2LoginAuthenticationToken`. The principal of the `OAuth2LoginAuthenticationToken` should also be of type `OAuth2User`. 

![Authentication](/assets/images/2023/05/spring-security-authentication.png)

The `OAuth2User` will have a map of attributes, like `name`, `first name`, `last name`, `email`, etc. For our context, we just need to retrieve the email to be used as the `username` for our `UserDetails`. After we get the email, we can use the `UserDetailsService` to try and retrieve the user from our repository, and if the user is not found, then we create a new user and add it to our repository. 


Lastly, to test the above implementation works, we shall add the unit tests as such.

```
@ExtendWith(MockitoExtension.class)
class AuthenticationEventListenerTest {

    @InjectMocks
    AuthenticationEventListener authenticationEventListener;

    @Mock
    AuthenticationSuccessEvent successEvent;

    @Mock
    UserDetailsService userDetailsService;

    @Mock
    OAuth2LoginAuthenticationToken authentication;

    @Mock
    OAuth2User oAuth2User;

    @Mock
    UserDetails userDetails;

    @Mock
    AppUserRepository appUserRepository;

    @Test
    void givenUserDoesNotExist_whenLoginWithOAuth2_thenAddUser() {

        when(successEvent.getAuthentication()).thenReturn(authentication);
        when(authentication.getPrincipal()).thenReturn(oAuth2User);
        when(oAuth2User.getAttribute("email")).thenReturn("user@example.com");
        when(userDetailsService.loadUserByUsername(anyString())).thenThrow(new UsernameNotFoundException(""));

        authenticationEventListener.onSuccess(successEvent);

        verify(userDetailsService).loadUserByUsername("user@example.com");
        verify(appUserRepository).save(any(AppUser.class));
    }

    @Test
    void givenUserDoExist_whenLoginWithOAuth2_thenUserNotAdded() {

        when(successEvent.getAuthentication()).thenReturn(authentication);
        when(authentication.getPrincipal()).thenReturn(oAuth2User);
        when(oAuth2User.getAttribute("email")).thenReturn("user@example.com");
        when(userDetailsService.loadUserByUsername(anyString())).thenReturn(userDetails);

        authenticationEventListener.onSuccess(successEvent);

        verify(userDetailsService).loadUserByUsername("user@example.com");
        verify(appUserRepository, never()).save(any(AppUser.class));
    }
}
```

A full working copy of the above implementation can be found on [https://github.com/thecodinganalyst/oauth2](https://github.com/thecodinganalyst/oauth2).
