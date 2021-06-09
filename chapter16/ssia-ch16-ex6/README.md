* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex6](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch16-ex6)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-16/128] (https://livebook.manning.com/book/spring-security-in-action/chapter-16/128) 

## Chapter 16 : GLOBAL METHOD SECURITY: PRE- AND POSTAUTHORIZATIONS 
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 


Global method security offers us three approaches to define the authorization rules that we discuss in this chapter:

* The pre-/postauthorization annotations
* The JSR 250 annotation, @RolesAllowed
* The @Secured annotation
 
### Using the @Secured and @RolesAllowed annotations

**Page 410**, 
Throughout this chapter, we discussed applying authorization rules with global
method security. We started by learning that this functionality is disabled by default
and that you can enable it using the ***@EnableGlobalMethodSecurity*** annotation
over the configuration class. Moreover, you must specify a certain way to apply the
authorization rules using an attribute of the ***@EnableGlobalMethodSecurity***
annotation. We used the annotation like this:

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

The prePostEnabled attribute enables the @PreAuthorize and @Post-
Authorize annotations to specify the authorization rules. The @EnableGlobal-
MethodSecurity annotation offers two other similar attributes that you can use to
enable different annotations. You use the jsr250Enabled attribute to enable the
@RolesAllowed annotation and the securedEnabled attribute to enable the
@Secured annotation. Using these two annotations, @Secured and @Roles-
Allowed, is less powerful than using @PreAuthorize and @PostAuthorize,
and the chances that you’ll find them in real-world scenarios are small. Even so, I’d
like to make you aware of both, but without spending too much time on the details.

You enable the use of these annotations the same way we did for preauthorization
and postauthorization by setting to true the attributes of the @EnableGlobal-
MethodSecurity. You enable the attributes that represent the use of one kind of
annotation, either @Secure or @RolesAllowed. You can find an example of how to
do this in the next code snippet:

```java
@EnableGlobalMethodSecurity(
    jsr250Enabled = true,
    securedEnabled = true
)
```
Once you’ve enabled these attributes, you can use the @RolesAllowed or
@Secured annotations to specify which roles or authorities the logged-in user needs
to have to call a certain method. The next code snippet shows you how to use the
@RolesAllowed annotation to specify that only users having the role ADMIN can
call the getName() method:
```java
@Service
public class NameService {
    @RolesAllowed("ROLE_ADMIN")
    public String getName() {
        return "Fantastico";
    }
}
```
Similarily, you can use the @Secured annotation instead of the @RolesAllowed
annotation, as the next code snippet presents:

```java
@Service
public class NameService {
    @Secured("ROLE_ADMIN")
    public String getName() {
        return "Fantastico";
    }
}
```

You can now test your example. The next code snippet shows how to do this:
```bash
curl -u emma:12345 http://localhost:8080/hello |jq "."
```

The response body is
```bash
Hello, Fantastico
```
To call the endpoint and authenticating with the user Natalie, use this command:
```bash
curl -u natalie:12345 http://localhost:8080/hello |jq "."
```
The response body is

```json
{
"status":403,
"error":"Forbidden",
"message":"Forbidden",
"path":"/hello"
}
```
You find a full example using the @RolesAllowed and @Secured annotations in
the project ssia-ch16-ex6.