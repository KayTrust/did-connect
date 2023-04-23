# [DID Connect](spec/DIDConnect.md)

This document describes a method to authenticate a user against their [Decentralized Identifier (DID)](https://www.w3.org/TR/did-core/). This method builds upon [OpenID Connect (OIDC)](https://openid.net/developers/specs/), which itself builds upon the [OAuth 2.0 framework](https://tools.ietf.org/html/rfc6749).

The objective and main challenge of DID Connect is to extend OIDC in a way that makes it as resource-owner-centric as possible, to enable Self-Sovereign Identity (SSI) scenarios.

# Status

The DID Connect effort started in 2017 with KayTrust (called everisID at the time). We always provided this OIDC-like authentication mechanism based on the simple principle that your wallet is your own Authorization Server, with dynamic signing keys. At the time and until 2022, no similar approach existed. However, SIOP in its version 2 now has converged towards the same principles as DIDConnect, so we've decided to archive DIDConnect and support SIOPv2, backed by a strong community of spec editors and standards experts, rather than trying to write our own timid specification to do the same thing. We will always be proud to have been the precursors in envisioning a decentralized approach to OIDC.
