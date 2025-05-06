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
  GitHub:
    title: About security hardening with OpenID Connect
    target: https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect
    date: 2025


--- abstract

In workload-to-service communication, a common pattern is for a workload to present a JSON Web Token
(JWT) to an authorization server in order to obtain an access token for the service it needs to access. It is a partial adaptation for workloads of existing flows designed for users. Implementing this pattern combines multiple existing standards from different working groups and standards bodies. Since this pattern is not
described in a specification, it leads to variability in interoperability. The purpose
of this document is to capture this common workload identity practice as an RFC in order to obtain consistency and promote interoperability in industry.

--- middle

# Introduction

In workload-to-service communication, a common pattern is for a workloads to use a JSON Web Token
(JWT) to identify and authenticate itself as part of a process to obtain an access token for a service. This is done by having the workload present an asynchronously-provisioned bearer token in the form of a
signed JWT to an Authorization Server. The Authorization Server verifies the JWT and then
provides an OAuth access token as described in [RFC7523]. The "bootstrap" problem of
discovering the original JWT issuer is solved by requesting a JSON configuration
document using the process described in OpenID Connect Discovery
{{OIDC.Discovery}} or OAuth 2.0 Authorization Server Metadata [RFC8414].

Since this pattern is not described in a specification, it leads to
variability in interoperability. The purpose of this document is to capture
this common workload identity practice as an RFC in
order to obtain consistency and promote interoperability in industry.

# Architecture and Message Flow {#architecture-and-message-flow}


{{fig-message-flow}} illustrates the OIDC-based message flow described in {{jwt-authentication}}:

~~~ aasvg
      4) Verify signature
         using JWK

     +----------------+ 3) Retrieve JWKs
     |                |    from "jwks_uri"
     | Authorization  +<----------------.
     |    Server      |                  |
     |                |                  |
     +----+-------+---+                  |
          ^       |                      |
2) JWT    |       | 5) Provide           |
   Bearer |       |    Access            v
   Token  |       |    Token     +-------+------+
          |       |              |              |
          |       |              |  JWT Issuer  |
          |       v              |              |
      +---+-------+-----+        +-------+------+
      |                 |                |
      |    Workload     |<--------------'
      |                 |  1) Initial provisioning
      +--------+--------+
               |
               |  6) Authenticate with
               |     Access Token
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
   JWT Bearer Token as specified in Section 4 of [RFC7523].
2. The Authorization Server takes the value from the `iss` claim and appends
   `/.well-known/openid-configuration` to retrieve the JWT Issuer's
   configuration via HTTP, as specified in [OIDC.Discovery]. Alternatively, the
   OAuth 2.0 Authorization Server Metadata endpoint [RFC8414] may be used.
3. The Authorization Server then retrieves the JWKs via HTTP from the `jwks_uri`
   declared in the JWT Issuer's configuration response.
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

In order for the workload to access the resource, the following requires prior configuration:

1. The JWT Issuer must be recognized by the Authorization Server,
2. Claims in the JWT must be validated and "exchanged" for an identity legible
   to the Resource Server,
3. and the resulting Resource Server identity must be authorized to access the
   Resource.

Step \#1 requires an explicit trust relationship between the Authorization
Server and the JWT Issuer. Despite previous attempts to standardize dynamic
client registration in [RFC7591] and [OIDC.Dynamic], in practice this trust
relationship, still depends on vendor-specific configuration.

As an example of current practice for configuration of Steps \#2 and \#3, see
[GitHub].

# Security Considerations {#security-considerations}

This document illustrates a common pattern in trust domain federation. However,
the "identity exchange" Step \#2 in {{interoperability-considerations}} is not
standardized. In practice, the workload platform and the Resource Server
platform define principals differently, and the translation mechanism between
the two identities is implemented differently by each Resource Server platform.
This lack of standardization is not merely inconvenient; it is a rich source of
privilege escalation attacks due to misconfigurations.

Therefore, the following recommendations apply to configurations that control
the "identity exchange" step:

1. The configuration SHOULD use specific JWT claims to prevent any JWT signed by
   the JWT Issuer from being used to impersonate any Resource Server identity.
2. The configuration that controls the translation of the workload JWT to a
   Resource Server identity SHOULD NOT rely on claims that can be controlled by an
   attacker.

The security considerations in section 8 of [RFC7521] generally apply. As bearer
tokens, stolen JWTs are particularly valuable to attackers:

1. A secure channel (e.g. TLS) SHOULD be used when providing a JWT for
   authentication.
2. JWTs SHOULD be protected from unauthorized access using operating system or
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
