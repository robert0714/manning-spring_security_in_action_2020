## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex4-rs)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex4-as](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-exa-rs) 


### 15.3.2 Configuring the resource server to read the custom details of a JWT
* **Page 383**, In this section, we discuss the changes we need to do to the resource server to read the
additional details we added to the JWT. Once you change your authorization server to
add custom details to a JWT, you’d like the resource server to be able to read these
details. The changes you need to do in your resource server to access the custom
details are straightforward. You find the example we work on in this section in the ssiach15-
ex4-rs project.

We discussed in section 15.1 that ***AccessTokenConverter*** is the object that converts
the token to an ***Authentication***. This is the object we need to change so that it
also takes into consideration the custom details in the token. Previously, you created a
bean of type ***JwtAccessTokenConverter***, as shown in the next code snippet:

```java
    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        var converter = new AdditionalClaimsAccessTokenConverter();
        converter.setVerifierKey(publicKey);
        return converter;
    }
``` 

We used this token to set the key used by the resource server for token validation. We
create a custom implementation of ***JwtAccessTokenConverter***, which also takes
into consideration our new details on the token. The simplest way is to extend this
class and override the ***extractAuthentication()*** method. This method converts
the token in an ***Authentication*** object. The next listing shows you how to implement
a custom ***AcessTokenConverter***.

```java
public class AdditionalClaimsAccessTokenConverter
        extends JwtAccessTokenConverter {

    @Override
    public OAuth2Authentication extractAuthentication(Map<String, ?> map) {

        //Applies the logic implemented by the JwtAccessTokenConverter class and gets the initial authentication object
        var authentication = super.extractAuthentication(map);

        //Adds the custom details to the authentication
        authentication.setDetails(map);

        //Returns the authentication object
        return authentication;
    }
}
```

In the configuration class of the resource server, you can now use the custom access token converter. The next listing defines the AccessTokenConverter bean in the configuration class.

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
   //Omitted code

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {

        //Creates an instance of the new AccessTokenConverter object
        var converter = new AdditionalClaimsAccessTokenConverter();

        converter.setVerifierKey(publicKey);
        return converter;
    }
}
```

An easy way to test the changes is to inject them into the controller class and return them in the HTTP response. Listing 15.14 shows you how to define the controller class.

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(OAuth2Authentication authentication) {

      //Gets the extra details that were added to the Authentication  object 
        OAuth2AuthenticationDetails details =
                (OAuth2AuthenticationDetails) authentication.getDetails();

        //Returns the details in the HTTP response
        return "Hello! " + details.getDecodedDetails();
    }
}
```
You can now start the resource server and test the endpoint with a JWT containing
custom details. The next code snippet shows you how to call the /hello endpoint and
the results of the call. The getDecodedDetails() method returns a Map containing
the details of the token. In this example, to keep it simple, I directly printed the entire value returned by getDecodedDetails(). If you need to use only a specific value, you can inspect the returned Map and obtain the desired value using its key.

```bash
export TOKEN=jwt_access_token

curl -H "Authorization:Bearer $TOKEN" http://localhost:9090/hello

curl -H "Authorization:Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp… " http://localhost:9090/hello
```
The response body is

```bash
Hello! {user_name=john, scope=[read], generatedInZone=Asia/Taipei, exp=1623099924, authorities=[read], jti=1abf97da-16ff-4f82-b5e0-ac95c25c7f01, client_id=client}
```

You can spot in the response the new attribute generatedInZone=Asia/Taipei. 