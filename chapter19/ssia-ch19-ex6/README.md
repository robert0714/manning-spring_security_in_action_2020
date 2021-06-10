* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch19-ex6](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch19-ex6)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-19/134](https://livebook.manning.com/book/spring-security-in-action/chapter-19/134) 

# Chapter 19 : SPRING SECURITY FOR REACTIVE APPS
![cover](../../cover.webp) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 


## 19.4 Reactive apps and OAuth 2
You’re probably wondering by now if we could use reactive applications in a system
designed over the OAuth 2 framework. In this section, we discuss implementing a
resource server as a reactive app. You learn how to configure your reactive application
to rely on an authentication approach implemented over OAuth 2. Because using
OAuth 2 is so common nowadays, you might encounter requirements where your
resource server application needs to be designed as a reactive server. I created a new
project named ssia-ch19-ex6, and we’ll implement a reactive resource server application.
You need to add the dependencies in pom.xml, as the next code snippet illustrates:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

We need an endpoint to test the application, so we add a controller class. The next
code snippet presents the controller class:
```java
@RestController
public class HelloController {
  @GetMapping("/hello")
  public Mono<String> hello() {
  return Mono.just("Hello!");
  }
}
```

And now, the most important part of the example: the security configuration. For this
example, we configure the resource server to use the public key exposed by the authorization
server for token signature validation. This approach is the same as in chapter
18, when we used Keycloak as our authorization server. I actually used the same configured
server for this example. You can choose to do the same, or you can implement
a custom authorization server, as we discussed in chapter 13.

To configure the authentication method, we use the ***SecurityWebFilterChain***,
as you learned about in section 19.3. But instead of using the ***httpBasic()*** method,
we call the ***oauth2ResourceServer()*** method. Then, by calling the ***jwt()*** method,
we define the kind of token we use, and by using a ***Customizer*** object, we specify the
way the token signature is validated. In listing 19.12, you find the definition of the
configuration class.

Listing 19.12 The configuration class
```java
@Configuration
public class ProjectConfig {

  @Value("${jwk.endpoint}")
  private String jwkEndpoint;

  @Bean
  public SecurityWebFilterChain securityWebFilterChain(
    ServerHttpSecurity http) {
   
    return http.authorizeExchange()
                  .anyExchange().authenticated()
               .and().oauth2ResourceServer()     //Configures the resource server authentication method
                  .jwt(jwtSpec -> {              //Specifies the way the token is validated
                    jwtSpec.jwkSetUri(jwkEndpoint);
                  })
               .and().build();

    }
}
```

In the same way, we could’ve configured the public key instead of specifying an URI
where the public key is exposed. The only change was to call the ***publicKey()*** method of the ***jwtSpec*** instance and provide a valid public key as a parameter. You can use any of the approaches we discussed in chapters 14 and 15, where we analyzed in detail approaches for the resource server to validate the access token.

Next, we change the application.properties file to add the value for the URI where the key set is exposed, as well as change the server port to 9090. This way, we allow Keycloak to run on 8080. In the next code snippet, you find the contents of the application.properties file:
```properties
server.port=9090
jwk.endpoint=http://localhost:8080/auth/realms/master/protocol/openid-connect/certs
```

Let’s run and prove the app has the expected behavior that we want. We generate an
access token using the locally installed Keycloak server:
```bash
curl -XPOST 'http://localhost:8080/auth/realms/master/protocol/openid-connect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=bill' \
--data-urlencode 'password=12345' \
--data-urlencode 'client_id=fitnessapp' \
--data-urlencode 'scope=fitnessapp'
```

In the HTTP response body, we receive the access token as presented here:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI…",
  "expires_in": 6000,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5c… ",
  "token_type": "bearer",
  "not-before-policy": 0,
  "session_state": "610f49d7-78d2-4532-8b13-285f64642caa",
  "scope": "fitnessapp"
}
```

Using the access token, we call the /hello endpoint of our application like this:
```bash
curl -H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJMSE9zT0VRSmJuTmJVbjhQbVpYQTlUVW9QNTZoWU90YzNWT2swa1V2ajVVIn…' \
'http://localhost:9090/hello'
```

The response body is
```
Hello!
```