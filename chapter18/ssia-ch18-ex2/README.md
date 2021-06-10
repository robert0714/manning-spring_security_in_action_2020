# Chapter 18 : HANDS-ON: AN OAUTH 2 APPLICATION
![cover](../../cover.webp) 
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-18](https://livebook.manning.com/book/spring-security-in-action/chapter-18) 

[Amazon](https://www.amazon.com/Spring-Security-Action-Laurentiu-Spilca/dp/1617297739) | [Manning](https://www.manning.com/books/spring-security-in-action) | [YouTube](https://t.co/4Or4P12LH2?amp=1) | [Books](https://laurspilca.com/books/) | [livebook](https://livebook.manning.com/book/spring-security-in-action) 


In chapters 12 through 15, we discussed in detail how an OAuth 2 system works and how you implement one with Spring Security. We then changed the subject and in chapters 16 and 17, you learned how to apply authorization rules at any layer of your application using global method security. In this chapter, we’ll combine these two essential subjects and apply global method security within an OAuth 2 resource server.

Besides defining authorization rules at different layers of our resource server implementation, you’ll also learn how to use a tool named Keycloak as the authorization server for your system. The example we’ll work on this chapter is helpful for the following reasons:

* Systems often use third-party tools such as Keycloak in real-world implementations to define an abstraction layer for authentication. There’s a good chance you need to use Keycloak or a similar third-party tool in your OAuth 2 implementation. You’ll find many possible alternatives to ***Keycloak*** like ***Okta***,***Auth0***, and ***LoginRadius***. This chapter focuses on a scenario in which you need to use such a tool in the system you develop.
* In real-world scenarios, we use authorization applied not only for the endpoints but also for other layers of the application. And this also happens for an OAuth 2 system.
* You’ll gain a better understanding of the big picture of the technologies and approaches we discuss. To do this, we’ll once again use an example to reinforce what you learned in chapters 12 through 17.

Let’s dive into the next section and find out the scenario of the application we’ll
implement in this hands-on chapter.

## Using OAuth 2 web security expressions
[https://livebook.manning.com/book/spring-security-in-action/chapter-18/150](https://livebook.manning.com/book/spring-security-in-action/chapter-18/150)
Spring Security allows us to easily refer to authorities, roles, and username. But with
OAuth 2 resource servers, we sometimes need to refer to other values specific to this
protocol, like client roles or scope. While the JWT token contains these details, we
can’t access them directly with SpEL expressions and quickly use them in the authorization
rules we define.

Fortunately, Spring Security offers us the possibility to enhance the SpEL expression
by adding conditions related directly to OAuth 2. To use such SpEL expressions, we
need to configure a ***SecurityExpressionHandler***. The ***SecurityExpressionHandler*** implementation that allows us to enhance our authorization expression
with OAuth 2–specific elements is ***OAuth2WebSecurityExpressionHandler***.
To configure this, we change the configuration class as presented in the next code
snippet:
```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig 
  extends ResourceServerConfigurerAdapter {

  // Omitted code

  public void configure(ResourceServerSecurityConfigurer resources) {
    resources.tokenStore(tokenStore());
    resources.resourceId(claimAud);
    resources.expressionHandler(handler());
  }

  @Bean
  public SecurityExpressionHandler<FilterInvocation> handler() {
    return new OAuth2WebSecurityExpressionHandler();
  }
}
```

With such an expression handler, you can write an expression like this:
```java
@PreAuthorize(
  "#workout.user == authentication.name and
   #oauth2.hasScope('fitnessapp')")
public void saveWorkout(Workout workout) {
  workoutRepository.save(workout);
}
```
Observe the condition I added to the ***@PreAuthorize*** annotation that checks for
the client scope ***#oauth2.hasScope('fitnessapp')***. You can now add such
expressions to be evaluated by the ***OAuth2WebSecurityExpressionHandler***
we added to our configuration. You can also use the clientHasRole() method in
the expression instead of ***hasScope()*** to test if the client has a specific role. Note
that you can use client roles with the client credentials grant type. To avoid mixing
this example with the current hands-on project, I separated it into a project named
ssia-ch18-ex2.

## 18.4 Testing the application
[https://livebook.manning.com/book/spring-security-in-action/chapter-18/99](https://livebook.manning.com/book/spring-security-in-action/chapter-18/99)
Now that we have a complete system, we can run some tests to prove it works as
desired (figure 18.28). In this section, we run both our authorization and resource
servers and use cURL to test the implemented behavior.


| ![content](CH18_F28_Spilca.png)|
|-----------|
|Figure 18.28 You got to the top! This is the last step of implementing the hands-on application for this chapter. Now we can test the system and prove that what we configured and implemented works as expected.|

The scenarios we need to test are the following:
* A client can add a workout only for the authenticated user
* A client can only retrieve their own workout records
* Only admin users can delete a workout

In my case, the Keycloak authorization server runs on port 8080, and the resource server I configured in the application.properties file runs on port 9090. You need to make sure you make calls to the correct component by using the ports you configured. Let’s take each of the three test scenarios and prove the system is correctly secured.

### 18.4.1 Proving an authenticated user can only add a record for themself

According to the scenario, a user can only add a record for themself. In other words, if I authenticate as Bill, I shouldn’t be able to add a workout record for Rachel. To prove this is the app’s behavior, we call the authorization server and issue a token for one of the users, say, Bill. Then we try to add both a workout record for Bill and a
workout record for Rachel. We prove that Bill can add a record for himself, but the
app doesn’t allow him to add a record for Rachel. To issue a token, we call the authorization
server as presented in the next code snippet:

```bash
curl -XPOST 'http://localhost:8080/auth/realms/master/protocol/openidconnect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=bill' \
--data-urlencode 'password=12345' \
--data-urlencode 'scope=fitnessapp' \
--data-urlencode 'client_id=fitnessapp'
```

Among other details, you also get an access token for Bill. I truncated the value of the
token in the following code snippet to make it shorter. The access token contains all
the details needed for authorization, like the username and the authorities we added
previously by configuring Keycloak in section 18.1.

```json
{
"access_token": "eyJhbGciOiJSUzI1NiIsInR…",
"expires_in": 6000,
"refresh_expires_in": 1800,
"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI…",
"token_type": "bearer",
"not-before-policy": 0,
"session_state": "0630a3e4-c4fb-499c-946b-294176de57c5",
"scope": "fitnessapp"
}
```
Having the access token, we can call the endpoint to add a new workout record. We
first try to add a workout record for Bill. We expect that adding a workout record for
Bill is valid because the access token we have was generated for Bill.

The next code snippet presents the cURL command you run to add a new workout
for Bill. Running this command, you get an HTTP response status of 200 OK, and a
new workout record is added to the database. Of course, as the value of the Authorization
header, you should add your previously generated access token. I truncated
the value of my token in the next code snippet to make the command shorter and easier
to read:

```bash
curl -v -XPOST 'localhost:9090/workout/' \
-H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOi...' \
-H 'Content-Type: application/json' \
--data-raw '{
  "user" : "bill",
  "start" : "2020-06-10T15:05:05",
  "end" : "2020-06-10T16:05:05",
  "difficulty" : 2
}'
```

If you call the endpoint and try to add a record for Rachel, you get back an HTTP response status of 403 Forbidden:

```bash
curl -v -XPOST 'localhost:9090/workout/' \
-H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOi...' \
-H 'Content-Type: application/json' \
--data-raw '{
  "user" : "rachel",
  "start" : "2020-06-10T15:05:05",
  "end" : "2020-06-10T16:05:05",
  "difficulty" : 2
}'
```
The response body is

```json
{
  "error": "access_denied",
  "error_description": "Access is denied"
}
```

### 18.4.2 Proving that a user can only retrieve their own records
In this section, we prove the second test scenario: our resource server only returns the
workout records for the authenticated user. To demonstrate this behavior, we generate
access tokens for both Bill and Rachel, and we call the endpoint to retrieve their
workout history. Neither one of them should see records for the other. To generate an
access token for Bill, use this curl command:

```bash
curl -XPOST 'http://localhost:8080/auth/realms/master/protocol/openidconnect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=bill' \
--data-urlencode 'password=12345' \
--data-urlencode 'scope=fitnessapp' \
--data-urlencode 'client_id=fitnessapp'
```

Calling the endpoint to retrieve the workout history with the access token generated for Bill, the application only returns Bill’s records:

```bash
curl 'localhost:9090/workout/' \
-H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSl...'
```

The response body is

```json
[
  {
    "id": 1,
    "user": "bill",
    "start": "2020-06-10T15:05:05",
    "end": "2020-06-10T16:10:07",
    "difficulty": 3
  },
  . . .
]
```

Next, generate a token for Rachel and call the same endpoint. To generate an access token for Rachel, run this curl command:

```bash
curl -XPOST 'http://localhost:8080/auth/realms/master/protocol/openidconnect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=rachel' \
--data-urlencode 'password=12345' \
--data-urlencode 'scope=fitnessapp' \
--data-urlencode 'client_id=fitnessapp'
```

Using the access token for Rachel to get the workout history, the application only returns records owned by Rachel:
```bash
curl 'localhost:9090/workout/' \
-H 'Authorization: Bearer eyJhaXciOiJSUzI1NiIsInR5cCIgOiAiSl...'
```

The response body is
```json
[
  {
    "id": 2,
    "user": "rachel",
    "start": "2020-06-10T15:05:10",
    "end": "2020-06-10T16:10:20",
    "difficulty": 3
  },
  ...
]
```

### 18.4.3 Proving that only admins can delete records
The third and last test scenario in which we want to prove the application behaves as
desired is that only admin users can delete workout records. To demonstrate this
behavior, we generate an access token for our admin user Mary and an access token
for one of the other users who are not admins, let’s say, Rachel. Using the access token
generated for Mary, we can delete a workout. But the application forbids us from calling
the endpoint to delete a workout record using an access token generated for
Rachel. To generate a token for Rachel, use this curl command:

```bash
curl -XPOST 'http://localhost:8080/auth/realms/master/protocol/openidconnect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=rachel' \
--data-urlencode 'password=12345' \
--data-urlencode 'scope=fitnessapp' \
--data-urlencode 'client_id=fitnessapp'
```

If you use Rachel’s token to delete an existing workout, you get back a 403 Forbidden
HTTP response status. Of course, the record isn’t deleted from the database. Here’s
the call:
```bash
curl -XDELETE 'localhost:9090/workout/2' \
--header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsIn...'
```
Generate a token for Mary and rerun the same call to the endpoint with the new
access token. To generate a token for Mary, use this curl command:

```bash
curl -XPOST 'http://localhost:8080/auth/realms/master/protocol/openidconnect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=mary' \
--data-urlencode 'password=12345' \
--data-urlencode 'scope=fitnessapp' \
--data-urlencode 'client_id=fitnessapp'
```

Calling the endpoint to delete a workout record with the access token for Mary
returns an HTTP status 200 OK. The workout record is removed from the database.
Here’s the call:

```bash
curl -XDELETE 'localhost:9090/workout/2' \
--header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsIn...'
```