# Getting Started

### Creating Account

Visit [https://app.exogress.com](https://app.exogress.com/) to create a new account and follow the instructions. If you signed up using email, make sure you have activated your account.

### Choose your account and subdomain names

You will be prompted to provide your account name and a subdomain name in `exg.link` zone. Your account will be associated with the third-level domain, and couldn't be changed in the future. You will be able to create fourth-level subdomains in this domain.

### Adding your Domain or fourth-level domain

Exogress allows to add your own domain or to create a subdomain under the `<your_account>.exg.link`.

Visit `Domain` section in the sidebar, click `Add Domain in `<your_account>.exg.link`` and enter desired fourth-level subdomain name, then click `Save`. Exogress will take care of issuing TLS certificates for you. Wait for the certificate status to become `ready` as you won't be able to use the service until that.

To add your own domain, navigate to the "Custom domains" tab and click `Add Custom Domain`. There, provide your custom domain name, add a CNAME record to your domain's DNS records and finally, provide a strict transport security expiration period in seconds: recommended value is 31536000. You must add the provided record in your DNS zone, otherwise you will not be able to register the domain. Make sure the DNS records are propagated, depenfing on zone configuration, it make take significant amount of time.

### Creating Project

Project is a place your configuration lives in, and you could add and delete projects at any time.
Navigate to `+Add project` under **Projects** menu. Give your Project a name and click `Save`.

### Adding Mount Point

Mount points live inside your projects, it's a core entity that connects your Exofiles with your domains. The benefit of mount points is that you don't link your Exofile configuration directly with domains, so that you may use the same Exofiles under different accounts, and different domain names.

Once you've registered, a `default` Mount Point is created in the project page. Choose your domain from the select box it will be linked to and click `Save Mapping`.
You can create and delete Mount Points at any time.

### Creating Access Token

Access tokens are required for spawning Exogress client.
Click the `Access Tokens` section in the sidebar. Provide access token name and click `Create Access Token`. You will need two tokens:

- `ACCESS_KEY_ID`
- `SECRET_ACCESS_KEY`

Make sure you save the secret token to a secure place; it will be impossible to retrieve it later.

### Spawning Exogress Client

Next step is to create a client-level config, which typically lives in your git repositories in the configuration file - "Exofile.yml".
In your directory, run commmand

```
exogress init <PARAMETER> --port=3000
```

Where specified port is passed as an argument (`3000` in the example), adn <PARAMETER> is one of the currently supported frameworks:

```
laravel-artisan          Initialize Exofile.yml for Laravel with Artisan server
matrix-synapse-docker    Initialize matrix-synapse docker app with Exofile.yml
proxy                    Initialize Exofile.yml for simple proxying
rails                    Initialize Exofile.yml for Ruby On Rails
svelte                   Initialize Exofile.yml for Svelte
```

Running the command with a selected parameter will create `Exofile.yml` in your current directory. Below is the default configuration of the Exofile:

```
---
version: 1.0.0
revision: 1
name: default
mount-points:
  default:
    handlers:
      proxy:
        kind: proxy
        priority: 50
        upstream: backend
upstreams:
  backend:
    port: 3000
```

The configuration above will forward requests to port `3000` on `localhost`. Please refer to [Configuration section](/exofile.md) for the details.

Run your Exogress client

```
$ exogress spawn --access-key-id=<ACCESS_KEY_ID> --secret-access-key=<SECRET_ACCESS_KEY> --account <ACCOUNT_NAME> --project <PROJECT_NAME>
```

That's it. Now the server on port `3000` is available under the selected domain.

## Additional configuration

### Project-level config

Project-level config simplifies configuration in cases when you have multiple client-level configs and want to set some common behaviour across all of them, and works even without any running clients. Upstreams, proxy and static-dir handlers are not allowed for the Project-level config.

### Certificates

We use LetsEncrypt to issue and renew TLS certificates for your domains. Make sure the status is `Ready` before sending the traffic to the domain.

### Project Parameters

Parameters may be referred from both project- and client- configs. Sometimes keeping the data in the config may be awkward
or insecure, or the same data may be required by more than one config. Parameters are created specifically to handle this
situations. There are certain config values which may be replaces with parameters.

#### Example

Here is the simple config which enforce github authorization.
```yaml
---
version: 1.0.0
revision: 1
name: proxy
mount-points:
  default:
    handlers:
      auth:
        kind: auth
        github:
          acl:
            - allow: github-user
        priority: 10
```
Eventually ACL may become large, and it would be hard to manage it in the config file. Let's create the parameter
`github-acl` and `acl` schema with the following content:

```yaml
---
- allow: github-user
```

And update the config as follows:
```yaml
---
version: 1.0.0
revision: 1
name: proxy
mount-points:
  default:
    handlers:
      auth:
        kind: auth
        github:
          acl: "@github-acl"
        priority: 10
```

It would work exactly the same, but is much cleaner in terms of configuration readability.

By default, we create a system parameter `compressible-mime-types` which is used to identify which content-type should
be compressed. System parameters can't be deleted, but user may change it's content.

In addition to system parameters, users may create any number of parameters with the following types:

- `aws-credentials`
- `s3-bucket`
- `google-credentials`
- `gcs-bucket`
- `acl`
- `static-response`
- `mime-types`

Please, refer to the configuration section for more details on using the parameters.
