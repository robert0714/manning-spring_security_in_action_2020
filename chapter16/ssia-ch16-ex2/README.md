* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex2](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex2)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-16/30] (https://livebook.manning.com/book/spring-security-in-action/chapter-16/30) 

## Chapter 16 : GLOBAL METHOD SECURITY: PRE- AND POSTAUTHORIZATIONS 
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 


Global method security offers us three approaches to define the authorization rules that we discuss in this chapter:

* The pre-/postauthorization annotations
* The JSR 250 annotation, @RolesAllowed
* The @Secured annotation
 
### 16.2 Applying preauthorization for authorities and roles

**Page 395**, Let’s extend our example to prove how you can use the values of the method parameters
to define the authorization rules (figure 16.6). You find this example in the project
named ssia-ch16-ex2.

For this project, I defined the same ***ProjectConfig*** class as in our first example so
that we can continue working with our two users, Emma and Natalie. The endpoint
now takes a value through a path variable and calls a service class to obtain the “secret
names” for a given username. Of course, in this case, the secret names are just an
invention of mine referring to a characteristic of the user, which is something that not
everyone can see. I define the controller class as presented in the next listing.

```java
@RestController
public class HelloController {

    //From the context, injects an instance of the service class that defines the protected method
    @Autowired
    private NameService nameService;

    //Defines an endpoint that takes a value from a path variable
    @GetMapping("/secret/names/{name}")
    public List<String> names(@PathVariable String name) {
        //Calls the protected method to obtain the secret names of the users
        return nameService.getSecretNames(name);
    }
}
```

Now let’s take a look at how to implement the ***NameService*** class in listing 16.6. The
expression we use for authorization now is #name ***== authentication.principal.username***. In this expression, we use ***#name*** to refer to the value of the ***getSecretNames()*** method parameter called ***name***, and we have access directly to the authentication object that we can use to refer to the currently authenticated user. The expression we use indicates that the method can be called only if the authenticated user’s username is the same as the value sent through the method’s parameter. In other words, a user can only retrieve its own secret names.

((Listing 16.6 The NameService class defines the protected method))
```java
@Service
public class NameService {

    private Map<String, List<String>> secretNames = Map.of(
            "natalie", List.of("Energico", "Perfecto"),
            "emma", List.of("Fantastico"));

    @PreAuthorize("#name == authentication.principal.username")
    public List<String> getSecretNames(String name) {
        return secretNames.get(name);
    }
}
```
We start the application and test it to prove it works as desired. The next code snippet shows you the behavior of the application when calling the endpoint, providing the value of the path variable equal to the name of the user:

```bash
curl -u emma:12345 http://localhost:8080/secret/names/emma
```
The response body is
```bash
["Fantastico"]
```
When authenticating with the user Emma, we try to get Natalie’s secret names. The
call doesn’t work:

```bash
curl -u emma:12345 http://localhost:8080/secret/names/natalie | jq "."
```

The response body is
```json
{
"status":403,
"error":"Forbidden",
"message":"Forbidden",
"path":"/secret/names/natalie"
}
```

The user Natalie can, however, obtain her own secret names. The next code snippet
proves this:

```bash
curl -u natalie:12345 http://localhost:8080/secret/names/natalie
```

The response body is

```json
["Energico","Perfecto"]
```

* ***NOTE***    Remember, you can apply global method security to any layer of your
application. In the examples presented in this chapter, you find the authorization
rules applied for methods of the service classes. But you can apply
authorization rules with global method security in any part of your application:
repositories, managers, proxies, and so on.