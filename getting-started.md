# Getting Started

### Creating Account

Visit [https://app.exogress.com](https://app.exogress.com/) to create a new account and follow the instructions. If you signed up using email, make sure you have activated your account.

### Choose your account and subdomain names

You will be prompted to provide your account name and a subdomain name in `exg.link` zone. Your account will be associated with the third-level domain, and couldn't be changed in the future. You will be able to create fourth-level subdomains in this domain.

### Creating Project

Project is a place your configuration lives in, and you could add and delete projects at any time.
Navigate to `+Add project` under **Projects** menu. Give your Project a name and click `Save`.

### Creating Access Token

Access tokens are required for spawning Exogress client.
Click the `Access Tokens` section in the sidebar. Provide access token name and click `Create Access Token`. You will need two tokens:

- `ACCESS_KEY_ID`
- `SECRET_ACCESS_KEY`

Make sure you save the secret token to a secure place; it will be impossible to retrieve it later.

### Adding your Domain

Exogress allows to create a subdomain under the `<your_account>.exg.link` or to add your own domain.

Visit `Domain` section in the sidebar, click `Add Domain in `<your_account>.exg.link`` and enter desired fourth-level subdomain name, then click `Save`. Exogress will take care of issuing TLS certificates for you. Wait for the certificate status to become `ready` as you won't be able to use the service until that.

To add your own domain, navigate to the "Custom domains" tab and click `Add Custom Domain`. There, provide your custom domain name, add a CNAME record to your domain's DNS records and finally, provide a strict transport security expiration period in seconds: recommended value is 31536000. You must add the provided record in your DNS zone, otherwise you will not be able to register the domain. Make sure the DNS records are propagated, depenfing on zone configuration, it make take significant amount of time.

### Adding Mount Point

Mount points live inside your projects, it's a core entity that connects your Exofiles with your domains. The benefit of mount points is that you don't link your Exofile configuration directly with domains, so that you may use the same Exofiles under different accounts, and different domain names.

Once you've registered, a `default` Mount Point is created in the project page. Choose your domain from the select box it will be linked to and click `Save Mapping`.
You can create and delete Mount Points at any time.

### Spawning Exogress Client

Now we are ready to launch Exogress client.

Create default exogress configuration file in your repository

```
$ exogress init
```

This will create `Exofile` in your current directory. Please refer to [Configuration section](/exofile.md) for the details

The default configuration will forward requests to port `3000` on `localhost`.

Run your Exogress client

```
$ exogress spawn --access-key-id=<ACCESS_KEY_ID> --secret-access-key=<SECRET_ACCESS_KEY> --account <ACCOUNT_NAME> --project <PROJECT_NAME>
```

That's it. Now the server on port `3000` is available under the selected domain.
