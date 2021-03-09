# [DID Connect](spec/DIDConnect.md)

This document describes a method to authenticate a user against their [Decentralized Identifier (DID)](https://www.w3.org/TR/did-core/). This method builds upon [OpenID Connect (OIDC)](https://openid.net/developers/specs/), which itself builds upon the [OAuth 2.0 framework](https://tools.ietf.org/html/rfc6749).

The objective and main challenge of DID Connect is to extend OIDC in a way that makes it as resource-owner-centric as possible, to enable Self-Sovereign Identity (SSI) scenarios.
