* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex1](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex1)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-16/10](https://livebook.manning.com/book/spring-security-in-action/chapter-16/30) 

## Chapter 16 : GLOBAL METHOD SECURITY: PRE- AND POSTAUTHORIZATIONS 
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 

#### 16.1.2 Enabling global method security in your project

Global method security offers us three approaches to define the authorization rules that we discuss in this chapter:

* The pre-/postauthorization annotations
* The JSR 250 annotation, @RolesAllowed
* The @Secured annotation
 

**Page 392**, in almost all cases, ***pre-/postauthorization annotations*** are the only approach used, we discuss this approach in this chapter. To enable this approach, we use the ***prePostEnabled*** attribute of the ***@EnableGlobalMethodSecurity*** annotation.

We present a short overview of the other two options previously mentioned at the end of this chapter.

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {
}
```

You can use global method security with any authentication approach, from HTTP
Basic authentication to OAuth 2. To keep it simple and allow you to focus on new
details, we provide global method security with HTTP Basic authentication. For this
reason, the pom.xml file for the projects in this chapter only needs the web and
Spring Security dependencies, as the next code snippet presents:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
### 16.2 Applying preauthorization for authorities and roles
In this section, we implement an example of preauthorization. For our example, we
continue with the project ssia-ch16-ex1 started in section 16.1. As we discussed in section
16.1, preauthorization implies defining authorization rules that Spring Security
applies before calling a specific method. If the rules aren’t respected, the framework
doesn’t call the method.

The application we implement in this section has a simple scenario. It exposes an
endpoint, ***/hello***, which returns the string ***"Hello, "*** followed by a name. To obtain the name, the controller calls a service method (figure 16.5). This method applies a preauthorization rule to verify the user has write authority.

I added a ***UserDetailsService*** and a ***PasswordEncoder*** to make sure I have
some users to authenticate. To validate our solution, we need two users: one user with
write authority and another that doesn’t have write authority. We prove that the first
user can successfully call the endpoint, while for the second user, the app throws an
authorization exception when trying to call the method. The following listing shows the
complete definition of the configuration class, which defines the ***UserDetails-Service*** and the ***PasswordEncoder***.

```java
@Configuration
//Enables global method security for pre-/postauthorization
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {

    //Adds a UserDetailsService to the Spring context with two users for testing
    @Bean
    public UserDetailsService userDetailsService() {
        var service = new InMemoryUserDetailsManager();

        var u1 = User.withUsername("natalie")
                    .password("12345")
                    .authorities("read")
                .build();

        var u2 = User.withUsername("emma")
                .password("12345")
                .authorities("write")
                .build();

        service.createUser(u1);
        service.createUser(u2);

        return service;
    }

    //Adds a PasswordEncoder to the Spring context
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

To define the authorization rule for this method, we use the ***@PreAuthorize*** annotation.
The ***@PreAuthorize*** annotation receives as a value a Spring Expression Language
(SpEL) expression that describes the authorization rule. In this example, we
apply a simple rule.

You can define restrictions for users based on their authorities using the ***hasAuthority()*** method. You learned about the ***hasAuthority()*** method in chapter
7, where we discussed applying authorization at the endpoint level. The following listing
defines the service class, which provides the value for the name.

```java
@Service
public class NameService {

    //Defines the authorization rule. Only users having write authority can call the method.
    @PreAuthorize("hasAuthority('write')")
    public String getName() {
        return "Fantastico";
    }
}
```

We define the controller class in the following listing. It uses NameService as a dependency.

```java
@RestController
public class HelloController {

    //Injects the service from the context
    @Autowired
    private NameService nameService;

    @GetMapping("/hello")
    public String hello() {
        //Calls the method for which we apply the preauthorization rules
        return "Hello, " + nameService.getName();
    }
}
```
You can now start the application and test its behavior. We expect only user Emma to be
authorized to call the endpoint because she has write authorization. The next code
snippet presents the calls for the endpoint with our two users, Emma and Natalie. To
call the /hello endpoint and authenticate with user Emma, use this cURL command:

```bash
curl -u emma:12345 http://localhost:8080/hello
```
The response body is

```bash
Hello, Fantastico
```

To call the /hello endpoint and authenticate with user Natalie, use this cURL command:

```bash
curl -u natalie:12345 http://localhost:8080/hello  |jq "."
```

The response body is
```json
{
  "timestamp": "2021-06-09T04:01:41.456+00:00",
  "status": 403,
  "error": "Forbidden",
  "message": "",
  "path": "/hello"
}
```

Similarly, you can use any other expression we discussed in chapter 7 for endpoint authentication. Here’s a short recap of them:

* ***hasAnyAuthority()***—Specifies multiple authorities. The user must have at least one of these authorities to call the method.
* ***hasRole()***—Specifies a role a user must have to call the method.
* ***hasAnyRole()***—Specifies multiple roles. The user must have at least one of them to call the method.