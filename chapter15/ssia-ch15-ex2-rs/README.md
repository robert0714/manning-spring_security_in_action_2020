## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex2-rs)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex2-rs](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex2-rs) 
# Notes

### 15.2.3 Implementing a resource server that uses public keys

* **Page 375**, The authorization server uses the private key to sign the tokens, and the
resource server uses the public one to validate the signature. Mind, we use the keys
only to sign the tokens and not to encrypt them. 

The resource server needs to have the public key of the pair to validate the token’s
signature, so let’s add this key to the application.properties file. In section 15.2.1, you
learned how to generate the public key. The next code snippet shows the content of
my application.properites file:

```properties
server.port=9090
publicKey=-----BEGIN PUBLIC KEY-----MIIBIjANBghk…-----END PUBLIC KEY-----
```

I abbreviated the public key for better readability. The following listing shows you how
to configure this key in the configuration class of the resource server.

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Value("${publicKey}")
    private String publicKey;

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.tokenStore(tokenStore());
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        var converter = new JwtAccessTokenConverter();
        converter.setVerifierKey(publicKey);
        return converter;
    }
}
```

Of course, to have an endpoint, we also need to add the controller. The next code
snippet defines the controller:

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```
Let’s run and call the endpoint to test the resource server. Here’s the command:


```bash
export TOKEN=jwt_access_token

curl    -H "Authorization:Bearer $TOKEN" http://localhost:9090/hello
```

The response body is

```bash
Hello!
```