Swiss army knife for web application delivery
=============================================

![alt text](https://user-images.githubusercontent.com/10335584/113483361-985b5780-94d5-11eb-9243-c9de99b9955d.png "Exogress")


At Exogress, we work on democratizing the infrastructure setup required to expose your web application.


To deliver web traffic to and from their web apps, engineers have to ensure multiple components are working. We've listed them below:

### Localhost / Tunnels
We typically start with running our apps on localhost. It works well unless it requires webhooks or sharing results with others. This is where services like [ngrok](https://ngrok.com) come in handy. 

### Load Balancers / Reverse-proxy
When an application runs on a server, it's at the very least expected to be:

- Protected with HTTPS
- Load-Balanced
- Reachable by the end user

### Content-Delivery Network (CDN)
Some content types require a really speedy delivery - this is achievable with a combination of `Cache-Control` and CDN.

### Static Web Sites Hosting
Sometimes it doesn't make sense to host static web applications on your servers. This is where it may be sufficient to use services like 
[Netlify](https://www.netlify.com).

All of these components are important and take a significant part in the modern web delivery infrastructure. But there is 
a downside on such a fragmentation - error-prone configuration, incompatibility, additional effort to make them play together.

Exogress, on the other hand, combines all these tools under a single hood. It is, at the same time, a Load Balancer,
a Serverless platform and a CDN, and it just works in any environment without extra setup costs. It works on localhost, 
on your server, or Raspberry Pi.
