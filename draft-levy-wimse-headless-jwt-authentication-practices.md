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
    fullname: "Marcel Levy"
    organization: SPIRL
    email: heymarcel@gmail.com
 -
    fullname: "Hirsch Singhal"
    organization: GitHub
    email: hpsin@github.com

normative:
  RFC5785: Defining Well-Known Uniform Resource Identifiers (URIs)
  RFC6749: OAuth 2.0
  RFC7521: Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants
  RFC7523: JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants
  RFC7591: OAuth 2.0 Dynamic Client Registration Protocol
  RFC8414: OAuth 2.0 Authorization Server Metadata
  OIDC.Core:
   title: OpenID Connect Core 1.0 incorporating errata set 2
   target: https://openid.net/specs/openid-connect-core-1_0.html
   date: 2023
   author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore
  OIDC.Discovery:
    title: OpenID Connect Discovery 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-discovery-1_0.html
    date: 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: E. Jay
  OIDC.Dynamic:
    title: OpenID Connect Dynamic Client Registration 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-registration-1_0.html
    date: 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones

informative:
  I-D.ietf-wimse-s2s-protocol:

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
service) present an asynchronously-provisioned bearer token in the form of a
signed JWT, which is then verified by an Authorization Server, which then
provides an access token as per [RFC6749]. The "bootstrap" problem of
discovering the original JWT issuer is solved by requesting a JSON configuration
document using the process described in OpenID Connect Discovery
{{OIDC.Discovery}} or OAuth 2.0 Authorization Server Metadata [RFC8414].

Since this pattern is not described in a specification, it leads to
variability in practice. The purpose of this document is to capture
this common workload identity authentication practice as an RFC in
order to obtain consistency and promote interoperability in industry.

# Architecture and Message Flow {#architecture-and-message-flow}


{{fig-message-flow}} illustrates the OIDC-based message flow described in {{jwt-authentication}}:

~~~ aasvg
      5) Verify signature
         using JWK
     +----------------+ 3) GET /.well-known/openid-configuration
     |                |<----------------------+
     | Authorization  |<-------------------+  |
     |    Server      | 4) Retrieve JWKs   |  |
     |                |    from "jwks_uri" |  |
     +------------+---+                    |  |
          ^       |                        |  |
2) JWT    |       | 6) Provide             |  |
   Bearer |       |    Access              v  v
   Token  |       |    Token     +---------------+
          |       |              |               |
          |       |              |  JWT Issuer   |
          |       v              |               |
      +---+-------------+        +-------+-------+
      |                 |                |
      |    Workload     |<---------------+
      |                 |    1) Initial provisioning
      +--------+--------+
               |
               |  7) Authenticate
               |     with token
               v
         +------------+
         |            |
         |  Resource  |
         |   Server   |
         |            |
         +------------+
~~~
{: #fig-message-flow title="OIDC message flow when used in a headless environment"}

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# JWT used for Authentication {#jwt-authentication}



The overall message flow is seen in {{fig-message-flow}}, and this section explains
it in more detail. It assumes the workload has previously acquired a JWT
adhering to the profile specified in [RFC7523]. JWT provisioning assumptions are
described in more detail in {{jwt-provisioning}}.


1. The workload calls an Authorization Server's token endpoint and presents a
   JWT Bearer Token as specified in Section 4 of [RFC7521].
2. The Authorization Server takes the value from the `iss` claim and appends
   `/.well-known/openid-configuration` to retrieve the JWT issuer's
   configuration via HTTP, as specified in [OIDC.Discovery]. Alternatively, the
   OAuth 2.0 Authorization Server Metadata endpoint [RFC8414] may be used.
3. The Authorization Server then retrieves the JWKs via HTTP from the `jwks_uri`
   declared in the JWT issuer's configuration response.
4. Using the appropiate issuer key, the Authorization Server verifies the signature
   of the JWT Bearer Token.
5. The Authorization Server then responds to the workload with an Access Token
   suitable for use with the Resource Server.
6. The Workload then authenticates with the Resource Server using the Access Token.

As we can see, the headless JWT authentication pattern closely follows that of
OIDC, but without the initial authentication by a user.

This document limits discussion to HTTP, as this is the protocol predominantly
used. Although other protocols are out of scope, this should not be read as a
limit on their future use.

# Key Discovery {#key-discovery}

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
a JSON document containing configuration information. An example is provided
here:

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

Note that this key discovery mechanism does not address the question of whether
the key itself should be trusted. This is a registration problem, and is
discussed further in {{interoperability-considerations}}.

# JWT Format and Processing Requirements {#jwt-format-and-processing}

## JWT Format {#jwt-format}

An example JWT adhering to [RFC7523] is seen below. Although this example uses a
WIMSE workload identifier ({{I-D.ietf-wimse-s2s-protocol}}) in the subject
("sub") claim, this is not a requirement in practice.

~~~ json
{
  "iss": "https://issuer.example.org",
  "sub": "spiffe://example.org/ns/default/sa/backend-job-runner",
  "aud": "https://auth.example.com/token",
  "jti": "jwt-grant-id-x1y2z3a4",
  "exp": 1744845092,
  "iat": 1744841036
}
~~~
{: title="Example RFC7523 JWT"}

## JWT Processsing {#jwt-processing}

### Authorization Server Processing {#authorization-server-processing}

The authorization server validates the JWT according to Section 3 in [RFC7523],
with the following exceptions:

1. The "sub" (subject) claim does not identify either a resource owner or an
   anonymous user.
2. The "sub" claim need not correspond to the "client id" of an OAuth client.

The authorization server validates the signature using the key discovered by the
process described in {{key-discovery}}.

### Workload Processing {#workload-processing}

The workload is considered the client in this interaction. It can treat the JWT
acquired during provisioning as an opaque token. It must handle any error
reponse from the authorization server as per Section 3.2 in [RFC7523].

## JWT Provisioning {#jwt-provisioning}

The workload is provisioned with a JWT from a trusted source. This can be the
underlying platform where the workload runs, or a separate issuing system.
Regardless of the actual mechanism, JWT provisioning relies on a registration
mechanism that establishes mutually-trusted, secure connections between the
workload and the JWT provisioner.

This provisioning mechanism illustrates a key difference from flows defined in
[RFC6749] and [OIDC.Core], in that there are no client credentials involved in
the interaction with the Authorization Server.

# Interoperability Considerations {#interoperability-considerations}

TODO Speak to the issue of registration/configuration, because this seems to be
getting to the heart of the problem. The Authorization Server has a mechanism to
get the issuer key, but how do we establish trust? Right now it seems to always
be a manual process. [RFC7591] and [OIDC.Dynamic] are popular things to say, but
they don't seem to be what people implement.

# Security Considerations {#security-considerations}

The security considerations in section 8 of [RFC7521] generally apply. As bearer
tokens, stolen JWTs are particularly valuable to attackers:

1. A secure channel (e.g. TLS) MUST be used when providing a JWT for
   authentication.
2. JWTs MUST be protected from unauthorized access using operating system or
   platform access controls.
3. JWT validity SHOULD be set to the shortest possible duration allowable by
   overall system availability constraints.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Pieter Kasselman for his guidance in structuring
this document.
