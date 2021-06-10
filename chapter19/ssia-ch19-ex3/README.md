* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch19-ex3](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch19-ex3)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-19/71](https://livebook.manning.com/book/spring-security-in-action/chapter-19/71) 

# Chapter 19 : SPRING SECURITY FOR REACTIVE APPS
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 

## 19.3 Configuring authorization rules in reactive apps
In this section, we discuss configuring authorization rules. As you already know from
the previous chapters, authorization follows authentication. We discussed in sections
19.1 and 19.2 how Spring Security manages users and the ***SecurityContext*** in reactive
apps. But once the app finishes authentication and stores the details of the
authenticated request in the ***SecurityContext***, it’s time for authorization.

As for any other application, you probably need to configure authorization rules
when developing reactive apps as well. To teach you how to set authorization rules in
reactive apps, we’ll discuss first in section 19.3.1 the way you make configurations at
the endpoint layer. Once we finish discussing authorization configuration at the endpoint
layer, you’ll learn in section 19.3.2 how to apply it at any other layer of your
application using method security.

### 19.3.1 Applying authorization at the endpoint layer in reactive apps
In this section, we discuss configuring authorization at the endpoint layer in reactive
apps. Setting the authorization rules in the endpoint layer is the most common
approach for configuring authorization in a web app. You already discovered this while working on the previous examples in this book. Authorization configuration at
the endpoint layer is essential—you use it in almost every app. Thus, you need to
know how to apply it for reactive apps as well.

You learned from previous chapters to set the authorization rules by overriding the
***configure(HttpSecurity http)*** method of the ***WebSecurityConfigurerAdapter*** class. This approach doesn’t work in reactive apps. To teach you how to configure
authorization rules for the endpoint layer properly for reactive apps, we start by
working on a new project, which I named ssia-ch19-ex3.

In reactive apps, Spring Security uses a contract named ***SecurityWebFilterChain*** to apply the configurations we used to do by overriding one of the configure()
methods of the ***WebSecurityConfigurerAdapter*** class, as discussed in
previous chapters. With reactive apps, we add a bean of type SecurityWebFilter-
Chain in the Spring context. To teach you how to do this, let’s implement a basic
application having two endpoints that we secure independently. In the pom.xml file
of our newly created ssia-ch19-ex3 project, add the dependencies for reactive web
apps and, of course, Spring Security:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

Create a controller class to define the two endpoints for which we configure the
authorization rules. These endpoints are accessible at the paths /hello and /ciao.
To call the /hello endpoint, a user needs to authenticate, but you can call the /ciao
endpoint without authentication. The following listing presents the definition of the
controller.

Listing 19.5 The HelloController class defining the endpoints to secure
```java
@RestController
public class HelloController {

  @GetMapping("/hello")
  public Mono<String> hello(Mono<Authentication> auth) {
    Mono<String> message = auth.map(a -> "Hello " + a.getName());
    return message;
  }

  @GetMapping("/ciao")
  public Mono<String> ciao() {
    return Mono.just("Ciao!");
  }
}
```
In the configuration class, we make sure to declare a ***ReactiveUserDetailsService***
and a ***PasswordEncoder*** to define a user, as you learned in section 19.2. The
following listing defines these declarations.

```java
@Configuration
public class ProjectConfig {

  @Bean
  public ReactiveUserDetailsService userDetailsService() {
    var  u = User.withUsername("john")
            .password("12345")
            .authorities("read")
            .build();

    var uds = new MapReactiveUserDetailsService(u);

    return uds;
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }

  // ...
}
```
In listing 19.7, we work in the same configuration class we declared in listing 19.6, but omit the declaration of the ***ReactiveUserDetailsService*** and the ***PasswordEncoder*** so that you can focus on the authorization configuration we discuss. In listing 19.7, you might notice that we add a bean of type ***SecurityWebFilterChain*** to the Spring context. The method receives as a parameter an object of type ***ServerHttpSecurity***, which is injected by Spring. ***ServerHttpSecurity*** enables us to build an instance of SecurityWebFilterChain. ***ServerHttpSecurity*** provides methods for configuration similar to the ones you used when configuring authorization for non-reactive apps.
```java
@Configuration
public class ProjectConfig {

  // Omitted code

  @Bean
  public SecurityWebFilterChain securityWebFilterChain(
    ServerHttpSecurity http) {
    
    return http.authorizeExchange()  //Begins the endpoint authorization configuration
              .pathMatchers(HttpMethod.GET, "/hello")
                   .authenticated()  //Selects the requests for which we apply the authorization rules
              .anyExchange() //Configures the selected requests to only be accessible when authenticated
                   .permitAll()
              .and().httpBasic() //Allows requests to be called without needing authentication
              .and().build();  //Builds the SecurityWebFilterChain object to be returned
    }
}
```

We start the authorization configuration with the ***authorizeExchange()*** method.
We call this method similarly to the way we call the ***authorizeRequests()*** method
when configuring endpoint authorization for non-reactive apps. Then we continue by
using the ***pathMatchers()*** method. You can consider this method as the equivalent of
using ***mvcMatchers()*** when configuring endpoint authorization for non-reactive apps.

As for non-reactive apps, once we use the matcher method to group requests to
which we apply the authorization rule, we then specify what the authorization rule is.
In our example, we called the ***authenticated()*** method, which states that only
authenticated requests are accepted. You used a method named ***authenticated()***
also when configuring endpoint authorization for non-reactive apps. The methods for
reactive apps are named the same to make them more intuitive. Similarly to the
***authenticated()*** method, you can also call these methods:

* ***permitAll()***—Configures the app to allow requests without authentication
* ***denyAll()***—Denies all requests
* ***hasRole()*** and ***hasAnyRole()***—Apply rules based on roles
* *** hasAuthority()*** and ***hasAnyAuthority()***—Apply rules based on authorities

It looks like something’s missing, doesn’t it? Do we also have an ***access()*** method as
we had for configuring authorization rules in non-reactive apps? Yes. But it’s a bit different,
so we’ll work on a separate example to prove it. Another similarity in naming is
the ***anyExchange()*** method that takes the role of what used to be ***anyRequest()*** in
non-reactive apps.

* ***NOTE*** Why is it called anyExchange(), and why didn’t the developers keep
the same name for the method anyRequest()? Why ***authorizeExchange()*** and why not ***authorizeRequests()***? This simply comes from the terminology used with reactive apps. We generally refer to communication between two components in a reactive fashion as ***exchanging data***. This
reinforces the image of data being sent as segmented in a continuous stream
and not as a big bunch in one request.

We also need to specify the authentication method like any other related configuration.
We do this with the same ***ServerHttpSecurity*** instance, using methods with the
same name and in the same fashion you learned to use for non-reactive apps: ***httpBasic()***, ***formLogin()***, ***csrf()***, ***cors()***, adding filters and customizing the filter
chain, and so on. In the end, we call the ***build()*** method to create the instance of
***SecurityWebFilterChain***, which we finally return to add to the Spring context.