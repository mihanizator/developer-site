# Getting Started

## Creating Account

You will need to Exogress account to use the service. Please, visit [https://app.exogress.com](https://app.exogress.com/) and create a new account.

## Creating Project

Project is the place where your configurations live in. You will be prompted to create your first project once your account is ready. Later, you'll be able to add more projects.

## Creating Access Token

Click the `Access Tokens` section under the sidebar. Write down the access token name and press `Create Access Token`. You will need two tokens:

- `ACCESS_KEY_ID`
- `SECRET_ACCESS_KEY`

Make sure you save the secret token to a secure place; it will be impossible to retrieve it later.

## Adding your Domain

By default, Exogress allows creating subdomain under the `sexg.link` or add your own domain.

Visit the` Domain` section in the sidebar.

Enter the desire domain name, and click `Create`. Exogress will take care of issuing TLS certificates for you. Wait for the certificate state to become `ready`

## Adding the Mount Point

Mount points lives inside your projects. Mount Point is the core entity. We will later use it in the Exofile configuration. Add new Mount Point with the name `my-mount-point` in the project
page, and chose the domain from the select box it will be linked to.

## Spawning Exogress Client

Now we are ready to try Exogress.

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
