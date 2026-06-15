---
title: "OAuth 2.0 direct wallet interaction"
abbrev: "OAuth direct wallet interaction"
category: info

docname: draft-gresehyseni-oauth-direct-wallet-interation-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "grese-hyseni/oauth-direct-wallet-interaction"
  latest: "https://grese-hyseni.github.io/oauth-direct-wallet-interaction/draft-gresehyseni-oauth-direct-wallet-interation.html"

author:
 -
    fullname: "grese-hyseni"
    organization: Your Organization Here
    email: "grese.hyseni@rbinternational.com"

normative: 
 RFC8414:
 RFC6749:
 RFC9396:
 I-D.ietf-oauth-first-party-apps:
 OpenID4VP:
 title: OpenID for Verifiable Presentations 1.0
 target: https://openid.net/specs/openid-4-verifiable-presentations-1_0.html
 date: July 9, 2025
 author: 
 - ins: O. Terbu 
 - ins: T. Lodderstedt 
 - ins: K. Yasuda 
 - ins: D. Fett 
 - ins: J. Heenan
 IANA.oauth-parameters:
 USASCII:
 title: "Coded Character Set -- 7-bit American Standard Code for Information Interchange, ANSI X3.4"
 author:
 name: "American National Standards Institute"
 date: 1986

informative: 
 DC.API:
 title: Digital Credentials, W3C Editor's Draft
 target: https://w3c-fedid.github.io/digital-credentials/
 date: 03 July 2025
 DC.Android:
 title: Android DigitalCredential SDK
 target: https://developer.android.com/reference/kotlin/androidx/credentials/DigitalCredential

...

--- abstract

This document defines a protocol for Authorization Servers to request Verifiable Presentations as part of authentication and authorization processes.

--- middle

# Introduction

This specification provides a mechanism on top of First-Party Apps (FiPA) {{I-D.ietf-oauth-first-party-apps}} that enables Authorization Servers to request Verifiable Presentations as part of the authentication and authorization proccess.

OAuth for First-Party Apps (FiPA) is used as a base protocol as it extends the OAuth 2.0 Authorization Framework {{RFC6749}} with a new endpoint to support applications that want to control the process of obtaining authorization from the user using a native experience, which serves a smooth user experience when using digital credentials from a wallet app.

The scope of this document is not limited to First-Party Apps but extends to Third-Party Apps as well while keeping the rest of the base model as defined in First-Party Apps (FiPA) {{I-D.ietf-oauth-first-party-apps}}.

This specification is intended to support use cases including login, transaction approval, and presentation during issuance credential based on {{OpenID4VP}} and {{OpenID4VCI}}.

The integration of OpenID4VP Verifier functionality within an Authorization Server, as well as the internal communication between an Authorization Server and a Verifier, are outside the scope of this specification.

The browser-based wallet interactions are not the focus of this document since such flows are expected to work well in web environments without any additional extension to the existing authorization models such as the Authorization Code Flow defined in the section 4.1. of {{RFC6749}}.


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
