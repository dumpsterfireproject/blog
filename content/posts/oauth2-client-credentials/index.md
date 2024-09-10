---
title: "Enhancing B2B Application Functionality with OAuth2 Client Credentials Flow"
date: 2024-09-09
slug: "/client-credentials"
description: Discover the technical intricacies of enabling users to generate their own OAuth2 client IDs and secrets using Microsoft Graph API and Azure B2C.
tags:
  - Azure
  - OAuth2
  - MSGraphAPI
---
In response to a recent feature request aimed at bolstering the functionality of our B2B application,
we embarked on a journey to support an OAuth2 client credentials flow for our customers authentication use cases. 
Leveraging our existing use of Azure B2C as our identity provider, we undertook the challenge of enabling users to
benefit from this authentication mechanism.

Our solution hinged on the capabilities of the MS Graph API, which provided the backbone for our implementation. 
Central to our approach was the utilization of the `/applications` endpoint within the MS Graph API. 
This endpoint enabled us to dynamically create new applications on behalf of users. 
Our web application provided a form which allowed our users to choose their own application name and secret name. 
Behind this form was a mutation in our GraphQL API, which triggered a POST to the `/applications` endpoint 
to create the new application in Azure B2C. 
The OID of the new application was associated with the user's subscription in our database for subsequent use.

To create the new client application in the MS Graph API, the body of the post contained the display name entered by the user.
We also wanted to specify the [signInAudience](https://learn.microsoft.com/en-us/graph/api/resources/application?view=graph-rest-1.0#signinaudience-values)
to allow "Accounts in any organizational directory and personal Microsoft accounts." If not specified, the default is AzureADMyOrg,
which resulted in an error "invalid_grant" with a message of `"AADB2C90085: The service has encountered an internal error. Please reauthenticate and try again."` when making the post to create a new application/client_id.
Lastly, we wanted to minimize the amount of manual configuration and approval required when a user created a new client.
The new client needed api.read and api.write privileges for the application it would be interacting with. So we injected
the app_id via an environment variable and assigned it to a variable named "resourceAppId". Likewise, we injected the role IDs
for the api.read and api.write access to that resource and assigned to variables named "appReadRoleID" and "appWriteRoleID", respectively.
So a sample body is as follows.
```json
{
  "displayName": "<display name entered by the user>",
  "signInAudience": "AzureADandPersonalMicrosoftAccount",
  "requiredResourceAccess": [
    {
      "resourceAppID": "<resourceAppId value>",
      "resourceAccess": [
        {"id":"<appWriteRoleID value>","type":"Role"},
        {"id":"<appReadRoleID value>","type":"Role"}
      ]
    }
  ]
}
```
Note that the app ID returned in the response will be the value the user will need when performing the client credentials
OAuth2 flow. Also note that though we were able to set all the values programatically, we still needed to log into Azure B2C
via the Azure Portal and manually grant consent for the application. We made sure to let the users know client applications
created will not be immediately available for authentication and clearly stated the SLA with which we would grant consent
for the new client application. Automated notifcations upon the creation of the client application help to adhere to the SLAs.

Next to create the client secret, a POST was performed against `/applications/<OID of the client application>/addPassword`.  
Make sure you use the OID and not the app ID. The body was simply as follows.
```json
{ "passwordCredential": { "displayName": "<secret name entered by the user>" } }
```

Note that in the payload of the response, the value of the secret created will be returned. 
We display it to the user at this point and inform that this is the only time this value will be available, 
so they will need to copy the value now. It will be in the SecretText field of the response.

If the user wants to delete a secret, perform a POST against `/applications/<OID of the client application>/removePassword`.
If the user wants to delete a client application, perform a DELETE against `/applications/<OID of the client application>`.

When retrieving information for display for the user, we can include the password credentials in the response.
This will get the secret names, the hint, and the expiration date, which will help the users manage their
client secrets. However, it will not include the value of the secret, as you cannot request that via
the MS Graph API. Again, the secret value is only included in the POST to addPassword.
The GET with query parameters were formatted as follows.

`/applications/<OID of the client application>?$select=id,appId,displayName,createdDateTime,passwordCredentials`

By using these calls to the MS Graph API, you can enable your users to create and manage their own client applications
and secrets and integrate with your application in a safe and secure fashion utilizing the OAuth2 Client Credentials flow.
