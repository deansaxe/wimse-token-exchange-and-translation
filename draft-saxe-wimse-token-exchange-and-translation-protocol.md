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
  group: WIMSE
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

The following document defines the processes of token exchange and token translation for workloads.  Token exchange is well defined for OAuth 2.0 in RFC8693, allowing the exchange of access tokens, refresh tokens, id_tokens, and SAML assertions for new OAuth access or refresh tokens.  However, for workloads, there exist a broad array of input and output token types which must be considered beyond the input types supported by RFC8693.  These token types include, but are not limited to, SPIFFE SVIDs, x.509 certificates, Amazon sigv4A, macaroons, <...>.  Further, these tokens may be encoded in formats including JWT, CBOR, and protocol buffers (protobufs).  Given the variety and complexity of input and output token types and encoding, a strict token exchange that maintains all of the contextual information from the input token to the output token may not be possible.  Therefore, we define these potentially lossy conversions as token translation (e.g. information is lost in translation).  In this document we describe the process and mechanisms for token exchange, using the existing mechanisms in RFC8693, and a new set of potentially lossy translations between arbitrary token types.  The authors expect that specific token translations will be profiled to ensure consistent handling across deployments. 


--- middle

# Introduction

TODO


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
