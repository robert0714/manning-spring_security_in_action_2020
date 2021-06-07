## CHAPTER 15 OAuth 2: Using JWT and cryptographic signatures (ssia-ch15-ex1-rs)

* [https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex1-rs](https://github.com/robert0714/spring_security_in_action_2020/tree/master/ssia-ch15-ex1-rs)
* [https://livebook.manning.com/book/spring-security-in-action/chapter-15/49](https://livebook.manning.com/book/spring-security-in-action/chapter-15/49)

# Notes

## 15.1.3 Implementing a resource server that uses JWT

* **Page 367**, The next code snippet shows you how to call the endpoint using cURL:

```bash
export TOKEN=jwt_access_token

curl    -H "Authorization:Bearer $TOKEN" http://localhost:9090/hello
```

The response body is

```bash
Hello!
```

A key used for symmetric encryption or signing is just a random string of bytes. You
generate it using an algorithm for randomness. In our example, you can use any
string value, say “abcde.” In a real-world scenario, it’s a good idea to use a randomly
generated value with a length, preferably, longer than 258 bytes. For more information,
I recommend ***Real-World Cryptography*** by David Wong (Manning, 2020). In chapter 8 of David Wong’s book, you’ll find a detailed discussion on randomness and
secrets:

[https://livebook.manning.com/book/real-world-cryptography/chapter-8/](https://livebook.manning.com/book/real-world-cryptography/chapter-8/)

Because I run both the authorization server and the resource server locally on the
same machine, I need to configure a different port for this application. The next code
snippet presents the content of the application.properties file:

```properties
server.port=9090
jwt.key=MjWP5L7CiD
```