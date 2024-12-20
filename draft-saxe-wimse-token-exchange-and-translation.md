---
title: WIMSE Token Exchange and Translation Protocol
abbrev: WIMSE Token Exchange & Translation
category: info

docname: draft-saxe-wimse-token-exchange-and-translation-latest

submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 1
date:
consensus: true
v: 3
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
  latest: "https://deansaxe.github.io/wimse-token-exchange-and-translation/draft-saxe-wimse-token-exchange-and-translation.html"

author:
 -  fullname: Dean H. Saxe
    email: dean@thesax.es

 -  fullname: George Fletcher
    organization: Capital One
    email: george.fletcher@capitalone.com

 -  fullname: Andrii Deinega
    email: andrii.deinega@gmail.com

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

The specification defines the processes of token exchange and token translation for workloads.  Token exchange is introduced in {{RFC8693}}, allowing the exchange of access tokens, OpenID Connect ID Tokens, and SAML assertions for new OAuth access tokens. However, for workloads, there exist a broad array of input and output token types which must be considered beyond the input types supported by {{RFC8693}}.  These token types include, but are not limited to, SPIFFE SVIDs, x.509 certificates, Kerberos service tickets, Amazon sigv4A, macaroons, <...>.  Further, these tokens may be encoded in formats including JWT OAuth 2.0 Acesss Tokens {{RFC9068}}, CWT {{RFC8392}}, and protocol buffers (protobufs).  Given the variety and complexity of input and output token types and encoding, a strict token exchange that maintains all of the contextual information from the input token to the output token may not be possible.  We define these non-RFC8693 use cases with potentially lossy conversions as "token translation" (e.g. information may be lost in translation).   In this document we describe a workload profile for token exchange, using the mechanisms in {{RFC8693}}, and a new set of translations between arbitrary token types.  Additionally, we define mechanisms to enrich tokens during translation to support the use cases defined in (todo: add link to use cases doc).


--- middle

# Introduction

This specification defines a protocol for converting from one security token to another with support for both lossless and lossy conversions.  We refer to the lossless exchange as "token exchange" following the model defined in OAuth 2.0 Token Exchange {{RFC8693}}.  In this document we profile {{RFC8693}} to enable OAuth token exchange for workloads where the output is an OAuth Access Token or Refresh Token where no data is lost during the exchange.  "Token translation" describes all other conversions, including those where data loss may occur during conversion.  The terms Security Token, Security Token Service (STS), delegation, and impersonation are used in this document following the definitions in {{RFC8693}}.

Within the realm of workload identities, there are numerous types of security tokens that are commonly used including SPIFFE SVIDs, OAuth 2.0 Bearer Access Tokens {{RFC6750}}, and x.509 certificates. Additionally, security tokens are encoded in multiple formats such as JSON, CBOR, and protobufs.  In order to provide a mechanism for interoperability between different workloads we require the ability to convert from one token type or encoding to another for use across disparate systems.

In addition to translating security tokens between different types and formats, workload identity systems must be able to support changing the cryptographic properties of tokens, embedding tokens in one another, change the embedded context in a token, change the validity constraints, change or add subjects to the token, or add sender constraints.  This set of use cases for token exchange and translation are further described in https://github.com/yaroslavros/wimse-tokentranslation-requirements/blob/main/draft-rosomakho-wimse-tokentranslation-requirements.md. (todo: replace with a link to the ID once published.)

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

Token translation fills a gap that development teams must solve for themselves today without standardized mechanisms.  For example, a common SPIFFE use case is to have a Kubernetes workload assume an AWS IAM role to access an S3 bucket.  This is accomplished by creating an OpenID Provider (OP) in the Kubernetes cluster and configuring AWS IAM as a Relying Party (RP) to obtain an ID token from the SPIFFE service. Using the id token, AWS STS AssumeRoleWithWebIdentity generates temporary sigV4 credentials for AWS allowing the workload to assume an AWS role and any permissions assigned to that role.  Similar mechanisms have been designed to support multiple cloud providers in the absence of standardized protocols.

Token translation accounts for different token types, formats, encodings, and encyryption allowing for translation between most, but not all, token types using token translation profiles. This protocol does not define the specifics of token translation between arbitrary token types.  Profiles must be defined to describe token translations between different token types, including any loss of context during translation.  Where the input and output token are of the same type and the conversion is lossless, the protocol defined within this document is sufficient to meet the use cases defined in "USE CASES DOC".  Not all token input/output pairs are expected to be profiled.

## Token Context Enrichment

TODO - what context do we enrich tokens with during translation? Embedding tokens, attestations, assertions, validity, change/add subject, sender constraints.  This doc can give specific guidance on adding context to a scoped set of token types that are common. Maybe a reference to the use cases is sufficient, along with a short description of any fields that the translation endpoint MUST add to a newly issued token.

## Lossy Translation

TODO - define what we mean by lossy.  What's lost?  Does this mean that some token translations lose valuable information?
TODO - provide a specific lossy scenario and use case.

Translation may be lossless, such as when exchanging an input token for an output token of the same format, or lossy when exchanging an input token for an output token of a different format. An example of lossy translation is detailed in the example above.  In this case, the aud claim of the id token maps to the AWS IAM role used to create the AWS temporary credentials.
The aud (if no azp claim is present), sub, and amr claims are mapped to STS Session Keys with the same name. Other claims in the id token are dropped, resulting in an loss of context.

Lossy translation may impact downstream systems.  Implementers must be aware of the risks of lost context through token translation.


# Token Translation Endpoint

TODO - Define a new translation endpoint.

# Token Translation Profiles

TODO - this draft does not define normative specs for translating from arbitrary format to another arbitrary format.  Profiles describing specific token translations must be developed and their names (possibly?) registered with IANA.  Profiles will define any losses that may occur during translation and the risks associated with the loss of context.  Not all token pairs can be translated, some may only be translatable in one direction.

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

As countermeasures against replay attacks and various forms of misuse, an authorization server performing token exchange
- MUST ensure a client (whether it is an OAuth client or a workload) is allowed to perform such operation.
- SHOULD ensure that a provided security token allows to perform such operation on it.
- SHOULD ensure it itseld, as an AS, is the intended audience of a token being exchanged. It is typical for self-contained tokens to include the aud claim (an array of strings) representing their intended audience (other types of tokens provide other means for the same).
- a value in the subject_token_type parameter MUST correspond to an actual type of a security token provided in the subject_token parameter ({{RFC8693}}).
These countermeasures become even more significant when an entity issuing security tokens and an AS performing exchange of them reside in different security domains.

An extra care should be taken for tokens that can be passed around using the front channel, and for those tokens that do not explicitly define their type. Examples here would be OpenID Connect ID Token, and various assertions represented as JWTs.

TODO Security - data loss in token translation may impact authZ decisions.  Be careful when allowing multiple token translations since losses may grow over each step of translation.

Embedding input tokens into output tokens can reduce this risk by allowing more complete context, at the risk of expanding the token size beyond what is practical.


# IANA Considerations

This document has no IANA actions.

# Appendices

## Appendix 1 - Non-normative token exchange examples


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
