---
title: "WIMSE Headless JWT Authentication"
abbrev: "WIMSE Headless JWT Authentication"
category: info

docname: draft-levy-wimse-headless-jwt-authentication-practices-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Workload Identity in Multi System Environments"
keyword:
 - workload
 - identity
 - credential
 - exchange
venue:
  group: "Workload Identity in Multi System Environments"
  type: ""
  mail: "wimse@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/wimse/"
  github: "heymarcel/draft-ietf-wimse-headless-jwt-authentication"
  latest: "https://heymarcel.github.io/draft-ietf-wimse-headless-jwt-authentication/draft-levy-wimse-headless-jwt-authentication-practices.html"
workgroup: "Workload Identity in Multi System Environments"

author:
 -
    fullname: Marcel Levy
    organization: SPIRL
    email: heymarcel@gmail.com
    role: editor

normative:
  RFC6749: OAuth 2.0
  OIDC.Discovery:
    title: OpenID Connect Discovery 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-discovery-1_0.html
    date: 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: E. Jay
informative:

--- abstract

TODO Abstract

--- middle

# Introduction

In service-to-service communication, a common pattern is to use JWKs
for authentication purposes. This is done by having the workload
(i.e. service) present a bearer token in the form of a signed JWT,
which is then verified by the receiving party. The "bootstrap" problem
of establishing the signing JWK is solved by using an OpenID Connect
Discovery Point {{OIDC.Discovery}}.

Since this pattern is not described in a specification, it leads to
variability in practice. The purpose of this document is to capture
this common workload identity authentication practice as an RFC in
order to obtain consistency and promote interoperability in industry.

# Architecture and Message Flow
TODO - ASCII Diagram showing the flow between the client with a JWT, the Server receiving it and the OIDC endpoint from which the keys are retrieved.

{{fig-message-flow}} illustrates the message flow described in {{JWT.authentication}}:

~~~ aasvg
                            2) GET /.well-known/openid-configuration
                     +-----------------------------------------+
                     |                                         |
                     |                                +--------v----------------+
      +--------------v--+                             |                         |
      |                 |                             |      JWT Issuer/        |
      |    Resource     <----------------------------->  Authorization Server   |
      |     Server      |   3) Retrieve JWKs          |                         |
      |                 |      from "jwks_uri"        +-------------------------+
      +---^-------+-----+
          |       |       4) Verify signature
          |       |          using JWK
          |       |
1) JWT    |       |
   Bearer |       |  5) Response after
   Token  |       |     authentication
          |       |
          |       |
      +---+-------v-----+
      |                 |
      |    Workload     |
      |                 |
      +-----------------+
~~~
{: #fig-message-flow title="OIDC message flow when used in a headless environment"}




# Conventions and Definitions

{::boilerplate bcp14-tagged}

# JWT used for Authentication over HTTP {#JWT.authentication}
TODO describes how the JWT is used over HTTP (protocol binding) - maybe just refer to RFC 7523. Leave door open for other protocol bindings that can be defined elsewhere




# Key Discovery
TODO describes the key discovery mechanism - refer to OIDC discovery mechanisms.

# JWT Format and Processing Requriements

## JWT Format
TODO - describe claims and format of JWT needed.

## JWT Processsing
TODO - how should the client and server process the JWT (verification etc)

## JWT Provisioning
TODO - describe where the JWT may come from. Who issues it etc (could also be a security consideration)

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
