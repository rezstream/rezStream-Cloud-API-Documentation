# rezStream Cloud API Documentation

> [!IMPORTANT]
> This is an early-stage document that is likely to evolve and change, possibly quickly. So, please be sure to check back frequently for updates. If you have any questions or comments, please feel free to reach out to the rezStream development team at development@rezstream.com.

## Authentication

External authentication for rezStream Cloud services can be done through the new https://account.rezstream.com/ site. We don’t currently use this as our primary means of authentication, but it will soon be part of a unified authentication experience for both external applications and all rezStream Cloud systems. As a starting point for OAuth and OIDC, we provide a well-known configuration endpoint which facilitates endpoint and public key discovery: https://account.rezstream.com/.well-known/openid-configuration . The account application can be used to:
1. Manually generate tenant API access tokens
2. Generate API access tokens via OAuth
3. Support OIDC sign-on scenarios using rezStream accounts

Different kinds of tokens in different formats for different contexts can be issued through the user interface and the OAuth endpoints. There are effectively two kinds of tokens issued from the account application. 
1. The first kind of token is for API usage where the subject of the token is the tenant/business itself and is in a reference token format. 
2. The second is an access token for rare single sign-on integrations where the subject is a user, and it doubles as an identity token using a JWT format. 

## Manual API Token Generation 

Long-lived API access tokens providing access to a single rezStream Cloud tenant can be created and revoked manually within the account portal. This can be handy for bootstrapping development and for small custom integrations but is not advisable for large scale integrations with multiple customers and tenants. The generated reference token from this process should be kept private and only transmitted to our API endpoints from a secured system. To create a token for access to a specific tenant manually: 

1. Login to https://account.rezstream.com/ using an account which has user management access to the subject business or tenant.
2. Navigate to the **Businesses** section and then to the business you wish to manage 
3. If you have appropriate access, you should see a link to **Manage app authorizations**. Navigate there. 
4. On the **Manage integrations** page you can select the required scopes for the new token and click the **Generate Token** button. 
5. After the token is generated, a token string is presented to you. This is the only opportunity to retrieve the private and secret token value, so copy it somewhere safe. 
6. Click **Continue** to return to the **Manage integrations** page. 
   * At any time, you can revoke the generated token from this page.

## OAuth API Authorization and Token Generation 

A much more secure, scalable, and user-friendly workflow to generate refresh tokens and short-lived access tokens is through the OAuth authorization code flow. These tokens provide scoped access to a single rezCloud tenant, and contain no information embedded within them. When multiple tenants are available to a user during the authorization process, the user can select which tenant will be the subject for the integration authorization and resulting tokens. After an integration is approved, the standard authorization code flow process takes place and can be used to generate a refresh token and/or access token. 

While it is not required, we (very) strongly recommend that integrations use refresh tokens as the access tokens have much more limited lifetime. When relying only on access tokens a user will repeatedly have to re-authorize integrations to generate access tokens. Refresh tokens allow the integrating service to continually maintain and cycle the used tokens to reduce user interaction and reduce the risks in the event an access token is exposed. Refresh tokens can be selected by including the `offline_access` scope when initiating a flow. 

Before getting started with this style of integration, reach out to us so we can register an integration for you within our internal systems. We will need a few things from you to do so: 

* A business relationship of some kind 
* A list of redirect URLs that would be used in various OAuth flows 
* A description of how the integration will technically function 

📱 **Note for mobile & browser integrations:** For security reasons, we do not support OAuth flows directly from end-user devices. If there is a need for this kind of scenario, reach out and we might be able to work something out. Ideally, we would like API integrations to be from secured services communicating directly to our service endpoints. 

When we are ready to integrate, we can supply you with the client ID and client secret that are required for the OAuth flows. The OAuth endpoints are already publicly available in the “well-known” document: https://account.rezstream.com/.well-known/openid-configuration. 

## Postman Example 

> [!NOTE]
> This requires us to add postman’s redirect service as an allowed redirect URL for your integration. We might not permit this redirect URL permanently for a production client ID as a security measure. 

Postman makes testing the integration very easy. Within the **Authorization** section of a request or collection, select `OAuth 2.0` as the **Type**. Finally, as the meme goes, *“draw the rest of the owl”* by configuring the remainder of the OAuth section with the values shown below. Substitute your desired values for `Auth URL` and `Access Token URL` from the well-known document, the values for `Client ID` and `Client Secret` from what we give you, and then season the rest to taste. Reach out to us if you need any additional integration assistance. 

![image](https://github.com/rezstream/rezStream-Cloud-API-Documentation/assets/1365831/d7ab2569-2bda-4175-95e9-7cabe104edd1)

After the OAuth 2 section is configured, clicking **Get New Access Token** will open a web browser to start the authorization code flow. After authenticating, you can approve the integration for a specific tenant. Granting access will trigger the remaining parts of the flow and provide Postman with an access token and if requested, a refresh token. The access token can be used by associated Postman requests to authenticate against our rezStream Cloud API.