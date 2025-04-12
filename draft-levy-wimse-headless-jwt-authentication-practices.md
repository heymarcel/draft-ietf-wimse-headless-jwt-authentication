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

author:
 -
    fullname: Marcel Levy
    organization: SPIRL
    email: heymarcel@gmail.com
    role: editor

normative:
  RFC5785: Defining Well-Known Uniform Resource Identifiers (URIs)
  RFC6749: OAuth 2.0
  RFC7521: Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants
  RFC7523: JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants
  RFC8414: OAuth 2.0 Authorization Server Metadata
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

In service-to-service communication, a common pattern is to use a JSON Web Token
(JWT) for authentication purposes. It is a partial adaptation for workloads of
existing authorization flows designed for users. Since this pattern is not
described in a specification, it leads to variability in practice. The purpose
of this document is to capture this common workload identity authentication
practice as an RFC in order to obtain consistency and promote interoperability
in industry.

--- middle

# Introduction

In service-to-service communication, a common pattern is to use a JSON Web Token
(JWT) for authentication purposes. This is done by having the workload (i.e.
service) present a bearer token in the form of a signed JWT, which is then
verified by the receiving party. The "bootstrap" problem of establishing the
signing JWK is solved by requesting a JSON configuration document using the
process described in OpenID Connect Discovery {{OIDC.Discovery}} or OAuth 2.0
Authorization Server Metadata [RFC8414].

Since this pattern is not described in a specification, it leads to
variability in practice. The purpose of this document is to capture
this common workload identity authentication practice as an RFC in
order to obtain consistency and promote interoperability in industry.

# Architecture and Message Flow
TODO - ASCII Diagram showing the flow between the client with a JWT, the Server receiving it and the OIDC endpoint from which the keys are retrieved.

{{fig-message-flow}} illustrates the OIDC-based message flow described in {{JWT.authentication}}:

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
   Token  |       |     verification
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

# JWT used for Authentication {#JWT.authentication}

The overall message flow is seen in {{fig-message-flow}}, and this section explains
it in more detail. It assumes the workload has previously acquired a JWT
adhering to the profile specified in [RFC7523]. JWT provisioning assumptions are
described in more detail in {{JWT.provisioning}}.

1. The workload calls a Resource Server over HTTP and presents a JWT Bearer
   Token as specified in Section 4 of [RFC7521].
2. The Resource Server takes the value from the `iss` claim and appends
   `/.well-known/openid-configuration` to retrieve the Authorization Server's
   configuration via HTTP, as specified in [OIDC.Discovery]. Alternatively, the
   OAuth 2.0 Authorization Server Metadata endpoint [RFC8414] may be used.
3. The Resource Server then retrieves the JWKs via HTTP from the `jwks_uri`
   declared in the Authorization Server's configuration response.
4. Using the appropiate issuer key, the Resource Server verifies the signature
   of the JWT Bearer Token.
5. The Resource Server then responds to the workload according to the outcome of
   the signature verification.

As we can see, the headless JWT authentication pattern closely follows that of
OIDC, but like much great literature it starts "in medias res," without the
initial authorization by a user.

This document limits discussion to HTTP, as this is the protocol predominantly
used. Although other protocols are out of scope, this should not be read as a
limit on their future use.

# Key Discovery

Issuer key discovery follows the steps outlined in Section 4 of
[OIDC.Discovery]. The Resource Server makes a request to a location that is
well-known according to [RFC5785]:

~~~ text
GET /.well-known/openid-configuration HTTP/1.1
Host: example.com
~~~
{: title="Example request to issuer to obtain OIDC configuration"}

For OAuth 2.0, the equivalent location is
`/.well-known/oauth-authorization-server`. In both cases, the requester expects
a JSON document containing configuration information. An example is provided here:

~~~ json
{
  "issuer": "https://example.com",
  "jwks_uri": "https://example.com/.well-known/jwks.json",
  "authorization_endpoint": "https://example.com/auth",
  "token_endpoint": "https://example.com/token"
}
~~~
{: title="Example issuer configuration response"}

For the sake of the pattern described in this document, only the `issuer` and
`jwks_uri` fields are relevant.

# JWT Format and Processing Requirements

## JWT Format
TODO - describe claims and format of JWT needed.

## JWT Processsing
TODO - how should the client and server process the JWT (verification etc)

## JWT Provisioning {#JWT.provisioning}

The workload is provisioned with a JWT from a trusted source. This can be the
underlying platform where the workload runs, or a separate issuing system. Regardless of the actual mechanism,
JWT provisioning relies on an enrollment mechanism that establishes
mutually-trusted connections between the workload and the JWT provisioner.

# Security Considerations

The security considerations in section 8 of [RFC7521] generally apply. As bearer
tokens, stolen JWTs are particularly valuable to attackers:

1. A secure channel (e.g. TLS) MUST be used when providing a JWT for authentication.
2. JWTs MUST be protected from unauthorized access using operating system or platform access controls.
3. JWT validity SHOULD be set to the shortest possible duration allowable by overall system availability constraints.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
