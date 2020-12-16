# Getting Started

## Creating Account

You will need the Exogress account to use the service. Please, visit [https://app.exogress.com](https://app.exogress.com/) and create a new account.

## Creating Project

Project is the place where your configurations live in. You will be prompted to create your first project once your account is ready. Later, you'll be able to add more projects.

## Creating Access Token

Click the `Access Tokens` section in the sidebar. Write down the access token name and press `Create Access Token`. You will need two tokens:

- `ACCESS_KEY_ID`
- `SECRET_ACCESS_KEY`

Make sure you save the secret token to a secure place; it will be impossible to retrieve it later.

## Adding your Domain

By default, Exogress allows to create subdomain under the `exg.link` or to add your own domain.

Visit the `Domain` section in the sidebar and enter the desired domain name, and click `Create`. Exogress will take care of issuing TLS certificates for you. Wait for the certificate state to become `ready`. You will be unable to use the service until that.

## Adding the Mount Point

Mount points lives inside your projects. Mount Point is the core entity, which connect you Exofiles with the domains. The benefit of mount points is that you don't connect your Exofile configuration directly with domains, so that you may use the same Exofiles under different accounts, and different domain names.

Add new Mount Point with the name `my-mount-point` in the project page, and chose the domain from the select box it will be linked to.

## Spawning Exogress Client

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
