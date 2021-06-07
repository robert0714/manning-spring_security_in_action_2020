## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex1-rs-migration)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex1-rs-migration](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex1-rs-migration)
* [https://livebook.manning.com/book/spring-security-in-action/chapter-15/49](https://livebook.manning.com/book/spring-security-in-action/chapter-15/49)

# Notes

### Using symmetric keys without the Spring Security OAuth project

* **Page 367**, The next code snippet shows you how to call the endpoint using cURL:

```bash
export TOKEN=jwt_access_token

curl    -H "Authorization:Bearer $TOKEN" http://localhost:9090/hello
```

The response body is

```bash
Hello!
```
As we discussed in chapter 14, you can also configure your resource server to use
JWTs with ***oauth2ResourceServer()*** . As we mentioned, this approach is more
advisable for future projects, but you might find it in existing apps. You, therefore,
need to know this approach for future implementations and, of course, if you want to
migrate an existing project to it. The next code snippet shows you how to configure
JWT authentication using symmetric keys without the classes of the Spring Security
OAuth project:

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Value("${jwt.key}")
    private String jwtKey;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
            .and()
                .oauth2ResourceServer(c -> c.jwt( jwt -> {
                    jwt.decoder(jwtDecoder());
                }));
    }
    // Omitted code
}
```
As you can see, this time I use the ***jwt()*** method of the Customizer object sent
as a parameter to ***oauth2ResourceServer()***. Using the ***jwt()*** method, we configure
the details needed by our app to validate tokens. In this case, because we are
discussing validation using symmetric keys, I create a ***JwtDecoder*** in the same
class to provide the value of the symmetric key. The next code snippet shows how I
set this decoder using the ***decoder()*** method:
```java
    @Bean
    public JwtDecoder jwtDecoder() {
        byte [] key = jwtKey.getBytes();
        SecretKey originalKey = new SecretKeySpec(key, 0, key.length, "AES");

        NimbusJwtDecoder jwtDecoder =
                NimbusJwtDecoder.withSecretKey(originalKey)
                .build();

        return jwtDecoder;
    }
}
```
The elements we configured are the same! It’s only the syntax that differs, if you
choose to use this approach to set up your resource server.