---
title: Partner access through Microsoft Defender XDR APIs
description: Learn how to create an app to get programmatic access to Microsoft Defender XDR on behalf of your users.
ms.service: defender-xdr
f1.keywords: 
  - NOCSH
ms.author: macapara
author: mjcaparas
ms.localizationpriority: medium
manager: dansimp
audience: ITPro
ms.collection: 
 - m365-security
 - tier3
 - must-keep
ms.topic: reference
search.appverid: 
  - MOE150
  - MET150
ms.custom: api
ms.date: 02/16/2021
---

# Create an app with partner access to Microsoft Defender XDR APIs

[!INCLUDE [Microsoft Defender XDR rebranding](../includes/microsoft-defender.md)]

**Applies to:**

- Microsoft Defender XDR

> [!IMPORTANT]
> Some information relates to prereleased product which may be substantially modified before it's commercially released. Microsoft makes no warranties, express or implied, with respect to the information provided here.

This page describes how to create a Microsoft Entra app that has programmatic access to Microsoft Defender XDR, on behalf of users across multiple tenants. Multi-tenant apps are useful for serving large groups of users.

If you need programmatic access to Microsoft Defender XDR on behalf of a single user, see [Create an app to access Microsoft Defender XDR APIs on behalf of a user](api-create-app-user-context.md). If you need access without a user explicitly defined (for example, if you're writing a background app or daemon), see [Create an app to access Microsoft Defender XDR without a user](api-create-app-web.md). If you're not sure which kind of access you need, see [Get started](api-access.md).

Microsoft Defender XDR exposes much of its data and actions through a set of programmatic APIs. Those APIs help you automate workflows and make use of Microsoft Defender XDR's capabilities. This API access requires OAuth2.0 authentication. For more information, see [OAuth 2.0 Authorization Code Flow](/azure/active-directory/develop/active-directory-v2-protocols-oauth-code).

In general, you'll need to take the following steps to use these APIs:

- Create a Microsoft Entra application.
- Get an access token using this application.
- Use the token to access Microsoft Defender XDR API.

Since this app is multi-tenant, you'll also need [admin consent](/azure/active-directory/develop/v2-permissions-and-consent#requesting-consent-for-an-entire-tenant) from each tenant on behalf of its users.

This article explains how to:

- Create a **multi-tenant** Microsoft Entra application
- Get authorized consent from your user administrator for your application to access the Microsoft Defender XDR that resources it needs.
- Get an access token to Microsoft Defender XDR
- Validate the token

Microsoft Defender XDR exposes much of its data and actions through a set of programmatic APIs. Those APIs will help you automate work flows and innovate based on Microsoft Defender XDR capabilities. The API access requires OAuth2.0 authentication. For more information, see [OAuth 2.0 Authorization Code Flow](/azure/active-directory/develop/active-directory-v2-protocols-oauth-code).

In general, you'll need to take the following steps to use the APIs:

- Create a **multi-tenant** Microsoft Entra application.
- Get authorized (consent) by your user administrator for your application to access Microsoft Defender XDR resources it needs.
- Get an access token using this application.
- Use the token to access Microsoft Defender XDR API.

The following steps with guide you how to create a multi-tenant Microsoft Entra application, get an access token to Microsoft Defender XDR and validate the token.

## Create the multi-tenant app

1. Sign in to [Azure](https://portal.azure.com) as a user with the **Global Administrator** role.

2. Navigate to **Microsoft Entra ID** > **App registrations** > **New registration**.

   :::image type="content" source="../../media/atp-azure-new-app2.png" alt-text="An application's registration section in the Microsoft Defender portal" lightbox="../../media/atp-azure-new-app2.png":::

3. In the registration form:

   - Choose a name for your application.
   - From **Supported account types**, select **Accounts in any organizational directory (Any Microsoft Entra directory) - Multitenant**.
   - Fill out the **Redirect URI** section. Select type **Web** and give the redirect URI as **https://portal.azure.com**.

   After you're done filling out the form, select **Register**.

   :::image type="content" source="../../media/atp-api-new-app-partner.png" alt-text="An application's registration sections in the Microsoft Defender portal" lightbox="../..//media/atp-api-new-app-partner.png":::

4. On your application page, select **API Permissions** > **Add permission** > **APIs my organization uses** >, type **Microsoft Threat Protection**, and select **Microsoft Threat Protection**. Your app can now access Microsoft Defender XDR.

   > [!TIP]
   > *Microsoft Threat Protection* is a former name for Microsoft Defender XDR, and will not appear in the original list. You need to start writing its name in the text box to see it appear.

   :::image type="content" source="../../media/apis-in-my-org-tab.PNG" alt-text="The APIs usage section in the Microsoft Defender portal" lightbox="../../media/apis-in-my-org-tab.PNG":::

5. Select **Application permissions**. Choose the relevant permissions for your scenario (for example, **Incident.Read.All**), and then select **Add permissions**.

   :::image type="content" source="../../media/request-api-permissions.PNG" alt-text="An application's permission pane in the Microsoft Defender portal" lightbox="../../media/request-api-permissions.PNG":::

    > [!NOTE]
    > You need to select the relevant permissions for your scenario. *Read all incidents* is just an example. To determine which permission you need, please look at the **Permissions** section in the API you want to call.
    >
    > For instance, to [run advanced queries](api-advanced-hunting.md), select the 'Run advanced queries' permission; to [isolate a device](/windows/security/threat-protection/microsoft-defender-atp/isolate-machine), select the 'Isolate machine' permission.

6. Select **Grant admin consent**. Every time you add a permission, you must select **Grant admin consent** for it to take effect.

    :::image type="content" source="../../media/grant-consent.PNG" alt-text="A section to grant admin consent in the Microsoft Defender portal" lightbox="../../media/grant-consent.PNG":::

7. To add a secret to the application, select **Certificates & secrets**, add a description to the secret, then select **Add**.

    > [!TIP]
    > After you select **Add**, select **copy the generated secret value**. You won't be able to retrieve the secret value after you leave.

      :::image type="content" source="../../media/webapp-create-key2.png" alt-text="The Secret addition section in the Microsoft Defender portal" lightbox="../../media/webapp-create-key2.png":::

8. Record your application ID and your tenant ID somewhere safe. They're listed under **Overview** on your application page.

   :::image type="content" source="../../media/app-and-tenant-ids.png" alt-text="The Overview pane in the Microsoft Defender portal" lightbox="../../media/app-and-tenant-ids.png":::

9. Add the application to your user's tenant.

   Since your application interacts with Microsoft Defender XDR on behalf of your users, it needs be approved for every tenant on which you intend to use it.

   A **Global Administrator** from your user's tenant needs to view the consent link and approve your application.

   Consent link is of the form:

   ```HTTP
   https://login.microsoftonline.com/common/oauth2/authorize?prompt=consent&client_id=00000000-0000-0000-0000-000000000000&response_type=code&sso_reload=true
   ```

   The digits `00000000-0000-0000-0000-000000000000` should be replaced with your Application ID.

   After clicking on the consent link, sign in with the Global Administrator of the user's tenant and consent the application.

   :::image type="content" source="../../media/app-consent-partner.png" alt-text="The consent application page in the Microsoft Defender portal" lightbox="../../media/app-consent-partner.png":::

   You'll also need to ask your user for their tenant ID. The tenant ID is one of the identifiers used to acquire access tokens.

- **Done!** You've successfully registered an application!
- See examples below for token acquisition and validation.

## Get an access token

For more information on Microsoft Entra tokens, see the [Microsoft Entra tutorial](/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds).

> [!IMPORTANT]
> Although the examples in this section encourage you to paste in secret values for testing purposes, you should **never hardcode secrets** into an application running in production. A third party could use your secret to access resources. You can help keep your app's secrets secure by using [Azure Key Vault](/azure/key-vault/general/about-keys-secrets-certificates). For a practical example of how you can protect your app, see [Manage secrets in your server apps with Azure Key Vault](/training/modules/manage-secrets-with-azure-key-vault/).

> [!TIP]
> In the following examples, use a user's tenant ID to test that the script is working.

### Get an access token using PowerShell

```PowerShell
# This code gets the application context token and saves it to a file named "Latest-token.txt" under the current directory.

$tenantId = '' # Paste your directory (tenant) ID here
$clientId = '' # Paste your application (client) ID here
$appSecret = '' # Paste your own app secret here to test, then store it in a safe place!

$resourceAppIdUri = 'https://api.security.microsoft.com'
$oAuthUri = "https://login.windows.net/$tenantId/oauth2/token"

$authBody = [Ordered] @{
    resource = $resourceAppIdUri
    client_id = $clientId
    client_secret = $appSecret
    grant_type = 'client_credentials'
}

$authResponse = Invoke-RestMethod -Method Post -Uri $oAuthUri -Body $authBody -ErrorAction Stop
$token = $authResponse.access_token

Out-File -FilePath "./Latest-token.txt" -InputObject $token

return $token
```

### Get an access token using C\#

> [!NOTE]
> The following code was tested with Nuget Microsoft.Identity.Client 3.19.8.

> [!IMPORTANT]
> The [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory) NuGet package and Azure AD Authentication Library (ADAL) have been deprecated. No new features have been added since June 30, 2020.   We strongly encourage you to upgrade, see the [migration guide](/azure/active-directory/develop/msal-migration) for more details.

1. Create a new console application.
1. Install NuGet [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/).
1. Add the following line:

    ```C#
    using Microsoft.Identity.Client;
    ```

1. Copy and paste the following code into your app (don't forget to update the three variables: `tenantId`, `clientId`, `appSecret`):

    ```C#
    string tenantId = "00000000-0000-0000-0000-000000000000"; // Paste your own tenant ID here
    string appId = "11111111-1111-1111-1111-111111111111"; // Paste your own app ID here
    string appSecret = "22222222-2222-2222-2222-222222222222"; // Paste your own app secret here for a test, and then store it in a safe place! 
    const string authority = https://login.microsoftonline.com;
    const string audience = https://api.securitycenter.microsoft.com;

    IConfidentialClientApplication myApp = ConfidentialClientApplicationBuilder.Create(appId).WithClientSecret(appSecret).WithAuthority($"{authority}/{tenantId}").Build();

    List<string> scopes = new List<string>() { $"{audience}/.default" };

    AuthenticationResult authResult = myApp.AcquireTokenForClient(scopes).ExecuteAsync().GetAwaiter().GetResult();

    string token = authResult.AccessToken;
    ```

### Get an access token using Python

```Python
import json
import urllib.request
import urllib.parse

tenantId = '' # Paste your directory (tenant) ID here
clientId = '' # Paste your application (client) ID here
appSecret = '' # Paste your own app secret here to test, then store it in a safe place, such as the Azure Key Vault!

url = "https://login.windows.net/%s/oauth2/token" % (tenantId)

resourceAppIdUri = 'https://api.security.microsoft.com'

body = {
    'resource' : resourceAppIdUri,
    'client_id' : clientId,
    'client_secret' : appSecret,
    'grant_type' : 'client_credentials'
}

data = urllib.parse.urlencode(body).encode("utf-8")

req = urllib.request.Request(url, data)
response = urllib.request.urlopen(req)
jsonResponse = json.loads(response.read())
aadToken = jsonResponse["access_token"]
```

### Get an access token using curl

> [!NOTE]
> Curl is pre-installed on Windows 10, versions 1803 and later. For other versions of Windows, download and install the tool directly from the [official curl website](https://curl.haxx.se/windows/).

1. Open a command prompt, and set CLIENT_ID to your Azure application ID.
1. Set CLIENT_SECRET to your Azure application secret.
1. Set TENANT_ID to the Azure tenant ID of the user that wants to use your app to access Microsoft Defender XDR.
1. Run the following command:

```bash
curl -i -X POST -H "Content-Type:application/x-www-form-urlencoded" -d "grant_type=client_credentials" -d "client_id=%CLIENT_ID%" -d "scope=https://securitycenter.onmicrosoft.com/windowsatpservice/.default" -d "client_secret=%CLIENT_SECRET%" "https://login.microsoftonline.com/%TENANT_ID%/oauth2/v2.0/token" -k
```

A successful response will look like this:

```bash
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIn <truncated> aWReH7P0s0tjTBX8wGWqJUdDA"}
```

## Validate the token

1. Copy and paste the token into the [JSON web token validator website, JWT,](https://jwt.ms) to decode it.
1. Make sure that the *roles* claim within the decoded token contains the desired permissions.

In the following image, you can see a decoded token acquired from an app, with ```Incidents.Read.All```, ```Incidents.ReadWrite.All```, and ```AdvancedHunting.Read.All``` permissions:

:::image type="content" source="../../media/webapp-decoded-token.png" alt-text="The Decoded Token pane in the Microsoft Defender portal" lightbox="../../media/webapp-decoded-token.png":::

<a name='use-the-token-to-access-the-microsoft-365-defender-api'></a>

## Use the token to access the Microsoft Defender XDR API

1. Choose the API you want to use (incidents, or advanced hunting). For more information, see [Supported Microsoft Defender XDR APIs](api-supported.md).
2. In the http request you're about to send, set the authorization header to `"Bearer" <token>`, *Bearer* being the authorization scheme, and *token* being your validated token.
3. The token will expire within one hour. You can send more than one request during this time  with the same token.

The following example shows how to send a request to get a list of incidents **using C#**.

```C#
   var httpClient = new HttpClient();
   var request = new HttpRequestMessage(HttpMethod.Get, "https://api.security.microsoft.com/api/incidents");

   request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);

   var response = httpClient.SendAsync(request).GetAwaiter().GetResult();
```

## Related articles

- [Microsoft Defender XDR APIs overview](api-overview.md)
- [Access the Microsoft Defender XDR APIs](api-access.md)
- [Create a 'Hello world' application](api-hello-world.md)
- [Create an app to access Microsoft Defender XDR without a user](api-create-app-web.md)
- [Create an app to access Microsoft Defender XDR APIs on behalf of a user](api-create-app-user-context.md)
- [Learn about API limits and licensing](api-terms.md)
- [Understand error codes](api-error-codes.md)
- [Manage secrets in your server apps with Azure Key Vault](/training/modules/manage-secrets-with-azure-key-vault/)
- [OAuth 2.0 authorization for user sign in and API access](/azure/active-directory/develop/active-directory-v2-protocols-oauth-code)
[!INCLUDE [Microsoft Defender XDR rebranding](../../includes/defender-m3d-techcommunity.md)]
