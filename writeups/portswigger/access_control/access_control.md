# Access Control (13/13)

**Table of contents**

1. [Introduction](#introduction)

2. [Labs](#labs)

    1. [Unprotected admin functionality](#unprotected-admin-functionality)
    
    2. [Unprotected admin functionality with unpredictable URL](#unprotected-admin-functionality-with-unpredictable-url)
    
    3. [User role controlled by request parameter](#user-role-controlled-by-request-parameter)
    
    4. [User role can be modified in user profile](#user-role-can-be-modified-in-user-profile)
    
    5. [User ID controlled by request parameter](#user-id-controlled-by-request-parameter)
    
    6. [User ID controlled by request parameter, with unpredictable user IDs](#user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
    
    7. [User ID controlled by request parameter with data leakage in redirect](#user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
    
    8. [User ID controlled by request parameter with password disclosure](#user-id-controlled-by-request-parameter-with-password-disclosure)
    
    9. [Insecure direct object references](#insecure-direct-object-references)
    
    10. [URL-based access control can be circumvented](#url-based-access-control-can-be-circumvented)
    
    11. [Method-based can be circumvented](#method-based-can-be-circumvented)
    
    12. [Multi-step process with no access control on one step](#multi-step-process-with-no-access-control-on-one-step)
    
    13. [Referer-based access control](#referer-based-access-control)



## Introduction

Access Control mechanisms, in the context of web applications, are those that limit the actions a user is able to perform. These validations, to be secure, must necessarily be done in the server-side of the application, so that malicious users can't bypass them by simply performing the sensitive requests manually.

Broken Access Control vulnerabilities can lead to:

- Vertical Privilege Escalation: when an attacker is able to perform actions that should be exclusive to users with higher privilege than their own

- Horizontal Privilege Escalation: when an attacker can execute actions in behalf of other users with the same privileges as them

- Subversion of the application’s logic: when the attacker is able to perform actions in a way that’s not predicted by the application



## Labs

### Unprotected admin functionality

The application in this lab contained an exposed admin panel, which we could discover by looking on `robots.txt`.

![Untitled](./assets/untitled-34.png)

![Untitled](./assets/untitled-35.png)

It doesn't perform any validation that could stop users from getting the contents of the panel, neither from performing the action of deleting another user's account.

![alt text](./assets/untitled-36.png)

---

### Unprotected admin functionality with unpredictable URL

In this case, the admin panel doesn’t have a guessable location, but there’s a disclose of it’s location on the page’s source code.

![Untitled](./assets/untitled-0.png)

![Untitled](./assets/untitled-1.png)

---

### User role controlled by request parameter

This website manages admin access control by checking for the value of a cookie in the request. This way, we can intercept our requests and insert the `Admin=true` cookie in each of them, or simply edit the cookie's value in the browser, allowing us to perform high-privilege operations. 

![Untitled](./assets/untitled-2.png)

![Untitled](./assets/untitled-3.png)

---

### User role can be modified in user profile

When changing our account’s e-mail, it’s noticeable that it sends and returns a JSON. The first containing a field naturally called e-mail, and the other, having a `roleid` key-value pair.

![Untitled](./assets/untitled-4.png)

I then simply included that `roleid` pair in my request with the value of 2, granting myself permanent admin privileges.

![Untitled](./assets/untitled-5.png)

The application probably defaults the value of `roleId` to `1` in the source code, but still accepts it to come from the client, not expecting it to be manipulated by a malicious user.

---

### User ID controlled by request parameter

After logging in and going to `/my-account`, we can see that our username is going as a GET parameter in the request.

![Untitled](./assets/untitled-6.png)

After noticing that, I intercepted the request to `/my-account` and edited it, adding the username of “carlos” as the account’s id, which allowed me to access another user’s account.

![Untitled](./assets/untitled-7.png)

---

### User ID controlled by request parameter, with unpredictable user IDs

Although this lab uses unpredictable user ID’s, it discloses the user's id in their posts.

![Untitled](./assets/untitled-9.png)

![Untitled](./assets/untitled-10.png)

---

### User ID controlled by request parameter with data leakage in redirect

Although the application redirects us to the login page if we try to access another user’s account, the redirect response still includes the content of the user’s account page.

![Untitled](./assets/untitled-13.png)

![Untitled](./assets/untitled-14.png)

---

### User ID controlled by request parameter with password disclosure

On my-account page, we can see that the password is filled in the input field. Although it’s hidden, we can see it in the page’s HTML.

![Untitled](./assets/untitled-15.png)

![Password disclose in HTML](./assets/untitled-16.png)

Password disclose in HTML

![Untitled](./assets/untitled-17.png)

![Untitled](./assets/untitled-18.png)

---

### Insecure direct object references

This application allows us to get transcripts of the live chat.

![Untitled](./assets/untitled-19.png)

Noticing that the first transcript file’s ID is 2, we can repeat the GET request and try to get other files that weren’t generated by us.

![Untitled](./assets/untitled-20.png)

Here we get carlos’ password.

![Untitled](./assets/untitled-21.png)

![Untitled](./assets/untitled-22.png)

### URL-based access control can be circumvented

This lab shows us the example of an application that contains an unauthenticated admin panel that contains restrictions disallowing external accesses to it. However, it’s back-end is built in a framework that supports the `X-Original-Url` custom header. This header allows users to overwrite the path being explicitly accessed in the request, and therefore, bypass that restriction.

![Untitled](./assets/untitled-23.png)

![Getting access to the admin panel](./assets/untitled-24.png)

Getting access to the admin panel

![Untitled](./assets/untitled-25.png)

![Untitled](./assets/untitled-26.png)

Although receiving the status code 403, our action was performed anyway and the user was deleted.

### Method-based can be circumvented

By analyzing the workflow of upgrading a user’s role with an administrative account, we notice that the request made uses the POST method. 

![Untitled](./assets/untitled-27.png)

If we attempt to repeat the request being logged in our regular, unprivileged user, we get 401 (unauthorized). However, if we send the same request, changing the method to GET and entering the data of the original request’s body in the URL, we are able to bypass the access control and grant ourselves admin privileges.

![Untitled](./assets/untitled-28.png)

### Multi-step process with no access control on one step

The lab gives us admin credentials, so we can analyze the account role upgrading process from there before actually exploring the vulnerability.

Here, the process of upgrading a user’s role take two steps: submitting the form and confirming the submission. Each process is handled by it’s own request.

![Submitting form](./assets/untitled-29.png)

Submitting form

![Confirming action](./assets/untitled-30.png)

Confirming action

The application successfully performs access control in the first step, but doesn’t do it in the second, which we request directly, successfully bypassing access control.

![Untitled](./assets/untitled-31.png)

Confirming the action direcly, with our unprivileged account.

### Referer-based access control

In this case, by analyzing the process of upgrading user privileges, we can attempt performing the request as our own user, but passing the same `Referer` header as the original one, tricking the application into thinking that the request was actually performed by a button in the admin page.

![Untitled](./assets/untitled-32.png)

This is ugly as hell.

![It works.](./assets/untitled-33.png)

It works.