## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex1-as)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex1-as](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex1-as)
*  [https://livebook.manning.com/book/spring-security-in-action/chapter-15/30] (https://livebook.manning.com/book/spring-security-in-action/chapter-15/30) 
# Notes

## 15.1.2 Implementing an authorization server to issue JWTs

* **Page 366**, We can now start the authorization server and call the /oauth/token endpoint to
obtain an access token. The next code snippet shows you the cURL command to call
the /oauth/token endpoint:

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
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjI5NDk2OTcsInVzZXJfbmFtZSI6ImpvaG4iLCJhdXRob3JpdGllcyI6WyJyZWFkIl0sImp0aSI6IjYwOWI3ODE4LTU3NjMtNDMxYy04MzNkLWRjZDExZWE5YmU3NCIsImNsaWVudF9pZCI6ImNsaWVudCIsInNjb3BlIjpbInJlYWQiXX0.bTGTephlClfUQuKBoVXKzPa6AYYnFuIG_194MqnLaEk",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJqb2huIiwic2NvcGUiOlsicmVhZCJdLCJhdGkiOiI2MDliNzgxOC01NzYzLTQzMWMtODMzZC1kY2QxMWVhOWJlNzQiLCJleHAiOjE2MjU0OTg0OTcsImF1dGhvcml0aWVzIjpbInJlYWQiXSwianRpIjoiMTM2NjY0YzgtNzkxOS00YmE0LTgyYTctYjYxY2E5N2UxYzIyIiwiY2xpZW50X2lkIjoiY2xpZW50In0.ef_HrDYGm1rvNDwZX8f5ab0o1RYsf6NqfM43RdUr7yg",
  "expires_in": 43199,
  "scope": "read",
  "jti": "609b7818-5763-431c-833d-dcd11ea9be74"
}

```