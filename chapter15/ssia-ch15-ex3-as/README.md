## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex3-as)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex3-as](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex3-as) 
# Notes

### 15.2.4 Using an endpoint to expose the public key

* **Page 377**, In this section, we discuss a way of making the public key known to the resource
server—the authorization server exposes the public key. In the system we implemented
in section 15.2, we use private-public key pairs to sign and validate tokens. We
configured the public key at the resource server side. The resource server uses the
public key to validate JWTs. But what happens if you want to change the key pair? It is
a good practice not to keep the same key pair forever, and this is what you learn to
implement in this section. Over time, you should rotate the keys! This makes your system
less vulnerable to key theft (figure 15.7).

Up to now, we have configured the private key on the authorization server side and
the public key on the resource server side (figure 15.7). Being set in two places makes
the keys more difficult to manage. But if we configure them on one side only, you
could manage the keys easier. The solution is moving the whole key pair to the authorization
server side and allowing the authorization server to expose the public keys
with an endpoint (figure 15.8).

We work on a separate application to prove how to implement this configuration with
Spring Security. You can find the authorization server for this example in project ssiach15-
ex3-as and the resource server of this example in project ssia-ch15-ex3-rs.

For the authorization server, we keep the same setup as for the project we developed
in section 15.2.3. We only need to make sure we make accessible the endpoint,
which exposes the public key. Yes, Spring Boot already configures such an endpoint,
but it’s just that. By default, all requests for it are denied. We need to override the endpoint’s
configuration and allow anyone with client credentials to access it. In listing
15.7, you find the changes you need to make to the authorization server’s configuration
class. These configurations allow anyone with valid client credentials to call the
endpoint to obtain the public key.
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig
        extends AuthorizationServerConfigurerAdapter {
     
    // Omitted code

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("client")
                .secret("secret")
                .authorizedGrantTypes("password", "refresh_token")
                .scopes("read")
             .and()
                .withClient("resourceserver")
                .secret("resourceserversecret");
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security.tokenKeyAccess("isAuthenticated()");
    }
}
```
You can start the authorization server and call the /oauth/token_key endpoint to
make sure you correctly implement the configuration. The next code snippet shows
you the cURL call:

```bash
curl -u resourceserver:resourceserversecret http://localhost:8080/oauth/token_key
```

The response body is

```json
{
"alg":"SHA256withRSA",
"value": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAhORXDLLrdozoNFsIyaY48NwZaSP2f94JobhEV1CYw4ImqOH7My+odLyI063aDu0HLOeV0yGUj+oZVRNM/8Y5Qhl/fIRZeCtCDVybT7yJdBz/WvzAulfI4aGWSdjGUCwS88z5Af2BJUKGv7bkwRtaF+btTq8OEC/ke0GKOkWh2nGDKeHK645OOv59qLEoa8v6Ns/SveQCfB93Zx7V+utuV6Xjp8jqUN2X5MtM9+AQ2eihhTuLGCfZm0c51QXUihXYx4GH4kLMOULOXvI3uCSdrgkF6heTFRhN6sPCex1TEWB1mbGpCDGkRZ6Q0IeSKb5fcuW+LhUqfTwCKz6cvXT6kwIDAQAB\n-----END PUBLIC KEY-----"

}
```

For the resource server to use this endpoint and obtain the public key, you only need
to configure the endpoint and the credentials in its properties file. The next code
snippet defines the application.properties file of the resource server:

```properties
server.port=9090
security.oauth2.resource.jwt.key-uri=http://localhost:8080/oauth/token_key
security.oauth2.client.client-id=resourceserver
security.oauth2.client.client-secret=resourceserversecret
```

Because the resource server now takes the public key from the ***/oauth/token_key***
endpoint of the authorization server, you don’t need to configure it in the resource
server configuration class. The configuration class of the resource server can remain
empty, as the next code snippet shows:

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig
    extends ResourceServerConfigurerAdapter {
}
```
You can start the resource server as well now and call the /hello endpoint it exposes to
see that the entire setup works as expected. The next code snippet shows you how to
call the /hello endpoint using cURL. Here, you obtain a token as we did in section
15.2.3 and use it to call the test endpoint of the resource server:





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


```bash
export TOKEN=jwt_access_token

curl    -H "Authorization:Bearer $TOKEN" http://localhost:9090/hello
```

The response body is

```bash
Hello!
```