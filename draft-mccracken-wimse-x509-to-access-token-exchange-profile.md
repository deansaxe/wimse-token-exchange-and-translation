---
title: WIMSE x.509 to Access Token Exchange Profile
abbrev: x.509 to Access Token Profile
category: info

docname: draft-mccracken-wimse-x509-to-access-token-exchange-profile-latest

submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 1
date:
consensus: true
v: 1
area: "Applications and Real-Time"
workgroup: "Workload Identity in Multi System Environments"
keyword:
 - workload identity
 - token exchange
 - token translation
venue:
  group: "Workload Identity in Multi System Environments"
  type: "Working Group"
  mail: "wimse@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/wimse/"
  github: "wimse-token-exchange-and-translation"
  latest: "https://deansaxe.github.io/wimse-token-exchange-and-translation/draft-mccracken-wimse-x509-to-access-token-exchange-profile.html"

author:
 -  fullname: Ken McCracken
    organization: Google
    email: kenmccracken@google.com

normative:

  RFC2119: # Keywords
  RFC6750: # OAuth
  RFC8174: # Ambiguity in Keywords
  RFC8392: # CBOR Web Token (CWT)
  RFC8693: # OAuth 2.0 Token Exchange
  RFC9068: # JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens

  OIDC:
    title: OpenID Connect Core 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-core-1_0.html
    author:
    - name: Nat Sakimura
      org: NRI
    - name: John Bradley
      org: Ping Identity
    - name: Mike Jones
      org: Microsoft
    - name: B. de Medeiros
      org: Google
    - name: Chuck Mortimore
      org: Salesforce
    date: 2014-11


informative:


--- abstract

TODO

--- middle

# Introduction

TODO 

# Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.


# Terminology

TODO: Define terms used by this specification

* authorization server [[RFC6749](https://datatracker.ietf.org/doc/html/rfc6749)]

The server issuing access tokens to the client after successfully authenticating the resource owner and obtaining authorization.

* workload [[draft-ietf-wimse-arch](https://datatracker.ietf.org/doc/html/draft-ietf-wimse-arch)]

A workload is a running instance of software executing for a specific purpose. Workload typically interacts with other parts of a larger system. A workload may exist for a very short durations of time (fraction of a second) and run for a specific purpose such as to provide a response to an API request. Other kinds of workloads may execute for a very long duration, such as months or years. Examples include database services and machine learning training jobs.

* token

An integrity-protected string denoting a lifetime and specific attributes of a security context. In the context of WIMSE, the token denotes attributes of a *workload* security context.

# Overview

TODO

## X.509 Certificate to Access Token Profile

In [[draft-ietf-wimse-arch](https://datatracker.ietf.org/doc/html/draft-ietf-wimse-arch)], Workloads may be issued Identity Credentials in the form of X.509 Certificates [[RFC5280](https://datatracker.ietf.org/doc/html/rfc5280)], for Workload-to-Workload communication over mututal TLS (mTLS). Workload Agents must request the X.509 Certificate Credentials by undergoing Attestation against both the local Host Operating System and Hardware, and a remote Server with access to a Certificate Authority (CA). If the Server confirms sufficient evidence has been presented for Attestation, the Workload is issued X.509 Certificates identifying it. The identity is conveyed in a URI Subject Aleternative Name (SAN) within the X.509 Certificate.

Authorization servers issue OAuth 2.0 Access Tokens to client applications.  If the resource owner has granted sufficient privileges to a protected resource, the issued access token can be used to access protected resources on resource servers.

The Workloads possessing X.509 Certificate Identity Credentials may operate in an environment that is isolated from the security domain of a protected resource. In the case where the protected resource is protected by an external OAuth 2.0 Authorization Server, X.509 Certificate-to-Access Token exchange may be configured. The Trust across the isolated security domains must first be established. Relying parties must describe, via secure configurations, a mapping that crosses the security domains from the X.509 certificate authority(ies) to the OAuth 2.0 authorization server. The specification for authenticating the relying party and for the format of the configurations are out of scope of this specification. The following configurations MAY be registered by a relying party at the OAuth 2.0 authorization server:

1. A set of one or more Trust Anchors MUST be configured for the relying party at the OAuth 2.0 authorization server, representing authoritative entities with a public key and associated data, as defined in [[RFC6024](https://datatracker.ietf.org/doc/html/rfc6024)]. These configurations MUST be represented as X.509 CA certificates in either DER or PEM format.
2. A set of intermediate entities with public key and associated data, expressed as X.509 CA certificates in either DER or PEM format, MAY be configured for the relying party at the OAuth 2.0 authorization server. The intermediate CA certificates are for the purposes of certificate chain path building in scenarios where clients cannot or may not provide these intermediate certificates during mTLS handshakes.
3. A mapping MUST describe the certificate attribute(s) used to select and or construct the subject claim in the OAuth 2.0 access tokens. Possible X.509 certificate properties include the following:
   - subject's common name (CN)
   - first subject alternative names (SAN) DNS Name entry
   - first subject alternative names (SAN) URI entry
4. A set of conditions MAY be defined to constrain the client certificates that SHALL be accepted.  An example might be a constraint that the certificate's first SAN URI entry must start with `spiffe://example.com/foo`, or that the certificate's first SAN DNS Name entry must end with `.example.com`.
5. A mapping MAY describe additional certificate attributes that may be encoded in the issued OAuth 2.0 access tokens to be interpreted as part of access policy decisions.  These MAY include any of the following properties
   - hex-encoded serial number
   - subject DN common name
   - subject DN organization name
   - subject DN subject organization unit
   - issuer DN common name
   - issuer DN organization name
   - issuer DN organization unit
   - the first subject alternative name (SAN) of type DNS name
   - the first subject alternative name (SAN) of type URI

Compatible OAuth 2.0 Authorization Servers supporting this token exchange profile MUST support mTLS. After the relying party has registered an X.509 Certificate federation profile with the OAuth 2.0 authorization server, in order to obtain access tokens, Workloads MUST present their X.509 Certificates during mTLS handshakes to establish a connection to the OAuth 2.0 Authorization Server.  Workloads MUST then send a request to token endpoint, in the manner described in [[RFC8693](https://datatracker.ietf.org/doc/html/rfc8693)]. The access token request is sent with the following properties:

* grant_type: REQUIRED. The value `urn:ietf:params:oauth:grant-type:token-exchange` indicates that a token exchange is being performed.
* resource: OPTIONAL. A URI that indicates the target service or resource where the client intends to use the requested security token.
* audience: REQUIRED for this Profile. A URI or other unique identifier for the relying party, assigned by the OAuth 2.0 Authorization Server.
* scope: OPTIONAL. A list of space-delimited, case-sensitive strings, as defined in [Section 3.3](https://datatracker.ietf.org/doc/html/rfc6749#section-3.3) of [[RFC6749](https://datatracker.ietf.org/doc/html/rfc6749)], that allow the client to specify the desired scope of the requested security token in the context of the service or resource where the token will be used.
* requested_token_type: MUST be `urn:ietf:params:oauth:token-type:access_token` for this token exchange profile.
* subject_token: REQUIRED. The fixed string `mtls_client_certificate` instructs the Authorization Server to obtain the subject token from the client's mTLS Certificate message, as defined in [Section 2](https://datatracker.ietf.org/doc/html/rfc8446#section-2) of [[RFC8446](https://datatracker.ietf.org/doc/html/rfc8446)]. The X.509 Certificate chain MUST chain to a previously-configured Trust Anchor certificate for the relying party, either directly or through one of the previously-configured Intermediate CA path-building certificates.
* subject_token_type: MUST be `urn:ietf:params:oauth:token-type:mtls` for this token exchange profile.

The request MUST ONLY be accepted if the X.509 Certificate used during mTLS chain to a previously-configured Trust Anchor via a certificate path that may include previously-configured intermediate CA certificates. The previously-configured subject claim selector MUST select a non-blank string from the certificate. The previously-configured conditions MUST accept the X.509 Certificate.

The response document contains the following properties (per [[RFC8693](https://datatracker.ietf.org/doc/html/rfc8693)]):

* access_token: REQUIRED. The security token issued by the authorization server in response to the token exchange request.
* issued_token_type: REQUIRED. Must be `urn:ietf:params:oauth:token-type:access_token` for this Profile.
* token_type: REQUIRED. Must be `bearer` for this Profile.
* expires_in: RECOMMENDED. The validity lifetime, in seconds, of the token issued by the authorization server.
* scope: OPTIONAL if the scope of the issued security token is identical to the scope requested by the client; otherwise, it is REQUIRED.
* refresh_token: MUST NOT be returned for this Profile.

The X.509 Certificate-to-Access Token Exchange Profile MUST NOT relax the validity constraint of the input security context. The returned Access Token MUST NOT have a not before claim that preceeds the `notBefore` constraint of the X.509 Certificate used. The returned Access Token MUST NOT have an expiration time claim that exceeds the `notAfter` constraint of the X.509 Certificate used.

To mitigate the impact of Access Token theft, it is RECOMMENDED that the returned Access Token be sender-constrained. The Authorization Server MAY bind the Access Token to the X.509 Certificate that was used to obtain it, in the manner described in [Section 3](https://datatracker.ietf.org/doc/html/rfc8705#name-mutual-tls-client-certifica) of [[RFC8705](https://datatracker.ietf.org/doc/html/rfc8705)]. Based on authentication policy, Resource Servers MAY enforce that an Access Token bound to an X.509 Certificate CAN NOT be used to access any protected resources, unless the same X.509 Certificate was used during the mTLS handshake to the Resource Server.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO

# IANA Considerations

This document has no IANA actions.

# Appendices

## Appendix 1 - Non-normative token exchange examples


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
