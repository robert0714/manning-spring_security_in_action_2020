* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch17-ex5](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch17-ex5)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-17/82](https://livebook.manning.com/book/spring-security-in-action/chapter-17/82) 
## Chapter 17 : GLOBAL METHOD SECURITY: PRE- AND POSTFILTERING
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 

We name such a functionality filtering,
and we classify it in two categories:
* ***Prefiltering***—The framework filters the values of the parameters before calling
the method.
* ***Postfiltering***—The framework filters the returned value after the method call.

### 17.3 Using filtering in Spring Data repositories

We discussed earlier in this section that using @PostFilter in the repository isn’t the
best choice. We should instead make sure we don’t select from the database what we
don’t need. So how can we change our example to select only the required data
instead of filtering data after selection? We can provide SpEL expressions directly in
the queries used by the repository classes. To achieve this, we follow two simple steps:

1. We add an object of type ***SecurityEvaluationContextExtension*** to the Spring context. We can do this using a simple @Bean method in the configuration class.
2. We adjust the queries in our repository classes with the proper clauses for selection.

In our project, to add the ***SecurityEvaluationContextExtension*** bean in the
context, we need to change the configuration class as presented in listing 17.12. To
keep all the code associated with the examples in the book, I use here another project
that named ssia-ch17-ex5.


Listing 17.12 Adding the ***SecurityEvaluationContextExtension*** to the context
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig {

  //Uses SpEL in the query to add a condition on the owner of the record
  @Bean
  public SecurityEvaluationContextExtension securityEvaluationContextExtension() {
    return new SecurityEvaluationContextExtension();
  }

    // Omitted declaration of the UserDetailsService and PasswordEncoder
}
```

In the ***ProductRepository*** interface, we add the query prior to the method, and we adjust the WHERE clause with the proper condition using a SpEL expression. The following listing presents the change.

Listing 17.13 Using SpEL in the query in the repository interface
```java
public interface ProductRepository
        extends JpaRepository<Product, Integer> {

    //Uses SpEL in the query to add a condition on the owner of the record
    @Query("SELECT p FROM Product p  WHERE p.name LIKE %:text% AND  p.owner=?#{authentication.name}")
    List<Product> findProductByNameContains(String text);
}
```
We can now start the application and test it by calling the /products/{text} endpoint.
We expect that the behavior remains the same as for the case where we used
***@PostFilter***. But now, only the records for the right owner are retrieved from the
database, which makes the functionality faster and more reliable. The next code snippets
present the calls to the endpoint. To call the endpoint /products and authenticate
with user Nikolai, we use this command:

```bash
curl -u nikolai:12345 http://localhost:8080/products/c
```

The response body is
```json
[
  {"id":2,"name":"candy","owner":"nikolai"}
]
```
To call the endpoint /products and authenticate with user Julien, we use this command:
```bash
curl -u julien:12345 http://localhost:8080/products/c
```

The response body is
```json
[
  {"id":3,"name":"chocolate","owner":"julien"}
]
```