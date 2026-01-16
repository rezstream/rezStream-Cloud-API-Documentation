# rezStream Cloud API Documentation

> [!IMPORTANT]
> This document is likely to evolve and change. Please be sure to check back for updates. If you have any questions or comments, please feel free to reach out to the rezStream development team at development@rezstream.com.

## Contents
* [Authentication and Authorization](#authentication-and-authorization)
* [Testing with Postman](#testing-with-postman)
* [rezStream Cloud API](#rezstream-cloud-api)
* [The Data and Endpoints](#the-data-and-endpoints)
   * [Inventory](#inventory)
   * [Availability](#availability)
   * [Rates](#rates)
   * [Reservations](#reservations)

## Authentication and Authorization

External authentication for rezStream Cloud accounts can be performed using [https://account.rezstream.com/](https://account.rezstream.com/). As a starting point for OAuth and OIDC, we provide an [openid-configuration](https://account.rezstream.com/.well-known/openid-configuration) endpoint which provides information about OAuth endpoints, public keys, and OAuth configuration. The account site can be used to:

1. Manually generate tenant API access tokens
2. Generate API access tokens via OAuth
3. Support OIDC sign-on scenarios using rezStream accounts

Different kinds of tokens in different formats for different contexts can be issued through the user interface and the OAuth endpoints. The following are examples of different scenarios:

- OAuth API tokens providing access to one or more tenants.
- Manually generated access tokens for testing or extremely specialized integration scenarios.
- Single sign-on scenarios where the subject is a user, providing a reference access token and [JWT](https://jwt.io/) identity token.

### Manual API Token Generation

Long-lived API access tokens providing access to a single rezStream Cloud tenant can be created and revoked manually within the account portal. This can be handy for bootstrapping development and for small custom integrations but is not advisable for large scale integrations with multiple customers and tenants. The generated access token from this process is extremely long lived, so handle and transmit it with great care. To create a token for access to a specific tenant manually: 

1. Login to https://account.rezstream.com/ using an account which has user management access to the subject business or tenant.
2. Navigate to the **Businesses** section and then to the business you wish to manage 
3. If you have appropriate access, you should see a link to **Manage app authorizations**. Navigate there. 
4. On the **Manage integrations** page you can select the required scopes for the new token and click the **Generate Token** button. 
5. After the token is generated, a token string is presented to you. This is the only opportunity to retrieve the private and secret token value, so copy it somewhere safe. 
6. Click **Continue** to return to the **Manage integrations** page.
   * At any time, you can revoke the generated token from this page. This is strongly recommended when the token is no longer needed.

### OAuth API Authorization and Token Generation

A much more secure, scalable, and user-friendly workflow to generate refresh tokens and short-lived access tokens is through the OAuth authorization code flow. These tokens provide scoped access to one or more rezCloud tenants, and contain no information embedded within them. When multiple tenants are available to a user during the authorization process, the user can select which tenant or tenants will be the subject for the integration authorization and resulting token. After an integration is approved, the standard authorization code flow process takes place and can be used to generate a refresh token and/or access token. 

When initiating an authorization request, scopes must be provided. A list of supported scopes can be found in the [/.well-known/openid-configuration](https://account.rezstream.com/.well-known/openid-configuration) document. An example of scopes for an API integration might be `offline_access guest_communication` which would request approval to access reservation data and also enables refresh tokens.

We strongly recommend that integrations utilize refresh tokens to generate new access tokens after expiration. When relying only on access tokens a user will repeatedly have to re-authorize integrations to generate access tokens. Refresh tokens allow the integrating service to continually maintain and cycle the access tokens automatically to reduce user interaction and reduce the risks in the event an access token is exposed. Refresh tokens can be selected by including the `offline_access` scope in the list of scopes when initiating an OAuth flow.

Before getting started with this style of integration, reach out to us so we can register an integration for you within our internal systems. We will need a few things from you to do so:

* A business relationship of some kind 
* A list of redirect URLs that would be used in various OAuth flows 
* A description of how the integration will technically function

> [!NOTE]
> 📱 **For mobile & browser integrations:** For security reasons, we do not support OAuth flows directly from end-user devices. If there is a need for this kind of scenario, contact us and we might be able to work something out. Ideally, we would like API integrations to be from secured services communicating directly to our API endpoints.

When we are ready to integrate, we can supply you with the client ID and client secret that are required for the OAuth flows. Our current OAuth endpoints can be found in the server metadata document: https://account.rezstream.com/.well-known/openid-configuration .

### Scopes

There are a number of possible scopes that can be provided when requesting a token. Those scopes can be found within the OpenID well known configuration document. It is strongly recommended that scopes focused on use cases be used over scopes focused on access roles. For example, scopes such as `guest_communication` or `revenue_management` should be preferred to other more granular scopes. This will provide a better user experience and also allows authorizations to better grow with the API.

### Multi-Tenant OAuth Tokens

While we prefer each token to have access to a single tenant, our system does support multi-tenant access for a token for cases where this is required. We treat single tenant tokens as the default and provide the smoothest path for them but this approach adds ambiguity for tokens associated with multiple tenants requiring more explicit information in requests.

To request multi-tenant support during OAuth flows, add the custom `multiTenant=true` argument to the initial OAuth request. In Postman, this can be added to the "Auth Request" section of the OAuth tool. This will allow a user to select multiple tenants in our system when they have permissions to more than one tenant.


### Multi-Tenant API Token Usage

In the Cloud API, `/auth/tokendetails` can be used to show which tenants a token has access to, and some information about those tenants. This can be used to detect if a token is actually multi-tenant.

When making a request to other Cloud API endpoints with a multi-tenant token, the request may be ambiguous and result in an HTTP 401 response. To resolve this ambiguity, a `X-RezStream-Tenant-ID` header must be sent as part of each request with a relevant Tenant ID value associated with that token.

## Testing with Postman

> [!NOTE]
> Postman requires us to add postman’s redirect service as an allowed redirect URL for your integration. We might not permit this redirect URL permanently for a production client ID as a security measure.

### Postman Collection Import

After importing the [Postman Collection](./rezStream%20Cloud%20API.postman_collection.json), authentication can be used after a few items are configured:

- Set the `rezcloudapi_clientid` collection or environment variable to your assigned client ID.
- Set the `rezcloudapi_clientsecret` collection or environment variable to your client secret.
- Specify appropriate space delimited scopes within the Auth section of the collection.

### Manual Postman Configuration

Postman makes testing the integration very easy. Within the **Authorization** section of a request or collection, select `OAuth 2.0` as the **Type**. Finally, as the meme goes, *“draw the rest of the owl”* by configuring the remainder of the OAuth section with the values shown below. Substitute your desired values for `Auth URL` and `Access Token URL` from the well-known metadata document, the values for `Client ID` and `Client Secret` from what we give you, and then season the remaining inputs to taste. Reach out to us if you need any additional integration assistance. Our OAuth endpoint also supports additional custom arguments which can be supplied in the "Auth Request" section of the postman OAuth tool.

![image](https://github.com/rezstream/rezStream-Cloud-API-Documentation/assets/1365831/d7ab2569-2bda-4175-95e9-7cabe104edd1)

After the OAuth 2 section is configured, clicking **Get New Access Token** will open a web browser to start the authorization code flow. After authenticating, you can approve the integration for a specific tenant. Granting access will trigger the remaining parts of the flow and provide Postman with an access token and if requested, a refresh token. The access token can be used by associated or child Postman requests to authenticate against our rezStream Cloud API.

## rezStream Cloud API

After obtaining an access token with sufficient permissions, queries to the rezStream Cloud API will be permitted. So far, all endpoints only support querying data from the system. In the future we will support the submission of commands and data mutations. We provide Open API documents for our API which can be used to get an overview of the offered endpoints: https://cloudapi.rezstream.com/openapi. The Open API documents aren’t the most human readable formats, but it outlines the supported endpoints and requests.

### Additional resources:

- [Postman Collection](./rezStream%20Cloud%20API.postman_collection.json)
- [Open API](https://cloudapi.rezstream.com/openapi)
- [OpenID Configuration](https://account.rezstream.com/.well-known/openid-configuration)

### Request Headers

* Authorization: Most request sent to the API **require** A valid "Bearer" token in the `Authorization` header.
* X-RezStream-Api-Version: This optional header can be used to override the default API version. See the Versioning section for more details.

### Versioning

We use a date style of versioning in the format of `YYYY-MM-DD` which can be use to override the default version of a request. The known list of API versions should be available within our [Open API](https://cloudapi.rezstream.com/openapi) documents. Version overrides can be added to requests through the `X-RezStream-Api-Version` HTTP header.

When we make incremental and “safe” changes to response models and API endpoints we may allow those changes to apply to older API versions. However, changes considered breaking should be isolated from older versions, where reasonable. This is a balance between providing a stable yet easy to use and maintainable API for integrations.

### Permissions

Each access token has permissions associated with it. A token that only has access to read reservations or invoices will fail with an HTTP 403 response when used to make modifications. These permissions are set upon authorizing an integration, so consider the requested scopes carefully when establishing an integration.

### Pagination

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

## The Data and Endpoints

This offers a general overview of our API endpoints. For specific schema details, use the [Open API](https://cloudapi.rezstream.com/openapi) json or yaml data.

### Inventory

There are multiple special purpose inventory endpoints available, but generally the `/inventory` endpoint. This single endpoint returns the following information in a single response:

- Business details including time zone and location
- Properties associated with a tenant business
- Units with details such as capacity
- Unit Types with details such as associated units and associated rates
- Guest types
- Rates

In addition to the broad `/inventory` endpoints there are more specialized endpoints which may be more appropriate for your needs.

- `/inventory/business`
- `/inventory/properties`
- `/inventory/units`
- `/inventory/unitTypes`
- `/inventory/rates`

### Availability

Current availability status or metrics can be queried by date range for units or unit types.

#### Unit Availability Status

Querying the unit availability calendar endpoint at `/availability/calendar/unit?from=2025-09-23&to=2025-09-27` results in the status of each unit for the given dates. See the Open API schema for details, but a result should look similar to the following:

```json
[
  //...
  {
    "unitId": "62b47eec-f586-4b42-8951-f1ec4ea2b497",
    "startDate": "2025-09-23",
    "endDate": "2025-09-27",
    "statuses": ["Available", "Reserved", "Blocked", "Available", "Available"]
  }
  // ...
]
```

#### Unit Type Availability Metrics

Querying the unit type availability calendar endpoint at `/availability/calendar/type?from=2025-09-23&to=2025-09-27` results in the availability metrics for each unit type on the given dates. See the Open API schema for details, but a result should look similar to the following:

```json
[
  //...
  {
    "unitTypeId": "5b49d220-b0bd-4317-b5de-fbed92f6193a",
    "startDate": "2025-09-23",
    "endDate": "2025-09-27",
    "unitCount": 2,
    "metrics": [
      { "available": 1, "reserved": 1, "blocked": 0 },
      { "available": 0, "reserved": 2, "blocked": 0 },
      { "available": 1, "reserved": 0, "blocked": 1 },
      { "available": 2, "reserved": 0, "blocked": 0 },
      { "available": 2, "reserved": 0, "blocked": 0 }
    ]
  }
  // ...
]
```

#### Availability Blocks

Items blocking availability which are not reservations can be queried from the `/availability/blocks/byDate?from=2025-09-23&to=2025-09-27` endpoint. For response and query details, see the Open API schema.

### Rates

Rate and pricing information can be queried via some endpoints.

> [!IMPORTANT]
> Please note that a "Rate Plan" within our system often does not correspond to a "Rate Plan" in other systems. Usually a "Rate" in our system which is a child of our "Rate Plan" is what maps to a "Rate Plan" in other systems. Integrating partners should typically focus on a our globally unique Rate ID. 

#### Nightly Rates (GET)

To retrieve the nightly configuration for rates within a given date range, use the `/rates/nightly?from=from=2025-09-23&to=2025-10-03` endpoint. While it isn't a true representation of how rates are work in our system, the format returned should meet the needs of most rate management systems.

```json
[
  //...
  {
    "rateId": "5e055c34-1371-453e-878b-82b81d131aaf",
    "rateName": "Rack",
    "ratePlanId": "5b49d220-b0bd-4317-b5de-fbed92f6193a",
    "ratePlanName": "Standard",
    "defaultRate": true,
    "seasonType": "Daily",
    "startDate": "2025-09-23",
    "endDate": "2025-10-03",
    "allowUpdates": true,
    "data": [
      {
        "price": 150, "baseOccupancy": 1,
        "extraGuestCharges": {
          "0e2e66ab-e626-40be-ad40-bb07ad2c29aa": 15.0000,
          "66e8b60d-2c33-419f-b23a-f3faf507ddbb": 5.0000
        }
      },
      {
        "minimumNights": 7,
        "price": 176.29, "baseOccupancy": 2
      },
      // ...
      { "price": 120, "baseOccupancy": 2 },
      { "price": 120, "baseOccupancy": 2 }
    ]
  },
  // ...
]
```

#### Nightly Rates (POST)

Rates can be updated on a nightly basis by sending a POST request to the `/rates/nightly` endpoint.

> [!IMPORTANT]
> Only daily rates are currently supported. We may support nightly updates for long term or extended rates in the future. See the `allowUpdates` facet in the results of the accompanying GET request to best know if a rate may be updated using this API.

Request:

```json
[
  //...
  {
    "rateId": "5e055c34-1371-453e-878b-82b81d131aaf",
    "data": [
        { "date": "2026-01-14", "price": 190.00, "closedToArrival": false },
        { "date": "2026-01-15", "price": 190.00 },
        { "date": "2026-01-16", "price": 215.00, "minimumNights": 2, "closedToDeparture": false },
        { "date": "2026-01-17", "price": 215.00, "minimumNights": 2, "closedToDeparture": true, "closedToArrival": true }
    ]
  },
  //...
]
```

Response: 200 OK when successfully applied.

Any non-null facets supplied per date will apply an update to the data. The available facets are as follows:

- price: The nightly price to charge for the configured base pricing occupancy.
- minimumNights: The minimum nights stay required to permit booking for the associated date.
- baseOccupancy: The occupancy count up to which base pricing will apply before extra guest charges are added to an invoice.
- closedToArrival: Indicates if the rate may or may not be used for reservations which arrive on the associated date.
- closedToDeparture: Indicates if the rate may or may not be used for reservations which depart on the associated date.

#### Reserved Rates

To get a list of reservations with pricing information, the endpoint `/rates/reserved/byDate?from=2025-09-23&to=2025-09-27` can be used to return past and future pricing information without any PII exposure. This data can be useful for giving rate management systems insight into actual quoted prices. Sample result:

```json
[
  //...
  {
    "id": "8a811a72-8392-41dd-b58a-5946ee798eee",
    "version": 10,
    "status": "Reserved",
    "madeAt": "2025-09-24T02:07:50.7507775Z",
    "updatedAt": "2025-09-24T02:24:46.043Z",
    "arrival": "2025-09-23",
    "departure": "2025-09-26",
    "ingestionSource": "PMS",
    "unitAssignments": [
      {
        "id": "06839b08-235d-3ab7-201f-39f8a0abc135",
        "arrival": "2025-09-23",
        "departure": "2025-09-26",
        "nights": 3,
        "unitId": "9c0bd951-945a-4504-b03a-c6189e03af3d",
        "unitTypeId": "5b49d220-b0bd-4317-b5de-fbed92f6193a",
        "propertyId": "6bdd5ab5-c22b-4713-8a73-d5c41b3d7d3e",
        "rateId": "5e055c34-1371-453e-878b-82b81d131aaf",
        "ratePlanId": "5b49d220-b0bd-4317-b5de-fbed92f6193a",
        "occupancyCount": 2,
        "rentalRevenue": 432.0,
        "rentalTax": 46.65,
        "rentalFee": 17.28
      }
    ]
  }
  // ...
]
```

### Reservations

We currently have two endpoints that can be used to find reservations in our system "by stay dates" or "by last update timestamp".

#### By Last Update

The list of reservations by update timestamp can be queried from a starting date and uses forward only pagination due to the shifting nature of the data. This can be a handy endpoint to use when polling for new or modified reservations. The simplest request can start with `/reservations/updated?from=2023-01-01T00:00:00Z&limit=20` and will provide the reservations in ascending update order. In the future, web-hooks should make this endpoint redundant. The design of the cursor pagination for this endpoint supports updates made concurrently during a query making this a reliable way to detect new changes when polling.

#### By Stay Date

Another method of locating reservations is to use the stay based window query, searching by date. This can be useful when you need perform an initial population of data or quickly ensure you have the latest data for a window of dates. Note that the input date values are in the local time zone of the tenant. The simplest query for this endpoint is `/reservations/byDate?from=2023-01-01&to=2024-01-01&limit=20` and pagination can provide more results if they exist.

#### Reservation Details

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

