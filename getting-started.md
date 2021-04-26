# Getting Started

### Creating Account

Visit [https://app.exogress.com](https://app.exogress.com/) to create a new account and follow the instructions. If you signed up using email, make sure you have activated your account.

### Choose your account and subdomain names

You will be prompted to provide your account name and a subdomain name in `exg.link` zone. Your account will be associated with the third-level domain, and you couldn't change it in the future. You will be able to create fourth-level subdomains in this domain.

### Adding your fourth-level Domain

Exogress allows you to create a subdomain under the `<YOUR_SUBDOMAIN>.exg.link` or add your domain.

Visit the `Domain` section in the sidebar, click `Add Domain in <YOUR_SUBDOMAIN>.exg.link` and enter desired fourth-level subdomain name, then click `Save`. Exogress will take care of issuing TLS certificates for you. Wait for the certificate status to become `ready` as you won't be able to use the service until that.

To add your domain, navigate to the "Custom domains" tab and click `Add Custom Domain`. There, provide your custom domain name, add a CNAME record to your domain's DNS records and finally, provide a strict transport security expiration period in seconds: recommended value is 31536000. You must add the provided record in your DNS zone; otherwise, you will not be able to register the domain. Ensure the DNS records are propagated: depending on zone configuration, it might take a significant amount of time.

### Creating Project

A Project is a place your configuration lives in, and you could add and delete projects at any time.
Navigate to `+Add project` under **Projects** menu. Give your Project a name and click `Save`.

### Adding Mount Point

Mount points live inside your projects and it's a core entity that connects your Exofiles with your domains. The benefit of mount points is that you don't link your Exofile configuration directly with domains. This way, the same Exofile could be used under different accounts and different domain names.

Once you've registered, we automatically create a `default` Mount Point on the project page. Choose your domain from the select box it will be linked to and click `Save Mapping`.
You can create and delete Mount Points at any time.

### Attaching Domain to Mount Point

You can use one Mount Point for one or multiple domains. To map them, navigate to Mount Points in your Project and select domains you would like to attach. 

### Creating Access Token

Access tokens are required for spawning Exogress clients.
Click the `Access Tokens` section in the sidebar. Provide access token name and click `Create Access Token`. You will need two tokens:

- `ACCESS_KEY_ID`
- `SECRET_ACCESS_KEY`

Ensure you save the secret token to a secure place; it will be impossible to retrieve it later.

### Spawning Exogress Client

The next step is to create a client-level config, which typically lives in your git repositories in the configuration file - "Exofile.yml".
In your directory, run the command

```
exogress init <PARAMETER> 
```

Where <PARAMETER> is one of the currently supported frameworks:

```
laravel-artisan          Initialize Exofile.yml for Laravel with Artisan server
matrix-synapse-docker    Initialize matrix-synapse docker app with Exofile.yml
proxy                    Initialize Exofile.yml for simple proxying
rails                    Initialize Exofile.yml for Ruby On Rails
svelte                   Initialize Exofile.yml for Svelte
```

For the proxy, add a specified port is passed as an argument (`3000` in the example) like in the example below:

```
exogress init <PARAMETER> --port=3000
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

Project-level config simplifies configuration in cases when you have multiple client-level configs and want to set some common behavior across all of them, and work even without running clients. Upstreams, proxy, and static-dir handlers are not allowed for the Project-level config.

### Certificates

We use LetsEncrypt to issue and renew TLS certificates for your domains. Make sure the status is `Ready` before sending the traffic to the domain.

### Project Parameters

Sometimes, keeping your data in the config might not be the best idea for security or other reasons. In some projects, more than one config might use the same data. To handle such situations, we introduced Parameters: you can replace specific config values with parameters and use them in both project- and client- configs.

#### Example

Here is the simple config that enforces GitHub authorization.
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
Eventually, ACL may become large, and it would be hard to manage it in the config file. Let's create the parameter
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

It would work exactly the same but is much cleaner in terms of configuration readability.

By default, we create a system parameter `compressible-mime-types` used to identify which content-type should
be compressed. You can't delete System parameters, but can change its content.

In addition to system parameters, users may create any number of parameters with the following types:

- `aws-credentials`
- `s3-bucket`
- `google-credentials`
- `gcs-bucket`
- `acl`
- `static-response`
- `mime-types`

Please, refer to the configuration section for more details on using the parameters.
