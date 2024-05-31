# JWT (2/8)

JWT (JSON Web Tokens) are tokens that are used in many web applications for access control and authentication mechanisms. It uses the following format:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

These three dot-separated parts compose the JWT. The first two are base65-encoded JSON objects and the last one is a signature. The first one, marked in red, is the header, which usually contains information about the token. The second one, in purple, is the payload. It will normally carry information about the user.

## Labs

### **JWT authentication bypass via unverified signature**

In this lab, we have an admin panel at /admin, which gives us a message stating that it’s only available if logged in as administrator. 

![Untitled](./assets/untitled-0.png)

If we edit our JWT session token and change the value of the username to “administrator”, the website doesn’t verify the token’s signature and lets us access the panel. I used burp’s JWT editor extension.

![Untitled](./assets/untitled-1.png)

### **JWT authentication bypass via flawed signature verification**

After trying to access `/admin`, we receive a message stating that the interface is only available for the administrator user. We can presume that’s the account username we need to target.

![Untitled](./assets/untitled-2.png)

I managed to manipulate the JWT, changing the `“alg”` (algorithm) field to none, making so the operating system doesn’t use any algorithm to verify the JWT signature, and also deleting the signature itself. In this case, we also need to leave the trailing dot, otherwise the application will interpret the JWT as broken.

![Untitled](./assets/untitled-3.png)

![Untitled](./assets/untitled-4.png)
