# XSS (Cross-site scripting) (5/30)

Cross-site scripting is an inject vulnerability that allows attackers to execute malicious script in other user’s browsers, which can lead to disclosure of the user’s sensitive information that are stored in the browser and also compromise the whole application in case the victim has higher privileges as a user in the website. The attacker that successfully engages on this attack could also be able to impersonate their victims and perform whatever actions they would normally be able to perform, on their behalf.

# Reflected XSS

Works on a single request/response cycle.

The attacker crafts a malicious URL from a legit website and sends it to the victim through a different source, such as a phishing e-mail. The victim clicks the URL, which causes the script to be executed by their browser, as it comes from an apparently trusted source.

## Exploiting

### Reflected XSS into HTML context with nothing encoded

On the example below, we have an application that has a search mechanism. We can try to execute a JavaScript `alert()` function to see if our browser executes it. To do so, we need to use the HTML `<script>` tag and enter our JavaScript code inside of it.

![Untitled](./assets/untitled-0.png)

By clicking on “Search”, we successfully receive the alert.

![Untitled](./assets/untitled-1.png)

As we can see in the source code, our search input has been successfully interpreted by the browser.

![Untitled](./assets/untitled-2.png)

That means that an attacker could try sending an URL that would trigger that kind of behavior in a victim’s browser. In this case, the URL looks like this:

`https://0a8f008e03a02199800d356f007b0049.web-security-academy.net/?search=<script>alert()<%2Fscript>`

### Using different tags

Sometimes you won’t be able to inject most of the tags and attributes, so you can try brute-forcing them and finding one that is appropriate for the attack.

The tag below consists in an `iframe` tag that loads the target application in the victim’s browser. Then, you are able to execute JavaScript with the `onresize` attribute within the `body` tag. In order to successfully trigger the attribute, we are also attributing an object resizing to the `iframe` tag on load.

`<iframe src=https://0a6f008003f3d3a1814d489800de003d.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>`

### **Reflected XSS into attribute with angle brackets HTML-encoded**

On this lab, the injection point is within the `value` attribute of an HTML `<input>` tag.

![Untitled](./assets/untitled-3.png)

When trying to submit a usual XSS payload, we see that it is html-encoding bad characters before parsing them to the tag’s value.

![Untitled](./assets/untitled-4.png)

However, this behavior only occurred when the payload included angle brackets. It doesn’t HTML encode the payload including only double quotes, which close the “value” attribute. This way, we can try to insert attributes that will trigger the execution of our JavaScript.

![Untitled](./assets/untitled-5.png)

This way, we need to think of a payload that doesn’t use angle brackets.

The payload `" autofocus onfocus=alert() x=”` closes the “value” attribute, adds the `autofocus` attribute to grant that that element will be focused on page load and tells the browser to execute the `alert()` function when the element receives focus.

![Untitled](./assets/untitled-6.png)

`<input type="text" placeholder="Search the blog..." name="search" value="" autofocus onfocus=alert() x="">`

### **Reflected XSS into a JavaScript string with angle brackets HTML encoded**

After performing a search in the application and accessing the page’s source code, we can see that our search’s value goes into a JavaScript string that uses single quotes.

![Untitled](./assets/untitled-7.png)

This way, we can try to escape that string and execute arbitrary JavaScript.

![Untitled](./assets/untitled-8.png)

![Untitled](./assets/untitled-9.png)

![Untitled](./assets/untitled-10.png)

# Stored XSS

This kind of XSS involves the injection of malicious scripts in a request that is going to be stored in the application’s server and further executed in responses other than the one that corresponds to the injection.

## Exploiting

### Stored XSS into HTML context with nothing encoded

The following website has a feature that allows users to comment on other user’s posts and doesn’t take any measures against stored XSS. If we leave a comment that contains a XSS payload, the script will be triggered after a user accesses this post, as our comment will be interpreted by the victim’s browser as an HTML tag that defines JavaScript code.

![Untitled](./assets/untitled-11.png)

### Stored XSS into anchor `href` attribute with double quotes HTML-encoded

On the example below, we see that the text written in the “Website” text field of the post comment formulary goes in the page as the value of the `href` attribute of our name in the post’s comments.

![Untitled](./assets/untitled-12.png)

I’ve tried to edit my page’s HTML and insert `javascript:alert(’oi’)` in the `href` attribute in order to try to execute code in the JavaScript [protocol handler](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/registerProtocolHandler/Web-based_protocol_handlers) whenever someone clicks in my name, and it worked.

![Untitled](./assets/untitled-13.png)

Finally, I inserted the payload in a new comment’s website field, successfully creating the PoC for stored XSS.

![Untitled](./assets/untitled-14.png)

# DOM-based XSS

This kind of XSS occur when user-controllable input gets used in a part of the JavaScript code that enables dynamic code execution.

### **DOM XSS in `document.write` sink using source `location.search`**

By looking at the page’s JavaScript, we notice that our query is being placed inside a document.write function that will append an HTML tag in the page that uses our input in part of it.

![Untitled](./assets/untitled-15.png)

We can also see it clearly here in the page’s HTML.

![Untitled](./assets/untitled-16.png)

The payload here is `‘”><script>alert(1)</script>`, closing both the single and double quotes, as well as the angle brackets, and inserting our script tag afterwords so it also gets written on the page.

![Untitled](./assets/untitled-17.png)

### DOM XSS in `innerHtml` sink using source

The documentation about DOM XSS in HackTricks, as well as the one from PortSwigger, both advise that `innerHtml` sinks don’t allow script nor `<svg>` tags to trigger scripts

![Untitled](./assets/untitled-18.png)

This way, we needed to use `<img>`

`<img src=x onerror="alert()">`

### **DOM XSS in jQuery anchor `href` attribute sink using `location.search` source**

This lab consists in an application that uses jQuery to get the value passed in the URL, specifically in the `returnPath` parameter, and attributes that value to the `href` attribute of the link to go back to the previous page.

We can use that parameter to send a payload that will trigger the `javascript:` pseudo protocol in order to execute arbitrary JavaScript code in a victim’s browser.

![Untitled](./assets/untitled-19.png)

### 

### **DOM XSS in jQuery selector sink using a hashchange event**

On this lab, the application is using a jQuery selector function to be able to perform auto scrolls based in the URL hash (#).

The problem is that some old versions of jQuery allow users to inject malicious JavaScript within that selector.

![Untitled](./assets/untitled-20.png)

`<iframe src="[https://0a16001b047dd6d4807b443700b6009b.web-security-academy.net/#](https://0a16001b047dd6d4807b443700b6009b.web-security-academy.net/#)" onload="this.src += '<img src=xereca onerror=print()>'">`