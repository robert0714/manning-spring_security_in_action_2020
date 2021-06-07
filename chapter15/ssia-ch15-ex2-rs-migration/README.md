## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex2-rs)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex2-rs](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex2-rs) 
# Notes

### 15.2.3 Implementing a resource server that uses public keys

#### Using asymmetric keys without the Spring Security OAuth project
* **Page 376**, In this sidebar, we discuss the changes you need to make to migrate your resource
server using the Spring Security OAuth project to a simple Spring Security one if the
app uses asymmetric keys for token validation. Actually, using asymmetric keys
doesn’t differ too much from using a project with symmetric keys. The only change is
the ***JwtDecoder*** you need to use. In this case, instead of configuring the symmetric
key for token validation, you need to configure the public part of the key pair. The following
code snippet shows how to do this:

```java
    @Bean
    public JwtDecoder jwtDecoder() {
        try {
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            var key = Base64.getDecoder().decode(publicKey);

            var x509 = new X509EncodedKeySpec(key);
            var rsaKey = (RSAPublicKey) keyFactory.generatePublic(x509);
            return NimbusJwtDecoder.withPublicKey(rsaKey).build();
        } catch (Exception e) {
            throw new RuntimeException("Wrong public key");
        }
    }
```

Once you have a JwtDecoder using the public key to validate tokens, you need to
set up the decoder using the ***oauth2ResourceServer()*** method. You do this like
a symmetric key. The next code snippet shows how to do this. You find this example
implemented in the project ssia-ch15-ex2-rs-migration.

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Value("${publicKey}")
    private String publicKey;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.oauth2ResourceServer(
                c -> c.jwt(
                        j -> j.decoder(jwtDecoder())
                )
        );

        http.authorizeRequests()
                .anyRequest().authenticated();
    }
    // Omitted code
}
```
