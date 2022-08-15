---
title: Keycloak and Magnolia SSO
description: Documentaton and sample about Magnolia AdminCentral login with Keycloak (Quarkus)
---

# Magnolia SSO and Keycloak (Quarkus)

!!! info

    Samples and descriptions are based on **Magnolia CMS 6.2.21**, **Magnolia SSO 2.0.4** and **Keycloak 19.0.1**.

## Using a recent Keycloak distribution

!!! caution

    If you are new to the most recent Keycloak distributions based on Quarkus, or if you just upgraded from a WildFly distribution, there are some things you have to consider. You can find more information on the [Migrating to Quarkus distribution](https://www.keycloak.org/migration/migrating-to-quarkus) pages.

### Most important changes in Keycloak to consider

I am not covering changes like how to load configuration at startup time etc., just critical changes regarding the integration with Magnolia SSO.

#### New default context path (without “/auth”)

Former URLs like the value used for *oidc.discoveryUri* will not work any longer because the “/auth” part has been removed.

Instead of

`oidc.discoveryUri: http://localhost:8180/auth/realms/mgnl/.well-known/openid-configuration
`

use

`oidc.discoveryUri: http://localhost:8180/realms/mgnl/.well-known/openid-configuration
`

!!! tip

    If you still want/must use the old format (maybe for additional clients you connect to Keycloak), you can start the server with the **http-relative-path** flag, like `bin/kc.[sh|bat] start-dev --http-relative-path /auth`.

#### Adding group membership to the token in Keycloak

To be able to apply appropriate permissions on Magnolia AdminCentral, an external user needs to “bring” some information about group- (role-) memberships after authentication. Otherwise, the Magnolia backend would not be able to resolve correct roles and ACLs.

As the Keycloak administration interface has slightly changed with recent versions, some user might not be able to follow along existing documentation.

**Add group memberships of a user to an ID token:**

- In Keycloak, select your **realm** and the client you want to configure (like *magnoliaAuthor*).
- Choose **Client scopes** → click on **clientName-dedicated**.

![Add client scope](_img/keycloak-sso/01_kc_client_add_scope.png)

- Click on **Add predefined mapper**.

![Add client mapper](_img/keycloak-sso/02_kc_client_add_mapper.png)

If you already defined mappers for your client, the dialog will look different:

- Click on **Add mapper** and **From predefined mappers**.

![Add client mapper](_img/keycloak-sso/03_kc_client_add_mapper.png)

- Add the **groups** mapper to your client configuration.

![Add groups mapper](_img/keycloak-sso/04_kc_client_add_mapper_groups.png)

- Select the added **groups** mapper to review settings.

![Add groups mapper](_img/keycloak-sso/05_kc_client_select_mapper.png)

---

## Magnolia SSO configuration

!!! note

    Check the official documentation for details about [Magnolia SSO](https://docs.magnolia-cms.com/magnolia-sso/2.0.5/index.html).

### Example config.yaml including groups claim

```yaml
authenticationService:
  path: /.magnolia/admincentral
  callbackUrl: http://localhost:8080/magnoliaAuthor/.auth
  postLogoutRedirectUri: http://localhost:8080/magnoliaAuthor/.magnolia/admincentral
  authorizationGenerators:
    groupsAuthorizationGenerator:
      class: info.magnolia.sso.oidc.GroupsAuthorizationGenerator
      mappings:
        superusers:
          roles:
            - superuser
            - rest-admin
        travel-demo-editors:
          roles:
            - security-base
            - travel-demo-editor
            - workflow-base
            - travel-demo-tour-editor
            - imaging-base
            - travel-demo-admincentral
            - resources-base
  pac4j:
    oidc.id: magnoliaAuthor
    oidc.secret: 10ef547b-a13c-4e59-8228-05f2b528d371
    oidc.scope: openid profile email
    oidc.discoveryUri: http://localhost:8180/realms/mgnl/.well-known/openid-configuration
    oidc.preferredJwsAlgorithm: RS256
```

#### Keycloak-specific values

The values for **callbackUrl**, **oidc.id** and **oidc.scope** are coming from your [Keycloak OIDC client](keycloak-client.md). Others, like **oidc.scope**, **oidc.discoveryUri** and **oidc.preferredJwsAlgorithm** are the default values for Keycloak according to the OpenID Connect protocol standards. 

---

## More hints

### Groups delivered as path

!!! caution

    This example does not have the leading “/” before the group names, as shown in the [Magnolia SSO documentation](https://docs.magnolia-cms.com/magnolia-sso/2.0.5/index.html). In former Keycloak versions, there was the default option **“Full group path”** in the protocol mapper (for including the groups claim in the token). As there is no use for Magnolia to receive a single string with several group names, this option is not needed. 
    But if you somehow still are using "full path" groups, then the **mapping in config.yaml** must be adjusted (*/superusers* instead of superusers and */travel-demo-editors* instead of *travel-demo-editors*).
