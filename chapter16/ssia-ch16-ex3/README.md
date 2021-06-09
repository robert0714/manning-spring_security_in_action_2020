* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex3](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex3)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-16/93] (https://livebook.manning.com/book/spring-security-in-action/chapter-16/93) 

## Chapter 16 : GLOBAL METHOD SECURITY: PRE- AND POSTAUTHORIZATIONS 
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 


Global method security offers us three approaches to define the authorization rules that we discuss in this chapter:

* The pre-/postauthorization annotations
* The JSR 250 annotation, @RolesAllowed
* The @Secured annotation
 
### 16.3 Applying postauthorization

**Page 397**, Now say you want to allow a call to a method, but in certain circumstances, you want to
make sure the caller doesn’t receive the returned value. When we want to apply an
authorization rule that is verified after the call of a method, we use postauthorization.
It may sound a little bit awkward at the beginning: why would someone be able to execute
the code but not get the result? Well, it’s not about the method itself, but imagine
this method retrieves some data from a data source, say a web service or a database. You
can be confident about what your method does, but you can’t bet on the third party
your method calls. So you allow the method to execute, but you validate what it returns
and, if it doesn’t meet the criteria, you don’t let the caller access the return value.

To apply postauthorization rules with Spring Security, we use the ***@PostAuthorize*** annotation, which is similar to ***@PreAuthorize***, discussed in section 16.2. The annotation receives as a value the SpEL defining an authorization rule. We continue with an example in which you learn how to use the ***@PostAuthorize*** annotation and define postauthorization rules for a method (figure 16.7).

The scenario for our example, for which I created a project named ssia-ch16-ex3,
defines an object Employee. Our ***Employee*** has a name, a list of books, and a list of
authorities. We associate each ***Employee*** to a user of the application. To stay consistent
with the other examples in this chapter, we define the same users, Emma and Natalie.
We want to make sure that the caller of the method gets the details of the
employee only if the employee has read authority. Because we don’t know the authorities
associated with the employee record until we retrieve the record, we need to
apply the authorization rules after the method execution. For this reason, we use the
***@PostAuthorize*** annotation.

The configuration class is the same as we used in the previous examples. But, for
your convenience, I repeat it in the next listing.

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {

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

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```
We also need to declare a class to represent the ***Employee*** object with its name, book
list, and roles list. The following listing defines the ***Employee*** class.

```java
public class Employee {
    private String name;
    private List<String> books;
    private List<String> roles;
    // Omitted constructor, getters, and setters
}
```
We probably get our employee details from a database. To keep our example shorter, I
use a ***Map*** with a couple of records that we consider as our data source. In listing 16.9,
you find the definition of the ***BookService*** class. The ***BookService*** class also contains
the method for which we apply the authorization rules. Observe that the expression
we use with the ***@PostAuthorize*** annotation refers to the value returned by the
method ***returnObject***. The postauthorization expression can use the value
returned by the method, which is available after the method executes.
```java
@Service
public class BookService {

    private Map<String, Employee> records =
            Map.of("emma",
                   new Employee("Emma Thompson",
                           List.of("Karamazov Brothers"),
                           List.of("accountant", "reader")),
                   "natalie",
                   new Employee("Natalie Parker",
                           List.of("Beautiful Paris"),
                           List.of("researcher"))
                  );
    //Defines the expression for postauthorization
    @PostAuthorize("returnObject.roles.contains('reader')")
    public Employee getBookDetails(String name) {
        return records.get(name);
    }
}
```
Let’s also write a controller and implement an endpoint to call the method for which
we applied the authorization rule. The following listing presents this controller class.
```java
@RestController
public class BookController {

    @Autowired
    private BookService bookService;

    @GetMapping("/book/details/{name}")
    public Employee getDetails(@PathVariable String name) {
        return bookService.getBookDetails(name);
    }
}
```
You can now start the application and call the endpoint to observe the app’s behavior.
In the next code snippets, you find examples of calling the endpoint. Any of the users
can access the details of Emma because the returned list of roles contains the string
***“reader”***, but no user can obtain the details for Natalie. Calling the endpoint to get
the details for Emma and authenticating with user Emma, we use this command:

```bash
 curl -u emma:12345 http://localhost:8080/book/details/emma  |jq "."
```
The response body is

```json
{
  "name": "Emma Thompson",
  "books": [
    "Karamazov Brothers"
  ],
  "roles": [
    "accountant",
    "reader"
  ]
}

```
Calling the endpoint to get the details for Emma and authenticating with user Natalie,
we use this command:

```bash
curl -u natalie:12345 http://localhost:8080/book/details/emma |jq "."
```
The response body is

```json
{
  "name": "Emma Thompson",
  "books": [
    "Karamazov Brothers"
  ],
  "roles": [
    "accountant",
    "reader"
  ]
}
```

Calling the endpoint to get the details for Natalie and authenticating with user Emma,
we use this command:
```bash
curl -u emma:12345 http://localhost:8080/book/details/natalie |jq "."
```
The response body is

```json
{
  "timestamp": "2021-06-09T07:37:16.035+00:00",
  "status": 403,
  "error": "Forbidden",
  "message": "",
  "path": "/book/details/natalie"
}
```

Calling the endpoint to get the details for Natalie and authenticating with user Natalie,
we use this command:
```bash
curl -u natalie:12345 http://localhost:8080/book/details/natalie|jq "."
```
The response body is

```json
{
  "timestamp": "2021-06-09T07:37:55.426+00:00",
  "status": 403,
  "error": "Forbidden",
  "message": "",
  "path": "/book/details/natalie"
}
```

*  ***NOTE*** You can use both @PreAuthorize and @PostAuthorize on the
same method if your requirements need to have both preauthorization and
postauthorization.