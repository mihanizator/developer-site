Expose your web app by simply running it
=============================================

![alt text](https://user-images.githubusercontent.com/10335584/111993826-09a01f80-8b52-11eb-8413-f1d8d8f7291f.png "Exogress")


At Exogress, we work on democratizing the infrastructure setup required to expose your web application.

### Removing multi-layer configuration

Application delivery configuration can get multi-layered and complicated. Separate setup required across all services involved - backends, load balancers, CDNs and so on.
Exogress introduces a tunneling concept. Each backend server has an Exogress client running, and can establish a secure direct connection with the edge gateway. It allows you to move delivery functions out of your infrastructure, and, as a developer, you need to simply run your code somewhere: a public cloud, development machine or raspberry pi.

Exogress has only two layers - **Edge Gateways Network as a service** and your **data sources** - backends and static content.

### Advanced configuration management

Traditionally, all delivery components require setting up various systems with different configuration management. Depending on the environment, some functions just impossible to implement, or there is a high risk of misconfiguration.
Exogress introduces a state-of-art YAML configuration file - `Exofile`. It's stored right in your code repository, allowing you to configure delivery in a single place.

### Cloud- and environment agnostic

With Exogress, your app delivery configuration doesn't change when you migrate to another cloud provider.
