## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex4-as)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex4-as](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-exa-as) 


### 15.3 Adding custom details to the JWT
In this section, we discuss adding custom details to the JWT token. In most cases, you
need no more than what Spring Security already adds to the token. However, in realworld
scenarios, you’ll sometimes find requirements for which you need to add custom
details in the token. In this section, we implement an example in which you learn
how to change the authorization server to add custom details on the JWT and how to
change the resource server to read these details. If you take one of the tokens we generated
in previous examples and decode it, you see the defaults that Spring Security
adds to the token. The following listing presents these defaults.

As you can see in listing 15.8, by default, a token generally stores all the details needed
for Basic authorization. But what if the requirements of your real-world scenarios ask
for something more? Some examples might be

* You use an authorization server in an application where your readers review
books. Some endpoints should only be accessible for users who have given
more than a specific number of reviews.
* You need to allow calls only if the user authenticated from a specific time zone.
* Your authorization server is a social network, and some of your endpoints
should be accessible only by users having a minimum number of connections.


For my first example, you need to add the number of reviews to the token. For the second,
you add the time zone from where the client connected. For the third example,
you need to add the number of connections for the user. No matter which is your
case, you need to know how to customize JWTs.

#### 15.3.1 Configuring the authorization server to add custom details to tokens

In this section, we discuss the changes we need to make to the authorization server for
adding custom details to tokens. To make the example simple, I suppose that the
requirement is to add the time zone of the authorization server itself. 

* **Page 381**,The project I work on for this example is ssia-ch15-ex4-as. 

To add additional details to your token, you need to create an object of type ***TokenEnhancer***. The following listing defines the ***TokenEnhancer*** object I created for this example.

```java
public class CustomTokenEnhancer 
             //Implements the TokenEnhancer contract
             implements TokenEnhancer {

    // Overrides the enhance() method,which receives the current token and returns the enhanced token                 
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken,
                                     OAuth2Authentication oAuth2Authentication) {

        // Creates a new token object based on the one we received                                 
        var token = new DefaultOAuth2AccessToken(oAuth2AccessToken);

        //Defines as a Map the details we want to add to the token
        Map<String, Object> info =
                Map.of("generatedInZone", ZoneId.systemDefault().toString());

        //Adds the additional details to the token        
        token.setAdditionalInformation(info);

        //Returns the token containing the additional details
        return token;
    }
}
```

* **Page 382**, The ***enhance()*** method of a ***TokenEnhancer*** object receives as a parameter the
token we enhance and returns the “enhanced” token, containing the additional
details. For this example, I use the same application we developed in section 15.2 and
only change the ***configure()*** method to apply the token enhancer. The following
listing presents these changes.

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig
        extends AuthorizationServerConfigurerAdapter {

// Omitted code

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {

        //Defines a TokenEnhancerChain
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();

        //Adds our two token enhancer objects to a list
        var tokenEnhancers =
                List.of(new CustomTokenEnhancer(),
                        jwtAccessTokenConverter());

        //Adds the token enhancer’s list to the chain
        tokenEnhancerChain.setTokenEnhancers(tokenEnhancers);

        endpoints
          .authenticationManager(authenticationManager)
          .tokenStore(tokenStore())
          //Configures the token enhancer objects
          .tokenEnhancer(tokenEnhancerChain);
    }
}
```

As you can observe, configuring our custom token enhancer is a bit more complicated.
We have to create a chain of token enhancers and set the entire chain instead
of only one object, because the access token converter object is also a token enhancer.
If we configure only our custom token enhancer, we would override the behavior of
the access token converter. Instead, we add both in a chain of responsibilities, and we
configure the chain containing both objects.

Let’s start the authorization server, generate a new access token, and inspect it to
see how it looks. The next code snippet shows you how to call the /oauth/token endpoint
to obtain the access token:

```bash
curl -v -XPOST -u client:secret "http://localhost:8080/oauth/token?grant_type=password&username=john&password=12345&scope=read" |jq "."
```
or 

```bash
$ curl  -v -X POST -u client:secret  http://localhost:8080/oauth/token\?grant_type=password\&username=john\&password=12345\&scope=read | jq "."
```
or 

```bash
\>curl -v -X POST --basic -u client:secret \
-H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" \
-k -d "grant_type=password&username=john&password=12345&scope=read" \
http://localhost:8080/oauth/token | jq "."
```
 

The response body is

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJqb2huIiwic2NvcGUiOlsicmVhZCJdLCJnZW5lcmF0ZWRJblpvbmUiOiJBc2lhL1RhaXBlaSIsImV4cCI6MTYyMzA5OTkyNCwiYXV0aG9yaXRpZXMiOlsicmVhZCJdLCJqdGkiOiIxYWJmOTdkYS0xNmZmLTRmODItYjVlMC1hYzk1YzI1YzdmMDEiLCJjbGllbnRfaWQiOiJjbGllbnQifQ.Pfzg_vzNtjthXKsDUBV4zib9z9v3I4lgd9HW9svh6kScHFN9Jla2admk8UyqAbsUQTmX7K8N3bnxMZb8_XiCoUpGabgHQpaEwr80FciKAGw6xt9NUPD5y7mtex8OEbQcshJswRA6b1IC5K7AUQKPKELZIozEuf6uU1w87Ck-IOMt7L4eVAQrwnjQlg0RpwWMyDvClN0KXzsXZwBrInWrhwu7jl0YGFLI93UDwqd6ylkoPGCs23T8Pqaj_ZGzQwQ_HQzHm_uS0t-ByOVGP3XrxwZa9bThwWDvCVlyjbKvq4BZDr-ChFJgmRku_h8gHmLqGq9fttQtZSWkEODthbInwg",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJqb2huIiwic2NvcGUiOlsicmVhZCJdLCJhdGkiOiIxYWJmOTdkYS0xNmZmLTRmODItYjVlMC1hYzk1YzI1YzdmMDEiLCJnZW5lcmF0ZWRJblpvbmUiOiJBc2lhL1RhaXBlaSIsImV4cCI6MTYyNTY0ODcyNCwiYXV0aG9yaXRpZXMiOlsicmVhZCJdLCJqdGkiOiJhMWI2ODNkZC02OWI1LTRiNzAtOTFmYS02MjNiMGNkMTBlYmQiLCJjbGllbnRfaWQiOiJjbGllbnQifQ.LlQUX2Y_99nEXnQIbZ-5X6-rv5DrM9rJawj55qSQUjji_a0ZlqrPHDTcoetKX-j5cv2cNuWLeysnvWcHG7JM3wS-8Q93hKADFKeDFX8eR9AMM-ybt1v4ELVhcRFbwzNaLWh5onVnr4YSbjW7tQGyd6_ccWfQSRHLaQJMRlTrH47TRwso6Tk6XoOYoNkWgo7gtD0wih3UUJEmd5TjeLA7N7e8H8fZwFRaby0PGqExoSG6dDM9zMJoLMiF4SnSapetPpMrLGpec2qHeeKptpxLoAs4KK7_RHgxZcI-BjEjd3PFkPDYUTpGjZBSP0xN5OLew72Ds5T4Vb6rKdwzF54OKQ",
  "expires_in": 43199,
  "scope": "read",
  "generatedInZone": "Asia/Taipei",
  "jti": "1abf97da-16ff-4f82-b5e0-ac95c25c7f01"
}
```
If you decode the token, you can see that its body looks like the one presented in listing
15.11. You can further observe that the framework adds the custom details, by
default, in the response as well. But I recommend you always refer to any information
from the token. Remember that by signing the token, we make sure that if anybody
alters the content of the token, the signature doesn’t get validated. This way, we know
that if the signature is correct, nobody changed the contents of the token. You don’t
have the same guarantee on the response itself.

```json
{
  "user_name": "john",
  "scope": [
    "read"
  ],
  "generatedInZone": "Asia/Taipei",   //The custom details we added appear in the token’s body.
  "exp": 1623099924,
  "authorities": [
    "read"
  ],
  "jti": "1abf97da-16ff-4f82-b5e0-ac95c25c7f01",
  "client_id": "client"
}
```