Swiss army knife for web application delivery
=============================================

![alt text](https://app.exogress.com/exogress.png "Exogress")

At Exogress, we work on democratizing the infrastructure setup required to expose your web application.


Engineers typically deals with multiple components to deliver the web traffic to and from their web apps.

### Localhost / Tunnels
Typically we start from running our apps on the localhost. It may work until you need to deal with webhooks or  you want
to share the results with others. In this case you may want to use services like [ngrok](https://ngrok.com).

### Load Balancers / Reverse-proxy
When your application is on the server, you would typically want it to be at least 

- Protected with HTTPS
- Load-Balanced
- Reachable by the end user

### Content-Delivery Network (CDN)
Some content need to be served really fast. By the combination of `Cache-Control` and CDN this is achievable.

### Static Web Sites Hosting
Sometimes it doesn't make sense to host static web applications on your servers. It may be enough to use services like 
[Netlify](https://www.netlify.com).

All of these components are important and takes a significant part in the modern web delivery infrastructure. But there is 
a downside on such a fragmentation - error-prone configuration, incompatibility, additional effort to make them play together.

Exogress, on the other hand, combines all this tools under the single hood. It is, at the same time, a Load Balancer,
a Serverless platform and a CDN, and, it just works in any environment without extra setup costs. It works on localhost, 
on your server or Raspberry Pi.
