# rezStream Cloud API Documentation

> [!IMPORTANT]
> This is an early-stage document that is likely to evolve and change, possibly quickly. So, please be sure to check back frequently for updates. If you have any questions or comments, please feel free to reach out to the rezStream development team at development@rezstream.com.

## Contents
* [Authentication](#authentication)
* [Manual API Token Generation](#manual-api-token-generation)
* [OAuth API Authorization and Token Generation](#oauth-api-authorization-and-token-generation)
* [Postman Example](#postman-example)
* [rezStream Cloud API](#rezstream-cloud-api)
   * [Versioning](#versioning)
   * [Permissions](#permissions)
 * [The Data and Endpoints](#the-data-and-endpoints)
   * [Reservations](#reservations)
      * [Finding Reservations](#finding-reservations)
         * [By Last Update](#by-last-update)
         * [By Stay Date](#by-stay-date)
      * [Reservation Details](#reservation-details)

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

A much more secure, scalable, and user-friendly workflow to generate refresh tokens and short-lived access tokens is through the OAuth authorization code flow. These tokens provide scoped access to one or more rezCloud tenants, and contain no information embedded within them. When multiple tenants are available to a user during the authorization process, the user can select which tenant or tenants will be the subject for the integration authorization and resulting token. After an integration is approved, the standard authorization code flow process takes place and can be used to generate a refresh token and/or access token. 

While it is not required, we very strongly recommend that integrations utilize refresh tokens to generate new access tokens after expiration. When relying only on access tokens a user will repeatedly have to re-authorize integrations to generate access tokens. Refresh tokens allow the integrating service to continually maintain and cycle the access tokens automatically to reduce user interaction and reduce the risks in the event an access token is exposed. Refresh tokens can be selected by including the `offline_access` scope when initiating an OAuth flow.

Before getting started with this style of integration, reach out to us so we can register an integration for you within our internal systems. We will need a few things from you to do so:

* A business relationship of some kind 
* A list of redirect URLs that would be used in various OAuth flows 
* A description of how the integration will technically function

> [!NOTE]
> 📱 **For mobile & browser integrations:** For security reasons, we do not support OAuth flows directly from end-user devices. If there is a need for this kind of scenario, contact us and we might be able to work something out. Ideally, we would like API integrations to be from secured services communicating directly to our API endpoints.

When we are ready to integrate, we can supply you with the client ID and client secret that are required for the OAuth flows. Our current OAuth endpoints can be found in the server metadata document: https://account.rezstream.com/.well-known/openid-configuration .

### Multi-Tenant OAuth Tokens

While we prefer each token to have access to a single tenant, our system does support multi-tenant access for a token for cases where this is required. We treat single tenant tokens as the default and provide the smoothest path for them but this approach adds ambiguity for tokens associated with multiple tenants requiring more explicit information in requests.

To request multi-tenant support during OAuth flows, add the custom `multiTenant=true` argument to the initial OAuth request. In Postman, this can be added to the "Auth Request" section of the OAuth tool. This will allow a user to select multiple tenants in our system when they have permissions to more than one tenant.


### Multi-Tenant API Token Usage

In the Cloud API, `/auth/tokendetails` can be used to show which tenants a token has access to, and some information about those tenants. This can be used to detect if a token is actually multi-tenant.

When making a request to other Cloud API endpoints with a multi-tenant token, the request may be ambiguous and result in an HTTP 401 response. To resolve this ambiguity, a `X-RezStream-Tenant-ID` header must be sent as part of each request with a relevant Tenant ID value associated with that token.

## Postman Example 

> [!NOTE]
> This requires us to add postman’s redirect service as an allowed redirect URL for your integration. We might not permit this redirect URL permanently for a production client ID as a security measure. 

Postman makes testing the integration very easy. Within the **Authorization** section of a request or collection, select `OAuth 2.0` as the **Type**. Finally, as the meme goes, *“draw the rest of the owl”* by configuring the remainder of the OAuth section with the values shown below. Substitute your desired values for `Auth URL` and `Access Token URL` from the well-known metadata document, the values for `Client ID` and `Client Secret` from what we give you, and then season the remaining inputs to taste. Reach out to us if you need any additional integration assistance. Our OAuth endpoint also supports additional custom arguments which can be supplied in the "Auth Request" section of the postman OAuth tool.

![image](https://github.com/rezstream/rezStream-Cloud-API-Documentation/assets/1365831/d7ab2569-2bda-4175-95e9-7cabe104edd1)

After the OAuth 2 section is configured, clicking **Get New Access Token** will open a web browser to start the authorization code flow. After authenticating, you can approve the integration for a specific tenant. Granting access will trigger the remaining parts of the flow and provide Postman with an access token and if requested, a refresh token. The access token can be used by associated or child Postman requests to authenticate against our rezStream Cloud API.

# rezStream Cloud API

After obtaining an access token with sufficient permissions, queries to the rezStream Cloud API will be permitted. So far, all endpoints only support querying data from the system. In the future we will support the submission of commands and data mutations. We provide Open API documents for our API which can be used to get an overview of the offered endpoints: https://cloudapi.rezstream.com/openapi. The Open API documents aren’t the most human readable formats, but it outlines the supported endpoints and requests.

Each request sent to the API **requires**:
* A valid "Bearer" token in the `Authorization` header
* An additional field in the Headers section with key: `X-RezStream-Api-Version` and value: `<version>`. Note the available versions can be found [here]([url](https://cloudapi.rezstream.com/openapi)) and the value must be a date in "YYYY-MM-DD" format.

## Versioning

We use a date style of versioning and require you to send us an explicit version header for each request. When we make incremental and “safe” changes to response models and API endpoints we may allow those changes to apply to older API versions. However, changes considered breaking should be isolated from older versions, where reasonable. This is a balance between providing a stable yet easy to use and maintainable API for integrations.

## Permissions

Each access token has permissions associated with it. A token that only has access to read reservations or invoices will fail with an HTTP 403 response when used to make modifications. These permissions are set upon authorizing an integration, so consider the requested scopes carefully when establishing an integration.

## Pagination

We try to avoid extraneous structure in our responses and that extends to paged responses. Furthermore, different endpoints may use different paging strategies even. For these reasons, you can find RFC 8288/5988 links in the response headers that show how to find the next, prev, first, and last pages when applicable. These are especially useful with cursor styles of pagination which are more complex than page number and page size. The response body for a result page should be a direct collection of the data itself without extra pagination metadata.

Example of pagination response headers:
```
Link: <https://cloudapi.rezstream.com/reservations/byDate?from=2023-01-01&to=2024-01-01&skip=20&limit=10>; rel="next"
Link: <https://cloudapi.rezstream.com/reservations/byDate?from=2023-01-01&to=2024-01-01&skip=0&limit=10>; rel="prev"
Link: <https://cloudapi.rezstream.com/reservations/byDate?from=2023-01-01&to=2024-01-01&skip=30&limit=10>; rel="last"
Link: <https://cloudapi.rezstream.com/reservations/byDate?from=2023-01-01&to=2024-01-01&skip=0&limit=10>; rel="first"
```

Example of paged cursor response headers:
```
Link: <https://cloudapi.rezstream.com/reservations/updated?from=2020-01-01T00%3A00%3A00Z&afterAt=2021-10-01T14%3A44%3A54.03Z&afterId=fa801bc8-92af-48d3-9ee4-a8685f794904&limit=10>; rel="next"
Link: <https://cloudapi.rezstream.com/reservations/updated?from=2020-01-01T00%3A00%3A00Z&limit=10>; rel="first"
```

# The Data and Endpoints

## Reservations

### Finding Reservations

We currently have two endpoints to find reservations in our system; “by stay dates” and “by last update timestamp”.

#### By Last Update

The list of reservations by update timestamp can be queried from a starting date and uses forward only pagination due to the shifting nature of the data. This can be a handy endpoint to use when polling for new or modified reservations. The simplest request can start with `/reservations/updated?from=2023-01-01T00:00:00Z&limit=20` and will provide the reservations in ascending update order. In the future, web-hooks should make this endpoint redundant. The design of the cursor pagination for this endpoint supports updates made concurrently during a query making this a reliable way to detect new changes when polling.

#### By Stay Date

Another method of locating reservations is to use the stay based window query, searching by date. This can be useful when you need perform an initial population of data or quickly ensure you have the latest data for a window of dates. Note that the input date values are in the local time zone of the tenant. The simplest query for this endpoint is `/reservations/byDate?from=2023-01-01&to=2024-01-01&limit=20` and pagination can provide more results if they exist.

### Reservation Details

Given a reservation ID, you can get a lot of information about a reservation. A basic query for this looks like `/reservations/a2857289-0f63-4a26-b509-444699485a33` and will return a large body of information about the reservation if found.

Contact us for any clarification on the data in this payload. Some important points to note, especially for large group reservations:
* A reservation can have multiple unit assignments
* A reservation can have a group of multiple people
* Different people can be assigned to different unit assignments
* Invoices can be assigned to different members of the group
* Different unit assignments can be associated with different invoices
* People may have a different mobile phone number from their regular phone number
* Some people don’t have a mobile phone number
* Unit assignments can have different dates of stay
