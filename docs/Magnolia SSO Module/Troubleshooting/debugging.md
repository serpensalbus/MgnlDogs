---
title: Debugging
description: Trace problems and erros when using Magnolia SSO.
---

# Problem and error tracing

![Debug dog](_img/debugging/bn_wet.jpg)

## Enable debug log output

For detailed information about what is going on under the covers I recommend to enable debug logging in your Magnolia **development** project.

Either use your [Log4j configuration](https://docs.magnolia-cms.com/product-docs/6.2/Administration/Monitoring.html#_log4j_2) and add

```xml
<Logger name="org.pac4j" level="DEBUG"/>
<Logger name="com.nimbusds" level="DEBUG"/>
<Logger name="info.magnolia.sso" level="DEBUG"/>
```

or make use of the [Log Tools app](https://docs.magnolia-cms.com/product-docs/6.2/Apps/List-of-apps/Log-Tools-app.html).

!!! caution

    Do not enable debug logging for security related data in productive environments. If you are forced to enable it, remember to turn it off immediately after use.

---

## Check the logs of your IAM solution

If the error message you get on the client side is not accurate or does not even exist, you might find the solution to your problem when checking the log capabilities of your IAM service. Sometimes it's necessary to enable logging (like in Magnolia) before getting detailed information. Examining the logs can also be helpful when the error messages you receive on your end are not clear enough or inaccurate.

### Keycloak Events

![Events in Keycloak](_img/debugging/kc_events.png)

---

## Examine OpenID connect token payloads 

After getting detailed output for Magnolia SSO, you can track down the problem of a failed setup. For example, to see if and how the groups information was added to the user's token after authentication, you could search for **status=200** or for **profile:** and check the data:

**Token response: status=200, content=**

```json
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJGX1BRWXVqbEh3VEE0TjBndXRsLU1lREJXWHU4a3F1dkVnQmV2Qkx1YkgwIn0.eyJleHAiOjE2NjAzMDUzOTUsImlhdCI6MTY2MDMwNTA5NSwiYXV0aF90aW1lIjoxNjYwMzA1MDk1LCJqdGkiOiIwMjc0NmE2Yi02MmRmLTRkNmEtODY2MS05ZDg2ZDllMDM5ZDAiLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgxODAvcmVhbG1zL21nbmwiLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoiYzM3YzFjNWMtZDIwMS00ZTA0LWFjMTgtNWQ1ODQxYjU5NTU2IiwidHlwIjoiQmVhcmVyIiwiYXpwIjoibWFnbm9saWFBdXRob3IiLCJzZXNzaW9uX3N0YXRlIjoiMWE0OTJiMWQtNWEyMS00NjM1LTllYjgtYzI2NjZhYTZkNmJlIiwiYWxsb3dlZC1vcmlnaW5zIjpbImh0dHA6Ly9sb2NhbGhvc3Q6ODA4MCJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsic3VwZXJ1c2VycyIsIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iLCJkZWZhdWx0LXJvbGVzLW1nbmwiXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6Im9wZW5pZCBlbWFpbCBwcm9maWxlIiwic2lkIjoiMWE0OTJiMWQtNWEyMS00NjM1LTllYjgtYzI2NjZhYTZkNmJlIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5hbWUiOiJTdXBpbmRvIFN1cGVydXNlciIsInBlbmlzIjpbInN1cGVydXNlcnMiLCJvZmZsaW5lX2FjY2VzcyIsInVtYV9hdXRob3JpemF0aW9uIiwiZGVmYXVsdC1yb2xlcy1tZ25sIl0sInByZWZlcnJlZF91c2VybmFtZSI6InN1cGVydXNlciIsImdpdmVuX25hbWUiOiJTdXBpbmRvIiwiZmFtaWx5X25hbWUiOiJTdXBlcnVzZXIiLCJlbWFpbCI6InN1cGVydXNlckBsb2NhbGhvc3QifQ.pOFHfDA1JybJqtog6-YNq2xa0sob7gKYOSUbPZuNwJhYZiKnlNbnCQ7PJm2KYOBhfsSeeudVXsHsjDdKUI0SJU2VjJ1kTuiu_9j5UA_7aoSA8EZWFg1iaPQ3WNuLwQ2EW-zFnOO3S1PEdIg4DxgWhbdQunf51AEyl0hSCGNWdAbjLAyNdoSmB7CB_QgtCjJ7Ru7HO-tLsN8p8zWQEe9xs-5w31jDhP1o764MZ1mYocPUErNe2xt0VtZ-4FBHUf_RetE5YbWXumed2kN44gk38zY1gKM3hotFgzgH-3KVJPmZU7PSIojcbdeCd9q1zZKlDhpGbWaizFB4JR5kvw766Q",
"expires_in":300,
"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI5NWY3NTkzZi02MmM3LTRmMDUtYThiMi1mOGM5NzA2ZTg3ODQifQ.eyJleHAiOjE2NjAzMDY4OTUsImlhdCI6MTY2MDMwNTA5NSwianRpIjoiN2M2YzFjNGEtYjkxOS00NmZkLWJkZjgtZTY0NWQ2Y2Q2YzQzIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9tZ25sIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9tZ25sIiwic3ViIjoiYzM3YzFjNWMtZDIwMS00ZTA0LWFjMTgtNWQ1ODQxYjU5NTU2IiwidHlwIjoiUmVmcmVzaCIsImF6cCI6Im1hZ25vbGlhQXV0aG9yIiwic2Vzc2lvbl9zdGF0ZSI6IjFhNDkyYjFkLTVhMjEtNDYzNS05ZWI4LWMyNjY2YWE2ZDZiZSIsInNjb3BlIjoib3BlbmlkIGVtYWlsIHByb2ZpbGUiLCJzaWQiOiIxYTQ5MmIxZC01YTIxLTQ2MzUtOWViOC1jMjY2NmFhNmQ2YmUifQ.f-BCwrkSQx1wDAkJcCm76qilDngEWq2hfDLsEkIXRoU",
"token_type":"Bearer","id_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJGX1BRWXVqbEh3VEE0TjBndXRsLU1lREJXWHU4a3F1dkVnQmV2Qkx1YkgwIn0.eyJleHAiOjE2NjAzMDUzOTUsImlhdCI6MTY2MDMwNTA5NSwiYXV0aF90aW1lIjoxNjYwMzA1MDk1LCJqdGkiOiJjNjNiNTI1ZS1mYmRkLTRmZDktODc1Yi1lOWY2MTYxYmFhZDciLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgxODAvcmVhbG1zL21nbmwiLCJhdWQiOiJtYWdub2xpYUF1dGhvciIsInN1YiI6ImMzN2MxYzVjLWQyMDEtNGUwNC1hYzE4LTVkNTg0MWI1OTU1NiIsInR5cCI6IklEIiwiYXpwIjoibWFnbm9saWFBdXRob3IiLCJzZXNzaW9uX3N0YXRlIjoiMWE0OTJiMWQtNWEyMS00NjM1LTllYjgtYzI2NjZhYTZkNmJlIiwiYXRfaGFzaCI6InRZODRwTGx0RDF3RUE4ak9jQVByaHciLCJzaWQiOiIxYTQ5MmIxZC01YTIxLTQ2MzUtOWViOC1jMjY2NmFhNmQ2YmUiLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwibmFtZSI6IlN1cGluZG8gU3VwZXJ1c2VyIiwicGVuaXMiOlsic3VwZXJ1c2VycyIsIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iLCJkZWZhdWx0LXJvbGVzLW1nbmwiXSwicHJlZmVycmVkX3VzZXJuYW1lIjoic3VwZXJ1c2VyIiwiZ2l2ZW5fbmFtZSI6IlN1cGluZG8iLCJmYW1pbHlfbmFtZSI6IlN1cGVydXNlciIsImVtYWlsIjoic3VwZXJ1c2VyQGxvY2FsaG9zdCJ9.XWE4Bjndf6QmhKhVd35-pPC6VLb8wr-bbiv9SCf8Fpm7pw-N_5dEvp_n3ekX0BZk4Avq-sjUhoNMbv2LQyvQxLE8BHePAtQ7cc7YLQ3afQtKVqIxdWEMaIvScgkvp7swwq6GICxFrB1ayzHVFz4RGTD3Un72RzDf7HrdnVYcdl8LDH4GxiwpelQI3-mrpwHPaLfkAUJGe9uMVr9yxYYvYxVRgmKQ_qry6tsRQGq5p7RSMsie7kmO6lJhw38qPTbj7G68OpF4DsRQSMNNv96WK7_eirkNEndTEa2zTNLnoXMUkNyfAfkraDjGDHErrB5QrsRLvU7HjhRSjZeWOlmFnQ",
"not-before-policy":0,
"session_state":"1a492b1d-5a21-4635-9eb8-c2666aa6d6be",
"scope":"openid email profile"}
```

!!! tip

    Use a tool like [JWT.IO](https://jwt.io) to inspect OpenID Connect JWT token data (in the example above the value for “id_token”).

---

## Username / userFieldMappings in config.yaml

When using Magnolia SSO, a temporary (in-memory) user is created in memory after successful authentication. This user object requires at least a username, otherwise login will fail because of a null pointer exception. 
The problem can be caused by the settings for **userFieldMappings** in your configuration.

```yaml
userFieldMappings:
  name: preferred_username
  # ...
```

The above example fails with a default **Okta** setup, because the OIDC token payload does not contain the attribute **preferred_username**. Therefore, Magnolia SSO cannot assign a username and login to Magnolia fails, even after successful authentication with the IdP.

To see what is actually delivered, enable debug logging in Magnolia and inspect the actual content of token data that has been sent to your application.

A working version for **Okta** (and maybe others) looks like:

```yaml
userFieldMappings:
  name: name
  # ...
```

From a login viewpoint, the username is the most important attribute that needs to be assigned. Others may be also significant for the logic of your application, so if you struggle to receive correct values, inspect the logs or adjust the configuration on the IdP and/or Magnolia side.

---

## Wrong / missing scope(s)

Basically, the **scope** parameter defines the kind/amount of data the IAM service will deliver after authentication. Mostly the scope is set to **openid** and in addition **profile** and **email**, but in some cases it might be necessary to add values to or adjust the scope.

Besides the standards named before, in most cases you are free to mess with scope settings on the backend of your security infrastructure. But some services might require  different scope values to receive the same result.

For example, if you are going to evaluate users' groups, to need to add the **groups** value in your configuration.

Consider using this scope setting:

```yaml
oidc.scope: openid profile email
```

In **Okta**, the OIDC token payload does not contain the group's data, even if you configured your application to receive this kind of information.

The token data ends with the “at_hash” attribute:

```json
 "at_hash": "EJNUYPXZfpGNiM8maVimqQ"
}
```

If groups are configured and the scope contain **groups**, group information is provided:

```json
"at_hash": "2Ednv-6g0NPAmhUU3sy52g",
  "groups": [
    "Everyone",
    "magnolia-superusers",
    "travel-demo-editors",
    "magnolia-users",
  ]
}
```

---

## If nothing seems to be wrong

If everything looks good, user authentication works, also redirection to the Magnolia instance without an error message coming from the IdP like “callback URL wrong”, then the problem might be permissions on the Magnolia instance. This means the user might have been authenticated but does not have any roles assigned and therefore has no or no sufficient permissions.

To narrow down this kind of issue, [create a local Magnolia user](https://docs.magnolia-cms.com/product-docs/6.2/Apps/List-of-apps/Security-app.html), assign the same groups/roles the SSO user has received and try to log in without SSO. If it does not work, [permissions are not correct](https://docs.magnolia-cms.com/product-docs/6.2/Administration/Security/Roles-and-access-control-lists.html#_evaluation_of_permissions). You have to consider that when using Magnolia SSO (at the time of writing), local logins are not possible, so you have to replicate the setup on a different instance.
