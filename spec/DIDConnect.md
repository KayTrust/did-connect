# OpenID Connect Profile for self-sovereign identity (DIDConnect)

## Copyright information

Copyright (c) 2021 NTT DATA and its authors.

This document is made available under the terms of the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license, meaning you are free to share it and build upon it as long as you give proper attribution.

## Motivation

[OpenID Connect (OIDC)](https://openid.net/developers/specs/) does a great job at letting users authenticate thanks to the role of an Identity Provider trusted by the Relying Party. However, this scheme has led to the apparition of near-monopolistic providers, as there is a conflict of interests for the Internet community between keeping things decentralized and developers not having to trust a large number of Identity Providers.

The recent developments around Decentralized Identifiers introduce an opportunity to solve that conflict of interests. It lets each user act as their own Identity Provider, by signing the identity token with the key of their choice after declaring that provider's public key in the DID Document.

In addition, that decentralization allows for native or mobile implementations of Identity Providers, so there should be a way to make an identity request without assuming the actual location of the Identity Provider – just like "mailto:" URIs let users free to pick whatever SMTP server they want to send an email, as opposed to what would be presenting a full HTTP endpoint.

## Summary

This document describes a method, called "DID Connect" or "DIDC", to authenticate a user with their [Decentralized Identifier (DID)](https://www.w3.org/TR/did-core/). This method builds upon [OpenID Connect (OIDC)](https://openid.net/developers/specs/), so familiarity with OIDC is recommended.

The objective of DID Connect is to extend OIDC in a way that makes it resource-owner-centric, to enable Self-Sovereign Identity (SSI) scenarios. That decentralization makes DID Connect different from traditional provider-centric OIDC flows in a few aspects, as detailed below.

| Aspect                    | OpenID Connect            | DID Connect
| ------------------------- | ------------------------- | -----------------------
| Control                   | Provider-centric          | User-centric
| Subject ID (`sub`)        | IdP-assigned (locally unique)             | User's DID (globally unique)
| `scope` in requests       | `scope=openid`            | `scope=did`
| Client registration flow  | Client registers against IdP (must be authorized by IdP).           | No authorization needed by a central entity.
| Client ID                 | IdP-assigned. | URL of a Verifiable Presentation controlled by the client.
| Authorization endpoint    | Contains the Identity Provider (IdP)'s hostname. Either static or discovered through OIDC Discovery. | Optional use of custom scheme (`didconnect:`).
| Userinfo endpoint           | Managed by IdP. Either static or discovered through OIDC Discovery.   | Contained in `id_token` (`userinfo` claim).
| Client information        | Held and shown by IdP.       | Verifiable Presentation, containing Verifiable Credentials issued by any relevant authorities.
| Public key of token's issuer | Either static or discovered through OIDC Discovery. | Listed in DID Document (DDO).
| Client's whitelisted redirect_uri endpoints | Registered and checked by the IdP. | Announced as a claim in the Client ID's Verifiable Presentation.
| Supported flows | Either static or discovered through OIDC Discovery. | Negotiated during authentication (**WIP**).

## Concepts

A few concepts in DID Connect differ from the OIDC ones.

### Client registration and client ID

Client registration in DID Connect is as decentralized as publishing any content on the Internet. The Relying Party issues a Verifiable Presentation containing any Verifiable Credentials that it wishes to present to the user's Identity Provider.

The Verifiable Presentation MUST contain a valid proof and MUST be within validity dates at the time of the request. Its holder is considered the Relying Party's DID.

The included Verifiable Credentials will typically state the name, logo etc. of the RP, will have the RP's DID as subject identifier, and will come from issuers that the user's Identity Provider trusts. They MUST contain a valid proof and MUST be within validity dates.

After publishing that VP using any URI scheme, such as `https` or `data`, the Relying Party's client ID is the URI where the VP is available.

### Relying Party's redirection URI registration

To register one or several redirection URIs, the Relying Party's client_id VP must include Verifiable Credentials with `response_uri` claims whose values are the accepted redirection URIs.

### Request scope

A DID Connect request MUST include a scope of `did`, unlike OpenID Connect which uses a scope of `openid`.

### ID Token

The subject identifier and issuer of a DIDC ID Token differ from those of an OIDC ID Token.

The subject identifier (`sub` claim) is the user's DID, which is a *globally* unique identifier.
The issuer identifier (`iss` claim) is the identifier of the public key that is signing the JWT. That key MUST be listed as a valid signing key in the subject's DID Document.

### Supported flows

Since there is no private registration, a DIDC Relying Party is always considered a public client, and as such should rely on PKCE when the authorization flow is used.

### Token and userinfo endpoints

If the authorization flow is used, the token and userinfo endpoints are taken from the `token_endpoint` and `userinfo_endpoint` parameters contained in the authorization response.

### Userinfo response

A DID Connect userinfo response is a Verifiable Presentation. The presentation must contain a valid proof, must be within validity dates, and the presentation's holder must be the user's authenticated DID. All Verifiable Credentials about the user MUST use the user's authenticated DID as subject identifier.

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
