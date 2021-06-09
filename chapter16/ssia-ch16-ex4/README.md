* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex4](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex4)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-16/128](https://livebook.manning.com/book/spring-security-in-action/chapter-16/128) 

## Chapter 16 : GLOBAL METHOD SECURITY: PRE- AND POSTAUTHORIZATIONS 
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 


Global method security offers us three approaches to define the authorization rules that we discuss in this chapter:

* The pre-/postauthorization annotations
* The JSR 250 annotation, @RolesAllowed
* The @Secured annotation
 
### 16.4 Implementing permissions for methods

**Page 401**, Up to now, you learned how to define rules with simple expressions for preauthorization
and postauthorization. Now, let’s assume the authorization logic is more complex,
and you cannot write it in one line. It’s definitely not comfortable to write huge
SpEL expressions. I never recommend using long SpEL expressions in any situation,
regardless if it’s an authorization rule or not. It simply creates hard-to-read code, and
this affects the app’s maintainability. When you need to implement complex authorization
rules, instead of writing long SpEL expressions, take the logic out in a separate
class. Spring Security provides the concept of ***permission***, which makes it easy to write
the authorization rules in a separate class so that your application is easier to read and
understand.

In this section, we apply authorization rules using permissions within a project. I
named this project ssia-ch16-ex4. In this scenario, you have an application managing
documents. Any document has an owner, which is the user who created the document.
To get the details of an existing document, a user either has to be an admin or
they have to be the owner of the document. We implement a permission evaluator to
solve this requirement. The following listing defines the document, which is only a
plain Java object.

Listing 16.11 The Document class
```java
public class Document {

  private String owner;

  // Omitted constructor, getters, and setters
}
```
To mock the database and make our example shorter for your comfort, I created a
repository class that manages a few document instances in a Map. You find this class in
the next listing.
```java
@Repository
public class DocumentRepository {
  //  Identifies each document by a unique code and names the owner
  private Map<String, Document> documents =
    Map.of("abc123", new Document("natalie"),
           "qwe123", new Document("natalie"),
           "asd555", new Document("emma"));

  public Document findDocument(String code) {
    //Obtains a document by using its unique identification code
    return documents.get(code);
  }
}
```
A service class defines a method that uses the repository to obtain a document by its
code. The method in the service class is the one for which we apply the authorization
rules. The logic of the class is simple. It defines a method that returns the ***Document***
by its unique code. We annotate this method with ***@PostAuthorize*** and use a
***hasPermission()*** SpEL expression. This method allows us to refer to an external
authorization expression that we implement further in this example. Meanwhile,
observe that the parameters we provide to the ***hasPermission()*** method are the
***returnObject***, which represents the value returned by the method, and the name
of the role for which we allow access, which is ***'ROLE_admin'***. You find the definition
of this class in the following listing.
```java
@Service
public class DocumentService {

  @Autowired
  private DocumentRepository documentRepository;

  @PostAuthorize
  ("hasPermission(returnObject, 'ROLE_admin')")
  public Document getDocument(String code) {
    //Uses the hasPermission() expression to refer to an authorization expression
    return documentRepository.findDocument(code);
  }
}
```
It’s our duty to implement the permission logic. And we do this by writing an object
that implements the ***PermissionEvaluator*** contract. The ***PermissionEvaluator***
contract provides two ways to implement the permission logic:

* ***By object and permission***—Used in the current example, it assumes the permission
evaluator receives two objects: one that’s subject to the authorization rule and
one that offers extra details needed for implementing the permission logic.
* ***By object ID, object type, and permission***—Assumes the permission evaluator
receives an object ID, which it can use to retrieve the needed object. It also
receives a type of object, which can be used if the same permission evaluator
applies to multiple object types, and it needs an object offering extra details for
evaluating the permission.

In the next listing, you find the ***PermissionEvaluator*** contract with two methods.

```java
public interface PermissionEvaluator {

    boolean hasPermission(
              Authentication a, 
              Object subject,
              Object permission);

    boolean hasPermission(
              Authentication a, 
              Serializable id, 
              String type, 
              Object permission);
}
```

For the current example, it’s enough to use the first method. We already have the subject,
which in our case, is the value returned by the method. We also send the role
name ***'ROLE_admin'***, which, as defined by the example’s scenario, can access any
document. Of course, in our example, we could have directly used the name of the
role in the permission evaluator class and avoided sending it as a value of the
***hasPermission()*** object. Here, we only do the former for the sake of the example.
In a real-world scenario, which might be more complex, you have multiple methods,
and details needed in the authorization process might differ between each of them.
For this reason, you have a parameter that you can send the needed details for use in
the authorization logic from the method level.

For your awareness and to avoid confusion, I’d also like to mention that you don’t
have to pass the ***Authentication*** object. Spring Security automatically provides this
parameter value when calling the hasPermission() method. The framework knows
the value of the authentication instance because it is already in the ***SecurityContext***. In listing 16.15, you find the ***DocumentsPermissionEvaluator*** class,
which in our example implements the ***PermissionEvaluator*** contract to define the
custom authorization rule.
```java
@Component
public class DocumentsPermissionEvaluator
        //Implements the PermissionEvaluator contract
        implements PermissionEvaluator {

    @Override
    public boolean hasPermission(Authentication authentication,
                                 Object target,
                                 Object permission) {

        //Casts the target object to Document
        Document document = (Document) target;

        //The permission object in our case is the role name, so we cast it to a String.
        String p = (String) permission;

        //Checks if the authentication user has the role we got as a parameter
        boolean admin =
           authentication.getAuthorities()
           .stream()
           .anyMatch(a -> a.getAuthority().equals(p));

        //If admin or the authenticated user is the owner of the document, grants the permission
        return admin || document.getOwner().equals(authentication.getName());
    }

    @Override
    public boolean hasPermission(Authentication authentication,
                                 Serializable targetId,
                                 String targetType,
                                 Object permission) {
        //We don’t need to implement the second method because we don’t use it.
        return false;
    }
}
```
To make Spring Security aware of our new ***PermissionEvaluator*** implementation,we have to define a ***MethodSecurityExpressionHandler*** in the configuration class. The following listing presents how to define a ***MethodSecurityExpressionHandler*** to make the custom ***PermissionEvaluator*** known.
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig 
  extends GlobalMethodSecurityConfiguration {

  @Autowired
  private DocumentsPermissionEvaluator evaluator;

  @Override //Overrides the createExpressionHandler() method
  protected MethodSecurityExpressionHandler createExpressionHandler() {

    //Defines a default security expression handler to set up the custom permission evaluator
    var expressionHandler =
        new DefaultMethodSecurityExpressionHandler();

    //Sets up the custom permission evaluator
    expressionHandler.setPermissionEvaluator(
        evaluator);

    //Returns the custom expression handler
    return expressionHandler;
  }

  // Omitted definition of the UserDetailsService and PasswordEncoder beans
}
```
* ***NOTE*** We use here an implementation for ***MethodSecurityExpressionHandler*** named ***DefaultMethodSecurityExpressionHandler*** that Spring Security provides. You could as well implement a custom ***MethodSecurityExpressionHandler*** to define custom SpEL expressions you use to apply the authorization rules. You rarely need to do this in a real-world scenario, and for this reason, we won’t implement such a custom object in our examples. I just wanted to make you aware that this is possible.

I separate the definition of the ***UserDetailsService*** and ***PasswordEncoder*** to let
you focus only on the new code. In listing 16.17, you find the rest of the configuration
class. The only important thing to notice about the users is their roles. User Natalie is
an admin and can access any document. User Emma is a manager and can only access
her own documents.

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ProjectConfig 
  extends GlobalMethodSecurityConfiguration {

  @Autowired
  private DocumentsPermissionEvaluator evaluator;

  @Override
  protected MethodSecurityExpressionHandler createExpressionHandler() {
    var expressionHandler =
        new DefaultMethodSecurityExpressionHandler();

    expressionHandler.setPermissionEvaluator(evaluator);

    return expressionHandler;
  }

  @Bean
  public UserDetailsService userDetailsService() {
    var service = new InMemoryUserDetailsManager();

    var u1 = User.withUsername("natalie")
             .password("12345")
             .roles("admin")
             .build();

     var u2 = User.withUsername("emma")
              .password("12345")
              .roles("manager")
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
To test the application, we define an endpoint. The following listing presents this definition.
```java
@RestController
public class DocumentController {

    @Autowired
    private DocumentService documentService;

    @GetMapping("/documents/{code}")
    public Document getDetails(@PathVariable String code) {
        return documentService.getDocument(code);
    }
}
```
Let’s run the application and call the endpoint to observe its behavior. User Natalie
can access the documents regardless of their owner. User Emma can only access the
documents she owns. Calling the endpoint for a document that belongs to Natalie
and authenticating with the user "natalie", we use this command:

```bash
curl -u natalie:12345 http://localhost:8080/documents/abc123 |jq "."
```

The response body is
```json
{
"owner":"natalie"
}
```

Calling the endpoint for a document that belongs to Emma and authenticating with
the user "natalie", we use this command:

```bash
curl -u natalie:12345 http://localhost:8080/documents/asd555  |jq "."
```
The response body is
```json
{
"owner":"emma"
}
```
Calling the endpoint for a document that belongs to Emma and authenticating with
the user "emma", we use this command:
```bash
curl -u emma:12345 http://localhost:8080/documents/asd555 |jq "."
```

The response body is
```json
{
"owner":"emma"
}
```
Calling the endpoint for a document that belongs to Natalie and authenticating with
the user "emma", we use this command:
```bash
curl -u emma:12345 http://localhost:8080/documents/abc123  |jq "."
```

The response body is
```json
{
  "timestamp": "2021-06-09T08:05:59.351+00:00",
  "status": 403,
  "error": "Forbidden",
  "message": "",
  "path": "/documents/abc123"
}
```
In a similar manner, you can use the second ***PermissionEvaluator*** method to write
your authorization expression. The second method refers to using an identifier and
subject type instead of the object itself. For example, say that we want to change the
current example to apply the authorization rules before the method is executed,
using ***@PreAuthorize***. In this case, we don’t have the returned object yet. But instead
of having the object itself, we have the document’s code, which is its unique identifier.
Listing 16.19 shows you how to change the permission evaluator class to implement
this scenario. I separated the examples in a project named ssia-ch16-ex5, which you
can run individually.