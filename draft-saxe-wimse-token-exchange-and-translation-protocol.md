---
title: WIMSE Token Exchange and Translation Protocol
abbrev: WIMSE Token Exchange & Translation
category: info

docname: draft-saxe-wimse-token-exchange-and-translation-protocol-latest

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
  github: "dhs-aws/wimse-token-exch-design-team"
  latest: "https://dhs-aws.github.io/wimse-token-exch-design-team/draft-saxe-wimse-token-exchange-and-translation-protocol.html"

author:
 -  fullname: Dean H. Saxe
    organization: Amazon Web Services
    email: deansaxe@amazon.com

 -  fullname: George Fletcher
    organization: Capital One
    email: george.fletcher@capitalone.com

 -  fullname: Andrii Deinega
    email: andrii.deinega@gmail.com

normative:

  RFC2119: # Keywords
  RFC6750: #OAuth
  RFC8174: # Ambiguity in Keywords
  RFC8693: # OAuth 2.0 Token Exchange

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


The specification defines the processes of token exchange and token translation for workloads.  Token exchange is well defined for OAuth 2.0 in {{RFC8693}}, allowing the exchange of access tokens, refresh tokens, OpenID Connect ID Token (OIDC), and SAML assertions for new OAuth access tokens. However, for workloads, there exist a broad array of input and output token types which must be considered beyond the input types supported by {{RFC8693}}.  These token types include, but are not limited to, SPIFFE SVIDs, x.509 certificates, Amazon sigv4A, macaroons, <...>.  Further, these tokens may be encoded in formats including JWT, CBOR, and protocol buffers (protobufs).  Given the variety and complexity of input and output token types and encoding, a strict token exchange that maintains all of the contextual information from the input token to the output token may not be possible.  We define these non-RFC8693 use cases with potentially lossy conversions as "token translation" (e.g. information may be lost in translation).   In this document we describe a workload profile for token exchange, using the mechanisms in {{RFC8693}}, and a new set of translations between arbitrary token types.  Additionally, we define mechanisms to enrich tokens during translation to support the use cases defined in <Use Cases Doc>.


--- middle

# Introduction

This specification defines a protocol for converting from one security token to another with support for both lossless and lossy conversions.  We refer to the lossless exchange as "token exchange" following the model defined in OAuth 2.0 Token Exchange {{RFC8693}}.  In this document we profile {{RFC8693}} to enable OAuth token exchange for workloads where the output is an OAuth Access Token or Refresh Token where no data is lost during the exchange.  "Token translation" describes all other conversions, including those where data loss may occur during conversion.  The terms Security Token, Security Token Service (STS), delegation, and impersonation are used in this document following the definitions in {{RFC8693}}.

Within the realm of workload identities, there are numerous types of security tokens that are commonly used including SPIFFE SVIDs, OAuth 2.0 Bearer Access Tokens {{RFC6750}}, and x.509 certificates. Additionally, security tokens are encoded in multiple formats such as JSON, CBOR, and protobufs.  In order to provide a mechanism for interoperability between different workloads we require the ability to convert from one token type or encoding to another for use across disparate systems.

In addition to translating security tokens between different types and formats, workload identity systems must be able to support changing the cryptographic properties of tokens, embedding tokens in one another, change the embedded context in a token, change the validity constraints, change or add subjects to the token, or add sender constraints.  This set of use cases for token exchange and translation are further described in https://github.com/yaroslavros/wimse-tokentranslation-requirements/blob/main/draft-rosomakho-wimse-tokentranslation-requirements.md. (todo: replace with a link to the ID once published.)

# Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.


# Terminology

TODO: Define terms used by this specification


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

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

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
