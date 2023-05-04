---
title: Azure AD Group/Permission Assignment
description: How to configure Magnolia SSO using Azure AD together with a custom group authenticator. 
---

# Azure AD and Magnolia SSO custom group authentication

!!! note

    This article is based on **Magnolia SSO 3.1.3** and **Magnolia CMS 6.2.33** (you need an Enterprise license). You cannot use the same configuration with SSO 2.x, but the logic of group resolution in combination with Azure AD is the same. In Azure, it's possible to configure application roles, but this is not covered in this article. If you want to get rid of the role mapping requirement in the SSO YAML configuration file, you can use the solution in this tutorial for other identity providers using your own custom classes.

This article shows you how to use a custom authentication class with Magnolia SSO with Azure AD. The reasons for using this approach may be

- You want to manage users, groups, and permissions using only Azure AD.
- You don't want to deal with cryptic IDs instead of group/role names: Azure AD does not put names in the token payload by default, only IDs.
- Permissions management in your environment is complex and/or flexible.
- People administering Azure AD cannot (or should not) work with YAMl files in a Magnolia installation.

Please review and apply the [Azure AD OpenID Connect client setup](/Magnolia%20SSO%20Module/OIDC%20Client%20Configuration/azuread-client/#register-a-magnolia-client) before starting this tutorial.
If in doubt, you may want to begin with [Azure AD (AAD) token testing using Postman](/Security%20IAM%20SSO/aad-token-test/) before working with Magnolia.

---

It can be quite a challenge to get all the pieces working with this kind of integration. In the beginning you might feel like this:

![Grasdackel](_img/azuread-groups/grasdackel.jpg)

This article will hopefully help make that task easier!

---

## How Azure AD delivers group information

### Group information in the token payload

Basically, you can configure Azure AD to deliver group information within the payload of the OpenID Connect token that is delivered to Magnolia **after a user has successfully authenticated**.

See [Add groups to the token](/Magnolia%20SSO%20Module/OIDC%20Client%20Configuration/azuread-client/#add-groups-to-the-token) on how to configure the token.
See also [Debugging](/Magnolia%20SSO%20Module/Troubleshooting/debugging/) for information on how to view the contents of the tokens delivered to Magnolia.

The Azure AD group information delivered in the token payload looks like this:

![Azure AD group information in token payload](_img/azuread-groups/01_azure_token_groupids.png)

You can map the group IDs in a Magnolia SSO YAML configuration file to apply the required permissions. This is covered in the [official documentation](https://docs.magnolia-cms.com/magnolia-sso/3.1.3/index.html).

### No group information

If there is no group information available in the token data, you can still use MS Graph to query group information once users have obtained the required bearer token.
You will still need to provide the appropriate security configuration in Azure AD, see [API permissions](/Magnolia%20SSO%20Module/OIDC%20Client%20Configuration/azuread-client/#api-permissions).

---

## Azure AD with a custom authorization generator

You will need a custom Magnolia module (Maven/Java) to use this solution. I will not cover how to do this because if you are missing the basics, this article is not for you.

See the [Custom authorization generators](https://docs.magnolia-cms.com/magnolia-sso/3.1.3/guides/custom-auth-generators.html) for more information.

### Service Provider Interface (SPI)

In your custom module, navigate to (create the **service** directory)

`src/main/resources/META-INF/service`

and create a text file named

`info.magnolia.sso.config.spi.AuthorizationGeneratorProvider`

Add the name of your custom provider to the file, such as

`info.magnolia.sso.auth.azure.CustomAzureAdAuthorizationGeneratorProvider`

---

### Maven and module dependencies

Add the Magnolia SSO dependency to your Maven **pom.xml**:

```xml
...
<dependency>
  <groupId>info.magnolia.sso</groupId>
  <artifactId>magnolia-sso</artifactId>
  <version>3.1.3</version>
</dependency>
...
```
and to the module descriptor file /src/main/resources/META-INF/magnolia/**your-module-name.xml**

```xml
 ...
 <dependency>
   <name>sso</name>
   <version>*</version>
 </dependency>
 ...
```
*Change versions and placeholders as needed.*

---

### Custom classes

Create the Java class for your **custom authorization generator**.

```java
package info.magnolia.sso.auth.azure;

public class CustomAzureAdAuthorizationGenerator implements AuthorizationGenerator { 

  private static final Logger log = LoggerFactory.getLogger(CustomAzureAdAuthorizationGenerator.class);

  @Override
  public Optional<UserProfile> generate(WebContext context, SessionStore sessionStore, UserProfile profile) {
      log.info("Hello this is my new custom authorization generator ...");
      return Optional.of(profile);
  }
}
```
Create the Java class for your custom **provider**:

```java
package info.magnolia.sso.auth.azure;

import org.pac4j.core.authorization.generator.AuthorizationGenerator;

public class CustomAzureAdAuthorizationGeneratorProvider implements info.magnolia.sso.config.spi.AuthorizationGeneratorProvider {
    
  @Override
  public AuthorizationGenerator create() {
      return new CustomAzureAdAuthorizationGenerator();
  }
}
```

---

### Magnolia SSO configuration

I recommend having some sort of fallback configuration so that you can see that your project is running and your custom provider is being invoked.
Magnolia SSO configuration is not very consistent, you cannot assume that you can apply the configuration to all possible providers in the same way/syntax. The example assumes *superuser access*.

**Basic config.yaml example for Azure AD:**

```yaml
# tested with SSO 3.1.3
path: /.magnolia/admincentral
callbackUrl: http://localhost:8080/magnoliaAuthor/.auth
postLogoutRedirectUri: http://localhost:8080/magnoliaAuthor/.magnolia/admincentral
authorizationGenerators:
  - name: fixedRoleAuthorization
    fixed:
      targetRoles:
        - superuser
  - name: dummyAuthorizationGenerator
  
clients:
  oidc.id: 7b5bbb6c-f71f-52e4-b646-d3b332a1c10e
  oidc.secret: TXm9Q~s_tS2iILqPzYm~jqwaGUIQoxyphGKrecip
  oidc.clientAuthenticationMethod: client_secret_post
  oidc.scope: openid profile email
  # use your tenant from Azure AD - you will need the value later for the custom provider class
  oidc.discoveryUri: https://login.microsoftonline.com/f7c33569-d9fg-87e6-a2af-3e4feq02310c/v2.0/.well-known/openid-configuration
  oidc.preferredJwsAlgorithm: RS256  
  oidc.authorizationGenerators: fixedRoleAuthorization,dummyAuthorizationGenerator,CustomAzureAdAuthorizationGeneratorProvider

userFieldMappings:
  name: preferred_username
  removeEmailDomainFromUserName: true
  removeSpecialCharactersFromUserName: false
  fullName: name
  email: email
  language: locale
```

!!! note

    Adapt this configuration to your setup! My example runs under the /magnoliaAuthor context. The Azure AD values are fake!
    
---

!!! tip

    **Build and launch your project, open Magnolia AdminCentral. Login with an Azure AD user account, the login should work with superuser privileges.**
    **In the Magnolia log file, you should see the log message we put in the skeleton of our custom authentication generator class.**
    If Magnolia does not start and tells you that something is wrong with your module, there is most likely a problem with the **syntax of config.yaml**.

---

## Add logic to your custom authentication

### Custom authentication generator logic

The **generate method** checks for a **groups** attribute in the payload of the OIDC token.

- The **groups** attribute is **found**: Take the array of group IDs and resolve the names using MS Graph and the authenticated user's bearer token.
- **No groups** attribute: MS Graph and the bearer token are used to query the groups owned by the authenticated user.
- All resolved group **names** are added to the PAC4J OIDC profile under the name **mgnlGroups**.
- After that, the normal flow of the Magnolia SSO module continues, retrieving the groups from the profile, matching them to the ones in Magnolia, and applying permissions.

With this approach, loading roles from groups in Magnolia works, and you don't end up having to do manual role mapping in config.yaml. This is something that did not work for me without the custom authenticator.

---

### Azure AD custom authentication class example

!!! warning

    **Replace the value for  =>private final static String tenant<=  with your own tenant ID (or name)!**

**CustomAzureAdAuthorizationGenerator.java**

```java
package info.magnolia.sso.auth.azure;

import info.magnolia.objectfactory.Components;
import info.magnolia.sso.config.Pac4jConfigProvider;
import net.minidev.json.JSONArray;
import net.minidev.json.JSONObject;
import net.minidev.json.JSONValue;
import org.pac4j.core.authorization.generator.AuthorizationGenerator;
import org.pac4j.core.config.Config;
import org.pac4j.core.context.HttpConstants;
import org.pac4j.core.context.WebContext;
import org.pac4j.core.context.session.SessionStore;
import org.pac4j.core.profile.UserProfile;
import org.pac4j.core.util.HttpUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.*;

import org.apache.commons.lang3.StringUtils;

import org.pac4j.oidc.profile.OidcProfileDefinition;
import com.nimbusds.oauth2.sdk.token.BearerAccessToken;

import static java.util.function.Predicate.not;
import static java.util.stream.Collectors.toList;

public class CustomAzureAdAuthorizationGenerator implements AuthorizationGenerator {

    private static final Logger log = LoggerFactory.getLogger(CustomAzureAdAuthorizationGenerator.class);

    // if the user groups property is delivered with the token payload, it usually contains the UIDs only, not the names
    private static final String GROUPS_PROPERTY = "groups";
    // used to resolve groups in SSOAuthenticationModule
    private static final String MGNL_GROUPS_ATTR_NAME = "mgnlGroups";

    // property for resolving group memberships when there is no group or role attribute in the token payload
    private static final String AZURE_USER_GROUPS_SERVICE_URL = "https://graph.microsoft.com/v1.0/me/memberOf";
    // properties for resolving group UIDs
    // https://graph.microsoft.com/v1.0/<tenant>/groups/<group-uid>
    private static final String AZURE_GROUP_ID_PREFIX = "https://graph.microsoft.com/v1.0/";
    private static final String AZURE_GROUP_ID_SUFFIX = "/groups/";
    // common properties
    private final static String MS_GRAPH_VALUE_PROPERTY = "value";
    private final static String MS_GRAPH_GROUP_NAME_KEY = "displayName";

    // TODO Replace with your own tenant ID!
    private final static String tenant = "f7c33569-d9fg-87e6-a2af-3e4feq02310c";

    @Override
    public Optional<UserProfile> generate(WebContext context, SessionStore sessionStore, UserProfile profile) {
        log.debug("CustomAzureAdAuthorizationGenerator - generate ...");

        List<String> userGroups = new ArrayList<>();
        try {
            Optional<Object> groupsAttribute = Optional.ofNullable(profile.getAttribute(GROUPS_PROPERTY));

            // take the access token from the logged-in user
            // it is needed to query MS Graph APIs
            BearerAccessToken bearerAccessToken = ((BearerAccessToken) profile.getAttribute(OidcProfileDefinition.ACCESS_TOKEN));
            String bearerToken = StringUtils.defaultString(bearerAccessToken.getValue(), "");
            // in production, do not show this in the logs
            log.debug("Bearer token: {}", bearerToken);

            if (groupsAttribute.isEmpty()) {
                // there is no roles and no groups element in the token payload, just general user data
                // we will query MS graph for the user's group memberships
                log.debug("There is no groups/roles claim in the token payload. Group memberships will be queried from MS Graph.");
                JSONObject queryResult = queryMsGraph(bearerToken, AZURE_USER_GROUPS_SERVICE_URL);
                log.debug("Query result: {}", queryResult);

                // parse the result and add group names to the user's group list
                userGroups = getAzureUserGroups(queryResult);
            } else {
                // the groups claim is present in the token payload, but it does only contain a list of the group UIDs, not the names
                log.debug("The groups claim with group UIDs is present in the token payload. Group names will be queried from MS Graph.");

                Object groupsAttributeValue = groupsAttribute.get();
                log.debug("Attempting to gather authorization with {} property value: {}", GROUPS_PROPERTY, groupsAttributeValue);

                List<String> groupIDs = new ArrayList<>();

                if (groupsAttributeValue instanceof ArrayList) {
                    // parse a json array using their parent type ArrayList
                    List<Object> jsonArray = (ArrayList) groupsAttributeValue;
                    groupIDs.addAll(jsonArray.stream()
                            .map(Object::toString)
                            .filter(not(String::isEmpty))
                            .collect(toList()));
                } else if (groupsAttributeValue instanceof String) {
                    // parse a comma-separated list of groups?
                    groupIDs.addAll(Arrays.stream(((String) groupsAttributeValue).split(","))
                            .filter(not(String::isEmpty))
                            .collect(toList()));
                } else {
                    log.error("Could not parse groups from user profile: unexpected \"{}\" attribute type: {} ({})", GROUPS_PROPERTY, groupsAttributeValue, groupsAttributeValue.getClass().getName());
                    return Optional.of(profile);
                }
                log.debug("Group UIDs: {}", groupIDs);
                getAzureAdGroupNamesFromId(bearerToken, tenant, groupIDs, userGroups);
            }

            log.debug("Resulting group list for the logged in user: {}", userGroups);

            if (!userGroups.isEmpty()) {
                profile.addAttribute(MGNL_GROUPS_ATTR_NAME, userGroups);
            }

        } catch (Exception e) {
            log.info("An attempt to retrieve roles or groups for the logged in user failed.", e);
        }

        log.debug("User profile after assigning group membership: {}", profile);
        return Optional.of(profile);
    }

    /**
     * Parse group display names from the MS graph query result.
     *
     * @param jsonObject Query result as JSON object.
     * @return List with user's group names or empty list.
     */
    protected static List<String> getAzureUserGroups(JSONObject jsonObject) {
        List<String> userGroupList = new ArrayList<>();

        Optional<Object> groupArray = Optional.ofNullable(jsonObject.get(MS_GRAPH_VALUE_PROPERTY));
        if (groupArray.isEmpty()) {
            log.error("Could not find the value where the groups are contained in the JSON object.");
        } else {
            JSONArray jsonArray = (JSONArray) groupArray.get();
            jsonArray.forEach(element -> {
                JSONObject obj = (JSONObject) element;
                String displayName = StringUtils.defaultString((String) obj.get(MS_GRAPH_GROUP_NAME_KEY), "");
                if (StringUtils.isNotBlank(displayName)) {
                    log.debug("Fetched group with name {}.", displayName);
                    if (StringUtils.isNotBlank(displayName)) {
                        userGroupList.add(displayName);
                    }
                }
            });
        }
        return userGroupList;
    }

    /**
     * Resolve group names from a list with group UIDs retrieved in the token payload.
     *
     * @param bearerToken Access token.
     * @param tenant      Azure AD tenant provided in config.yaml.
     * @param groupIdList Groups attribute from the token payload containing group UIDs.
     * @param groupList   Resolved group names are added to the resulting group list.
     */
    protected static void getAzureAdGroupNamesFromId(String bearerToken, String tenant, List<String> groupIdList, List<String> groupList) {
        String msGraphUrl = AZURE_GROUP_ID_PREFIX + tenant + AZURE_GROUP_ID_SUFFIX;

        for (String groupId : groupIdList) {
            log.debug("Trying to resolve group with ID {}.", groupId);
            // GET https://graph.microsoft.com/v1.0/<tenant>/groups/<group-uid>
            String queryUrl = msGraphUrl + groupId;
            log.debug("Querying resource {}", queryUrl);
            JSONObject queryResult = queryMsGraph(bearerToken, queryUrl);
            log.debug("Query result: {}", queryResult);

            String displayName = StringUtils.defaultString((String) queryResult.get(MS_GRAPH_GROUP_NAME_KEY), "");
            if (StringUtils.isNotBlank(displayName)) {
                log.debug("Fetched group with name {}.", displayName);
                if (StringUtils.isNotBlank(displayName)) {
                    groupList.add(displayName);
                }
            }
        }
    }

    /**
     * Query MS Graph URL using the access token we retrieved after a successful login.
     *
     * @param bearerToken Access token.
     * @param msGraphUrl  URL to query.
     * @return The response retrieved from Azure as JSON object.
     */
    private static JSONObject queryMsGraph(String bearerToken, String msGraphUrl) {
        JSONObject jsonObjectResult = new JSONObject();

        HttpURLConnection connection = null;
        final Map<String, String> headers = new HashMap<>();
        headers.put(HttpConstants.ACCEPT_HEADER, HttpConstants.APPLICATION_JSON);
        headers.put(HttpConstants.AUTHORIZATION_HEADER, HttpConstants.BEARER_HEADER_PREFIX + bearerToken);

        try {
            connection = openConnection(new URL(msGraphUrl), HttpConstants.HTTP_METHOD.GET.name(), headers);
            final int responseCode = connection.getResponseCode();
            log.debug("Response Code received: {}", responseCode);
            String body = HttpUtils.readBody(connection);
            log.debug("Response body: {}", body);
            log.debug(body);

            jsonObjectResult = (JSONObject) JSONValue.parse(body);

            return jsonObjectResult;
        } catch (Exception e) {
            log.error("Problem while querying MS Graph: ", e);
        } finally {
            HttpUtils.closeConnection(connection);
        }

        return jsonObjectResult;
    }

    // method taken from the PAC4J HttpUtils class (has protected access)
    protected static HttpURLConnection openConnection(final URL url, final String requestMethod, final Map<String, String> headers) throws IOException {
        final HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setDoInput(true);
        connection.setDoOutput(true);
        connection.setRequestMethod(requestMethod);
        connection.setConnectTimeout(HttpConstants.DEFAULT_CONNECT_TIMEOUT);
        connection.setReadTimeout(HttpConstants.DEFAULT_READ_TIMEOUT);
        if (headers != null) {
            for (final Map.Entry<String, String> entry : headers.entrySet()) {
                connection.setRequestProperty(entry.getKey(), entry.getValue());
            }
        }
        return connection;
    }
}
```

---

### Magnolia SSO

I recommend leaving the config.yaml as is and checking the log files. If everything works as expected, you can remove the **fixedRoleAuthorization**:

**config.yaml**

```yaml
# tested with SSO 3.1.3
path: /.magnolia/admincentral
callbackUrl: http://localhost:8080/magnoliaAuthor/.auth
postLogoutRedirectUri: http://localhost:8080/magnoliaAuthor/.magnolia/admincentral
authorizationGenerators:
  - name: dummyAuthorizationGenerator
    
clients:
  oidc.id: 7b5bbb6c-f71f-52e4-b646-d3b332a1c10e
  oidc.secret: TXm9Q~s_tS2iILqPzYm~jqwaGUIQoxyphGKrecip
  oidc.clientAuthenticationMethod: client_secret_post
  oidc.scope: openid profile email
  # use your tenant from Azure AD
  oidc.discoveryUri: https://login.microsoftonline.com/f7c33569-d9fg-87e6-a2af-3e4feq02310c/v2.0/.well-known/openid-configuration
  oidc.preferredJwsAlgorithm: RS256
  oidc.authorizationGenerators: dummyAuthorizationGenerator,CustomAzureAdAuthorizationGeneratorProvider

userFieldMappings:
  name: preferred_username
  removeEmailDomainFromUserName: true
  removeSpecialCharactersFromUserName: false
  fullName: name
  email: email
  language: locale
```

!!! note

    Don't forget to adjust the values from Azure AD including the **TENANT** in oidc.discoveryUri!

    The "dummyAuthorizationGenerator" part or something similar must be kept in the configuration, because without "authorizationGenerators" the SSO module will not start.
    It's also not possible to use the "authorizationGenerators" section in the same way as the default class for your custom authorization generator (adding mappings, etc.).
    On the other hand, you can use a custom authorization generator to get rid of the mappings completely and use groups in Magnolia that encapsulate roles/permissions.    

---

## Testing 

- Test with a user who has **no group assignments** - you should get a 403 error because there are no groups => no permissions.
- Test with a user who has groups that match the Magnolia groups, e.g. **travel-demo-editors** => login with correct permissions (the local Magnolia groups must match).

Check [Debugging](/Magnolia%20SSO%20Module/Troubleshooting/debugging/) if it does not work …

## Finally

If everything works as it should, groups can also be a very relaxing experience ...

![Dogs group](_img/azuread-groups/group_work.jpg)