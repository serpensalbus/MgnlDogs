---
title: Azure AD Token Test
description: Test if the Azure AD OIDC application setup works using Postman.
---

# Azure AD (AAD) token testing using Postman

This article shows you how to obtain and inspect access and OpenID connect tokens without using Magnolia when your identity provider is Azure AD (AAD). This makes sure all basic settings work as intended and can help to limit the amount of work you have to put into troubleshooting.

It's assumed you have configured your OpenID Connect application on AAD and the **groups** claim has been added to the token payload. In addition to that, testing happens on your local machine.

In this text, I will use [Postman](https://www.postman.com/). This is a free (and wide-spread) tool with 
convenient functions for testing your configuration.

---

## Azure AD application configuration

The only thing you have to do on the AAD configuration side, is to add an appropriate redirect URI for the Postman test.

There are several options:

- You can use a redirect URI that already exists (like http://localhost:8080/magnoliaAuthor).
- You add a redirect URI for localhost (http://localhost).
- It's also possible to use a URI provided by Postman when you want to use your browser to execute the test (https://oauth.pstmn.io/v1/callback).

![AAD Redirect URI](_img/aad-token/01_redirect_uri.png)

The screenshot shows an application configuration that has all the options.

---

## Postman

### Obtaining configuration values

!!! tip

    **In AAD, there is a link called *Endpoints* in the *overview* of your application configuration where you can see (and copy + paste) relevant endpoint configurations.**

![AAD Endpoints](_img/aad-token/02_overview_endpoints.png)

The OpenID Connect protocol mandates a public URL for the **OpenID Connect metadata document**, which you can also find in the **Endpoints** dialog.

For AAD, this URL usually looks like

*https://login.microsoftonline.com/{{your-tenant-id-or-name}}/.well-known/openid-configuration*

Calling this URL is a first test and should return a proper result. Many of the configuration options needed for connecting your application to AAD using the OpenID Connect protocol can be found in the result data.

---

### Postman token configuration

In Postman, create a new **Collection** and open the tab *Authorization*. From the select box, choose OAuth 2.0, go to **Configuration Options** and enter needed values.

|Option | Hint| Example|
|------------ | ------------| ------------|
|**Token Name**| |  Arbitrary name|
|**Callback URL** | Defined in AAD| http://localhost (see above)|
|**Auth URL** | Auth endpoint| https://login.microsoftonline.com/{{tenant}}/oauth2/v2.0/authorize|
|**Access Token URL** | Token endpoint| https://login.microsoftonline.com/{{tenant}}/oauth2/v2.0/token|
|**Client ID**| AAD App ID| 5b5ffb6c-f62e-43e5-b646-d3b782a1c03d|
|**Client Secret** | AAD Client Secret| TLm8Q~s_mS9pILtPzYm~oegpGUSXoxyphGKrepip|
|**Scope** | | openid profile email|
|**State** |Security feature| {{$randomInt}}|

![Postman Token Config](_img/aad-token/03_token_configuration.png)

The values for

- Callback URL (redirect URI)
- Client ID (application ID)
- Client Secret
- Scope

are also needed for Magnolia SSO configuration.

### Test token retrieval

Click on **Get New Access Token**, login with your AAD credentials. If everything was OK, you should have received your new token.

![Postman Token OK Message](_img/aad-token/04_test_token_OK.png)

The token is shown in the next screen.

![Postman Token Dialog](_img/aad-token/05_test_token.png)

Though you can use this token to access other APIs hosted on AAD, we can end our test now because we verified that login and retrieval of the token actually works.

---

### Authorize with browser

You can use your browser to execute the test. You might get detailed information for troubleshooting when using this approach. Remember, you have to define the redirect URI for Postman in your AAD application configuration (https://oauth.pstmn.io/v1/callback).

![Postman Browser Authorization Option](_img/aad-token/06_browser_auth.png)

---

### Check token content

Depending on your needs, you configured AAD to deliver group membership information in the payload of your **ID token**. We can decode the content with a tool like https://jwt.io.

**Note that data like group membership is added to the id_token—be sure to copy the correct token value before verifying the payload.**

For example, group membership looks like:

```json
"groups": [
    "ac5db011-28d6-4738-8f3b-b84b52c41a56",
    "3bf13aad-eead-44ee-b666-ba9669a5b120"
  ],
```

AAD does not deliver group names, only the IDs.

You can use the access token to resolve the names using MS Graph API, but this is a topic for a separate article.

Also consider that the **callback URL** / **redirect URI** is likely to be different when using the Magnolia SSO module.

---

### Problems with token generation

If you get errors while executing the steps above, keep on fixing them until you finally receive a valid ID token. Don't continue with Magnolia SSO until the whole process works.

![Cool Man](_img/aad-token/ban_tongue.jpg)