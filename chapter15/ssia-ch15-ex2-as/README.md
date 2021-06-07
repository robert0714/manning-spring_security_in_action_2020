## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex2-as)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex2-as](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex2-as)
* [https://livebook.manning.com/book/spring-security-in-action/chapter-15/88](https://livebook.manning.com/book/spring-security-in-action/chapter-15/88)

# Notes

### 15.2.1 Generating the key pair

* **Page 372**, For OpenSSL, you need to download
it from https://www.openssl.org/. If you use Git Bash, which comes with
OpenSSL, you don’t need to install it separately. I always prefer using Git Bash for
these operations because it doesn’t require me to install these tools separately. Once
you have the tools, you need to run two commands to
* Generate a private key
* Obtain the public key for the previously generated private key

#### GENERATING A PRIVATE KEY
To generate a private key, run the ***keytool*** command in the next code snippet. It
generates a private key in a file named ssia.jks. I also use the password “ssia123” to protect
the private key and the alias “ssia” to give the key a name. In the following command,
you can see the algorithm used to generate the key, RSA:

```bash
keytool -genkeypair -alias ssia -keyalg RSA -keypass ssia123 -keystore ssia.jks -storepass ssia123
```
#### OBTAINING THE PUBLIC KEY
To get the public key for the previously generated private key, you can run the keytool
command:

```bash
keytool -list -rfc --keystore ssia.jks | openssl x509 -inform pem -pubkey
```
You are prompted to enter the password used when generating the public key; in my
case, ssia123. Then you should find the public key and a certificate in the output.
(Only the value of the key is essential for us for this example.) This key should look
similar to the next code snippet:

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAijLqDcBHwtnsBw+WFSzGVkjtCbO6NwKlYjS2PxE114XWf9H2j0dWmBu7NK+lV/JqpiOi0GzaLYYf4XtCJxTQDD2CeDUKczcd+fpnppripN5jRzhASJpr+ndj8431iAG/vXrmZt3jLD3v6nwLDxzpJGmVWzcV/OBXQZkd1LHOK5LEG0YCQ0jAU3ON7OZAnFn/DMJyDCky994UtaAYyAJ7mr7IO1uHQxsBg7SiQGpApgDEK3Ty8gaFuafnExsYD+aqua1Ese+pluYnQxuxkk2Ycsp48qtUv1TWp+TH3kooTM6eKcnpSweaYDvHd/ucNg8UDNpIqynM1eS7KpffKQmDwIDAQAB
-----END PUBLIC KEY-----
```

That’s it! We have a private key we can use to sign JWTs and a public key we can use to
validate the signature. Now we just have to configure these in our authorization and
resource servers.

### 15.2.2 Implementing an authorization server that uses private keys
* **Page 373**, In the application.properties
file, I store the filename, the alias of the key, and the password I used to protect
the private key when I generated the password. We need these details to configure
***JwtTokenStore***. The next code snippet shows you the contents of my application.
properties file:

```properties 
password=ssia123
privateKey=ssia.jks
alias=ssia
```

Compared with the configurations we did for the authorization server to use a symmetric
key, the only thing that changes is the definition of the ***JwtAccessToken-Converter*** object. We still use ***JwtTokenStore***. If you remember, we used
***JwtAccessTokenConverter*** to configure the symmetric key in section 15.1. We use the same ***JwtAccessTokenConverter*** object to set up the private key. The following
listing shows the configuration class of the authorization server.

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig
        extends AuthorizationServerConfigurerAdapter {

    @Value("${password}")
    private String password;

    @Value("${privateKey}")
    private String privateKey;

    @Value("${alias}")
    private String alias;

    @Autowired
    private AuthenticationManager authenticationManager;

   // Omitted code

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        var converter = new JwtAccessTokenConverter();

        KeyStoreKeyFactory keyStoreKeyFactory =
                new KeyStoreKeyFactory(
                        new ClassPathResource(privateKey),
                        password.toCharArray());
        converter.setKeyPair(keyStoreKeyFactory.getKeyPair(alias));

        return converter;
    }
}
```

You can now start the authorization server and call the ***/oauth/token*** endpoint to
generate a new access token. Of course, you only see a normal JWT created, but the
difference is now that to validate its signature, you need to use the public key in the
pair. By the way, don’t forget the token is only signed, not encrypted. The next code
snippet shows you how to call the ***/oauth/token*** endpoint:


```bash
\>curl -v -X POST --basic -u client:secret \
-H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" \
-k -d "grant_type=password&username=john&password=12345&scope=read" \
http://localhost:8080/oauth/token | jq "."
```
or 

```bash
$ curl -u client:secret -X POST http://localhost:8080/oauth/token\?grant_type=password\&username=john\&password=12345\&scope=read | jq "."
```


The response body is

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjMwMjIwNDUsInVzZXJfbmFtZSI6ImpvaG4iLCJhdXRob3JpdGllcyI6WyJyZWFkIl0sImp0aSI6Ijk2MWM2MmQzLTJjMDEtNGM3Yy05ZGE4LTUyNDhmZWRhZmE4OCIsImNsaWVudF9pZCI6ImNsaWVudCIsInNjb3BlIjpbInJlYWQiXX0.PxZ50BaqmKcQUxos_hKNsLY57b4eV3h7rYmcUfBOteIKfKrySjbLtyEYztjrx5n8OWUovPNhVPpe_ufEAZThDZ8gfchU3TQYWuUGHSekdFyXDkeiFMitfwWwIPgaSCIlfsK6KKnd1V_XGX3mwfNTkcKjoFq71ZESNlZCBRliyen89z6n3V_GvBBNy5h6Nz4BLKaAxKIZL7H31b5JFmtOelU3FLr9vpbH5veC38bUo0wSzZLMPb0PwAdMGK7Tn55EM9GmmYpJoPto3Vnus_pq-yRNkgZuhM_Kaj8FRVrJf24ugYzIiZcWDurektQGLwAVmoASaVfZH6DCK-1C8Tb73g",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJqb2huIiwic2NvcGUiOlsicmVhZCJdLCJhdGkiOiI5NjFjNjJkMy0yYzAxLTRjN2MtOWRhOC01MjQ4ZmVkYWZhODgiLCJleHAiOjE2MjU1NzA4NDUsImF1dGhvcml0aWVzIjpbInJlYWQiXSwianRpIjoiNmU3N2UyMTctNWZjOC00MmUyLWJmZTktZWRmNTA5MDQzNTU4IiwiY2xpZW50X2lkIjoiY2xpZW50In0.b-IBfdeyy1HckWnJzZ5F398dAQ5Fi_nXWdohbbekJzsQ0j-IyMBtwapUJqdwgoX2HHx52VjMK6DR4vY1l1PHWncbdHujJPDTnToSnj6-CAA9VQPlYCnzpwcDCk8JUCtYqiuHa3gk0RvovH2nmFvL2J03bGlgr9BVEA4vovJsaYz2-FtSxFxE-vIzgP83wu9TlP8_oDkLb2mnvbYPQtst1xEDgaV94pdFpBL5T6Cc3Ut2b0Hp1SkWfr7abILG3hpuH6KTKOIGiYAARu2n8cF4o6IO-pRQTzj5K3xkkdFbiZIWYtuNF3Tkv8QHgFbsNREwAVVhT2r3x8T8n6Xg7Q0BIg",
  "expires_in": 43199,
  "scope": "read",
  "jti": "961c62d3-2c01-4c7c-9da8-5248fedafa88"
}
```