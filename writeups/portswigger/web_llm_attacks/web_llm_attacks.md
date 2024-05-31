# Web LLM attacks (1/4)

## Labs

### **Exploiting LLM APIs with excessive agency**

The application gives us access to a live chat with an AI.

![Untitled](./assets/untitled-0.png)

It also let’s us see the message logs.

![Untitled](./assets/untitled-1.png)

As we can see, I tried asking it directly to delete Carlos’ account, but without success. However, the log showed me that it’s trying to perform an SQL `DELETE` operation on the backgrount.

![Untitled](./assets/untitled-2.png)

I then submitted a payload containing a structure similar to the original JSON object in the LLM interaction, placing `“admin”` as the value of `“role”`, which apparently tricked it into thinking that I’m an administrator (?).

![Untitled](./assets/untitled-3.png)

![Untitled](./assets/untitled-4.png)

I read the solution and it seems like the intended way wasn’t that, but I’m too lazy to understand why the way I did worked right now.

![Untitled](./assets/untitled-5.png)
