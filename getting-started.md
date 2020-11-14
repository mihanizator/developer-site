# Gettting Started

## Creating Account

You will need to Exogress account to use the service. Please, visit [https://app.exogress.com](https://app.exogress.com/) and create new account.

## Creating Project

Project is the place, where your configurations lives in. You will be prompted to create your first project once your account is ready. Later, you'll be able to add more projects.

## Creating Access Token

Choose `Access Tokens` section under the sidebar. Write down the accesss token name and press `Create Access Token`. You will need two token:

- `ACCESS_KEY_ID`
- `SECRET_ACCESS_KEY`

Make sure you save the secret token to secure place, it will be impossible to retrieve it later.

## Adding your Domain

By default, Exogress allows to create subdomain under the `sexg.link`.

Visit `Domain` section under the sidebar.

Select the desire subdomain name, and click `Create`. Exogress will take care of issueing TLS certificates for you. Wait for the certificate state to become `ready`

## Adding the Mount Point

Mount points lives inside your projects. Mount Point is the core entity. We will later use it in the Exofile configuration. Add new Mount Point with the name `my-mount-point` in the project
page, and chose the domain from the select box it will be linked to.

## Spawning Exogress Client

Now we are ready to try Exogress.

Create default exogress configuration file in your repositoy

```
$ exogress init
```

This will create `Exofile` in your current directory. Please refer to [Configuration section](/exofile.md) for the details

Default configuration will forward requests to port `3000` on `localhost`.

Run your Exogress client

```
$ exogress spawn --access-key-id=<ACCESS_KEY_ID> --secret-access-key=<SECRET_ACCESS_KEY> --account <ACCOUNT_NAME> --project <PROJECT_NAME>
```

That's it. Now the server on port `3000` is available under the selected domain
