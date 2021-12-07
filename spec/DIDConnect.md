# OpenID Connect Profile for self-sovereign identity (DIDConnect)

## Copyright information

Copyright (c) 2021 everis and its authors.

This document is made available under the terms of the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license, meaning you are free to share it and build upon it as long as you give proper attribution.

## Introduction

This document describes a method, called "DID Connect" or "DIDC", to authenticate a user with their [Decentralized Identifier (DID)](https://www.w3.org/TR/did-core/). This method builds upon [OpenID Connect (OIDC)](https://openid.net/developers/specs/), so familiarity with OIDC is recommended.

The objective of DID Connect is to extend OIDC in a way that makes it resource-owner-centric, to enable Self-Sovereign Identity (SSI) scenarios. That decentralization makes DID Connect different from traditional provider-centric OIDC flows in a few aspects, as detailed below.

| Aspect                    | OpenID Connect            | DID Connect
| ------------------------- | ------------------------- | -----------------------
| Control                   | Provider-centric          | User-centric
| Subject ID (`sub`)        | IdP-assigned.             | User's DID.
| `scope` in requests       | `scope=openid`            | `scope=did`
| Client registration flow  | Client registers against IdP (must be authorized by IdP).           | No authorization needed by a central entity.
| Client ID                 | IdP-assigned. | URL of a Verifiable Presentation controlled by the client.
| Authorization endpoint    | Contains the Identity Provider (IdP)'s hostname. Either static or discovered through OIDC Discovery. | Custom scheme (`didconnect:`).
| Userinfo endpoint           | Managed by IdP. Either static or discovered through OIDC Discovery.   | Contained in `id_token` (`userinfo` claim).
| Client information        | Held and shown by IdP.       | Verifiable Presentation, containing Verifiable Credentials issued by any relevant authorities.
| Public key of token's issuer | Either static or discovered through OIDC Discovery. | Listed in DID Document (DDO).
| Client's whitelisted redirect_uri endpoints | Registered and checked by the IdP. | Announced as a claim in the Client ID's Verifiable Presentation.
| Supported flows | Either static or discovered through OIDC Discovery. | Negotiated during authentication (**WIP**).

## General flow

The OIDC specification defines the following general flow:

1. The Relying Party (RP) registers (once) to get a client_id.
2. RP presents an authentication request in the form of a URL pointing to the Authorization Server (AS).
3. Resource Owner authenticates on the AS if needed and provides consent to share the requested information with that RP.
4. Depending on the value of `response_type`, AS generates a combination of identity token, grant code, and/or access token, then redirects the user agent to the `redirect_uri` contained in the request, honouring the `response_mode` parameter (form, post or fragment).
5. RP receives the tokens and/or code.
6. *Authorization flow only.* RP gets the URL of the token endpoint and uses it to request the `id_token` and/or `access_token` by providing the grant code.
7. RP checks the signature of `id_token` as a JWT and reads user's identity from the token.
8. If a `userinfo` claim is available in `id_token` and an `access_token` was provided, client queries that endpoint, using the access token for authentication.

Specificities on how those steps are to be applied for DIDConnect are detailed in the following sections.

### 1. Client registration and obtaining client_id (once)

1. Relying Party obtains a DID.
2. RP obtains Verifiable Credentials whose subject is the RP's DID, and issued by entities trusted by the targeted users. How users should trust the RP's DID and/or credentials is outside the scope of this specification. The credentials should typically claim information about the RP, such as the application's name, originating organization, logo, website, etc, but the semantics of such claims is also outside the scope of this specification.
3. RP obtains a Verifiable Credential whose subject is the RP's DID, and containing the claim `redirect_uri` with the value of whitelisted callback URL(s). Like the previous point, how the AS should trust that credential is not defined in this document. However, it is recommended that the credential is self-issued, i.e. the issuer is also the RP's DID.
4. RP creates a Verifiable Presentation containing those credentials, and whose holder is the RP's DID.
5. The Verifiable Presentation is available at a location where users will be able to access it. The RP's Client ID is the presentation's URL.

### 2. Authentication request

A DIDC request must include a `scope` value of `did` (as opposed to a value of `openid` for OIDC).

The value of `client_id` is the one obtained during registration (see above).

### 3. Obtaining user consent from AS

When a user's Authorization Server receives a request from the application's client_id, it must treat `client_id` as an URL and fetch the Verifiable Presentation (VP) from it. Then, the AS must check the following conditions:

- The VP's proof is valid.
- The VP is within validity date.
- The `redirect_uri` value from the authentication request matches one of the values present in the VP's _valid credentials_.

Then, the AS should ask for user's consent after displaying all relevant claims, based on _valid credentials_.

Note: For `redirect_uri` and displayed claims, **_valid credentials_** are credentials with a valid proof, that are within validity dates, and whose subject ID is the same as the VP's holder.

### 4. Token generation

Unlike OpenID Connect, the token must be signed with a key that is listed in the DID document of the subject's DID. This is what makes DID Connect user-centric.

### 5. Getting the tokens and/or authorization code

See OpenID Connect specification. A Relying Party in DID Connect is always considered a public client, and as such should rely on PKCE when the authorization flow is used.

### 6. Accessing the tokens (if applicable)

See OpenID Connect specification. If the authorization flow is used, the token endpoint URI is taken from the `token_uri` parameter contained in the response.

### 7. Verification of user's identity

The RP's endpoint listening on `redirect_uri` receives the same data as defined in OIDC specification.

The JWT contained in the `id_token` is the same as defined in the OIDC specification, with the following differences:

| Parameter | Meaning in DID Connect
|-----------|-----------------------
| `sub`     | The user's DID.
| `iss`     | The identifier (resolved from the DID Document) of the public key that is signing the JWT. The key must be valid for that DID and the JWT must be signed by that key.
| `userinfo` | The URL of the Resource Server where a user's Verifiable Presentation is available.

The Relying Party must make sure that:

- The key identified in `iss` is an authorized key of the DID value in `sub`, by looking at the DID Document.
- The JWT is signed by that key.

### 8. Accessing the user's protected resources (when applicable)

If an access token was provided, the client can access user's information by querying the URL referenced as `userinfo` in the id_token, and provide the access token as an `Authorization` header, as described in [OpenID Connect specification](https://openid.net/specs/openid-connect-core-1_0.html#UserInfoRequest).

The response is a Verifiable Presentation. The presentation must contain a valid proof, must be within validity dates, and the presentation's holder must be the user's authenticated DID.

## Decentralized URI scheme for authorization endpoint

In some cases, the RP wants to let the user authenticate using a native or mobile wallet rather than a web application. This is possible because, unlike OIDC, the tokens are issued by the user rather than a central identity provider.

The decentralized authorization endpoint scheme is `didconnect:`. It works similarly to schemes such as `mailto:` or `tel:`, in that any application on the user agent can register itself on the RO's operating system to handle such requests.

Below are examples of decentralized Authorization Servers.

| AS implementation | Custom scheme registration |
|--|--|
| Mobile or desktop application | Specific to the operating system |
| Web application | Using `registerProtocolHandler()` on the web browser |

## Current limitations

See ["must address" issues](https://github.com/KayTrust/did-connect/labels/must%20address).
