# Introduction

Sometimes an AS needs to register itself to an existing DID.

This document describes a way to do this using the DIDConnect method with a new scope.

# Detail

In this scenario:
  - The Client is the application that controls a digital identity.
  - The Candidate Authorization Server (Candidate-AS) is the application that wants to gain control to the digital identity.

Process:
1. Client presents a DIDConnect link with a `register` scope to the Candidate-AS.
   - As a hint, the link may include a `userid` identifier that will allow the Candidate-AS to determine if it already controls that identity.
2. Candidate-AS creates a JWT as it normally would for an already controlled DID, but using its own public key identifier (e.g. Ethereum address) as `sub`.
3. Candidate-AS sends the JWT to the Client's redirect URI.
4. Client authorizes the AS and returns the DID to the Candidate-AS (now a proper AS for that DID).


# Steps

## 1. Client presents link

Example link format (without newlines and spaces):
```
didconnect://auth
  ?client_id=xxx
  &redirect_uri=https://lacchain/api/v1/wallet/authorize-device
  &state=0xbf80f0de5a54190e165392410778663f828fdbb00a281f7266fa39582e42be98
  &userid=1234567890
  &scope=register
```

New query parameters meaning:
| Parameter | Description 
|-----------|-----------------------
| scope     | Must be `register`.
| userid    | Identifier for the digital identity, so that the Candidate-AS can check if it already controls the identity.

## 2. Candidate-AS creates JWT

New JWT claims:

| Claim | Description 
|-------|-----------------------
| sub   | Identifier of the Candidate-AS's public key. Client must be able to verify that the identifier matches the device's signature.

## 3. Candidate-AS sends the JWT

Use the `redirect_uri` after verifying it normally (see DIDConnect spec).


## 4. Client authorizes Candidate-AS's key

Client verifies the key ID against the signature.
