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

docname: draft-saxe-wimse-token-exchange-and-translation-protocol
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 1
date: 
consensus: true
v: 3
area: AREA
workgroup: WIMSE
keyword:
 - workload identity
 - token exchange
 - token translation
venue:
  group: Worload Identity in Multi System Environments
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: dhs-aws/wimse-token-exch-design-team
  latest: https://example.com/LATEST

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

The specification defines the processes of token exchange and token translation for workloads.  Token exchange is well defined for OAuth 2.0 in RFC8693, allowing the exchange of access tokens, refresh tokens, id_tokens, and SAML assertions for new OAuth access or refresh tokens. However, for workloads, there exist a broad array of input and output token types which must be considered beyond the input types supported by RFC8693.  These token types include, but are not limited to, SPIFFE SVIDs, x.509 certificates, Amazon sigv4A, macaroons, <...>.  Further, these tokens may be encoded in formats including JWT, CBOR, and protocol buffers (protobufs).  Given the variety and complexity of input and output token types and encoding, a strict token exchange that maintains all of the contextual information from the input token to the output token may not be possible.  We define these potentially lossy conversions as "token translation" (e.g. information may be lost in translation).   In this document we describe the process and mechanisms for token exchange, using the existing mechanisms in RFC8693, and a new set of translations between arbitrary token types.  Additionally, we define mechanisms to enrich tokens during translation to support the use cases defined in <Use Cases Doc TODO>.

--- middle

# Introduction

Define the need for token exchange & translation - refer to the use cases. 

## Token Translation Endpoint

## Token Exchange vs. Token Translation

TODO - define exchange vs. translation in terms of RFC8693 and WS-Trust. Translation may be perfect or introduce lost context

## Lossy Translation

TODO - define what we mean by lossy.  What's lost?  Does this mean that some token translations lose valuable information? 
TODO - provide a specific lossy scenario and use case.

## Token Context Enrichment

TODO - what context do we enrich tokens with during translation? Embedding tokens, attestations, assertions, validity, change/add subject, sender constraints.  This doc can give specific guidance on adding context to a scoped set of token types that are common.

## Token Translation Profiles

TODO - this draft does not define normative specs for translating from arbitrary format to another arbitrary format.  Profiles describing specific token translations must be developed and their names (possibly?) registered with IANA.  Profiles will define any losses that may occur during translation and the risks associated with the loss of context.  Not all token pairs can be translated, some may only be translatable in one direction.



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
