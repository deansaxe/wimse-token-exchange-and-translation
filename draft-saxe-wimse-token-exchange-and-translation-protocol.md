---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
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

normative:

informative:


--- abstract

The specification defines the processes of token exchange and token translation for workloads.  Token exchange is well defined for OAuth 2.0 in RFC8693, allowing the exchange of access tokens, refresh tokens, id_tokens, and SAML assertions for new OAuth access or refresh tokens. However, for workloads, there exist a broad array of input and output token types which must be considered beyond the input types supported by RFC8693.  These token types include, but are not limited to, SPIFFE SVIDs, x.509 certificates, Amazon sigv4A, macaroons, <...>.  Further, these tokens may be encoded in formats including JWT, CBOR, and protocol buffers (protobufs).  Given the variety and complexity of input and output token types and encoding, a strict token exchange that maintains all of the contextual information from the input token to the output token may not be possible.  We define these non-RFC8693 use cases with potentially lossy conversions as "token translation" (e.g. information may be lost in translation).   In this document we describe a workload profile for token exchange, using the mechanisms in RFC8693, and a new set of translations between arbitrary token types.  Additionally, we define mechanisms to enrich tokens during translation to support the use cases defined in <Use Cases Doc TODO>.

--- middle

# Introduction

TODO: What is a security token?  What is a STS? (see https://datatracker.ietf.org/doc/html/rfc8693, the intro has great definitions)

TODO: Define the need for token exchange & translation - refer to the use cases.

This specification defines a protocol for converting from one security token to another with support for high fidelity and lossy conversions.  We refer to the high fidelity exchange as "token exchange" as has been embodied in OAuth 2.0 Token Exchange (RFC8693).  We profile RFC8693 to enable OAuth token exchange for workloads where the output is an OAuth Access Token or Refresh Token.  "Token translation" describes all other conversions, including those where data loss may occur during conversion.  This protocol does not define the specifics of token translation between arbitrary token types.  Profiles must be defined to describe token translations between different token types, including any loss of context during translation.  Where the input and output token are of the same type, and the protocol herein is sufficient to meet the use cases defined in <USE CASES DOC>.


# Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.


# Terminology

TODO: Define terms used by this specification


# Overview

TODO: high level description of the token translation flow

TODO - define exchange vs. translation in terms of RFC8693 and WS-Trust. Translation may be perfect or introduce lost context

Token translation fills a gap that workloads must reinvent today.  For example, a common SPIFFE workload use case is to have a Kubernetes workload assume an AWS IAM role to access an S3 bucket.  <describe in broad terms https://spiffe.io/docs/latest/keyless/oidc-federation-aws/ or or similar for Google, etc.>

Token translation accounts for different token types, formats, encodings, and encyryption allowing for translation between most, but not all, token types using token translation profiles.  Profiles are not required when the input and output token are the same type.  Not all token input/output pairs are expected to be profiled.  During translation, the token translation service (TTS) may add, replace, or remove contextual data including attestations, validity constraints, and subjects. Cryptographic operations on the tokens may be replaced or supplemented, such as by adding PQC algorithms to a token encrypted and signed with classical algorithms.  For each use case defined in <USE CASES DOC>, this document defines the protocol requirements.

## Token Context Enrichment

TODO - what context do we enrich tokens with during translation? Embedding tokens, attestations, assertions, validity, change/add subject, sender constraints.  This doc can give specific guidance on adding context to a scoped set of token types that are common. Maybe a reference to the use cases is sufficient, along with a short description of any fields that the translation endpoint MUST add to a newly issued token.

## Lossy Translation

TODO - define what we mean by lossy.  What's lost?  Does this mean that some token translations lose valuable information?
TODO - provide a specific lossy scenario and use case.

Translation may be lossy or lossless, such as when exchanging an input token for an output token of the same format.

For example, assume the token translation endpoint receives a input SAML token with signed claims over the user's full name, user ID, email address, and a list of groups.  The output token format, T, only carries the user ID and list of groups (in addition to signatures and other metadata).  The token translation endpoint will follow the SAML -> T profile, mapping the context from input to output tokens, and dropping the user's full name and email address in the output token.  While data loss has occurred, the data lost was meaningless to the downstream systems consuming the token, T.  Lossy translation may impact downstream systems.  Implementers must be aware of the risks of lost context through token translation chains.


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
