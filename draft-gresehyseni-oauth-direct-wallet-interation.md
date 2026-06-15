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
 OpenID4VCI:
  title: OpenID for Verifiable Credential Issuance 1.0
  target: https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html
  date: 16 September 2025
  author:
    - ins: T. Lodderstedt
    - ins: K. Yasuda
    - ins: T. Looker
    - ins: P. Bastian
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

# Protocol

In the model introduces in this document, the Client acts as an orchestrator of the Wallet interaction while the Authorization Server remains responsible for authorization decisions and for integrating Verifier Capabilities.

The mechanisms defined in this document support:

* Same-device Wallet interactions;
* Cross-device Wallet interactions;
* Deployments using OpenID4VP {{OpenID4VP}}; and
* Deployments using platform Digital Credentials APIs {{DC.API}} or equivalent native SDKs.

The Verifier is a separate logical component, as it has it's own endpoints and functionality, as defined in OpenID4VP {{OpenID4VP}}.


## Same Device Flow {#same-device-flow}

The following figure illustrates the High Level use case in which the Authorization Server (AS) requests a Verifiable Presentation from the Wallet as part of the authorization process and the wallet is on the same device as the client app. This does not represent the OID4VP over DC API, see section {{oid4vp-over-dc-api}}.


```text
                                                                +-------------------+                +------------+ 
                                                                |   Authorization   |                |  Verifier  |
                                                                |      Server       |                +------------+
+----------+                 +----------+  (A) Authorization    |+-----------------+|                      |
|  Wallet  |                 |  Third   |  Challenge Request    |+-----------------+| (A) Create           |
|          |                 |  Party-  |---------------------->||  Authorization  ||     Presentation     |
|          |                 |  Client  |          :            ||   Challenge     ||     Request          |
|          |                 |          |                       ||    Endpoint     ||--------------------->|
|          |                 |          |                       ||                 ||                      |
|          |                 |          |                       ||                 ||<---------------------|
|          |                 |          |                       ||                 || (B) Presentation     |
|          |                 |          |<----------------------||                 ||     Request          |
|          |                 |          | (B)Authorization      ||                 ||                      |
|          |                 |          | Error Response        ||                 ||                      |
|          |                 |          | (Presentation Req.)   ||                 ||                      |
|          |<----------------|          |                       ||                 ||                      |
|          |(C) Presentation |          |          :            ||                 ||                      |
|          |    Request      |          |          :            ||                 ||                      |
|          | (response_mode) |          |                       ||                 ||                      |
|          |                 |          |                       ||                 ||                      |
|          |---------------->|          | (E) Authorization     ||                 ||                      |
|          |(D) Presentation |          | Challenge Request     ||                 || (E) Validate         |
|          |    Response     |          | (Presentation Res.)   ||                 ||     Presentation     |
|          | (reponse_code/  |          |---------------------->||                 ||     Response         |
|          |  vp_token)      |          |                       ||                 ||--------------------->|
|          |                 |          |                       ||                 ||                      |
|          |                 |          |                       ||                 ||<---------------------|
|          |                 |          |<----------------------||                 || (F) Presentation     |
|          |                 |          | (F) Authorization     |+-----------------+|     Data             |
|          |                 |          |     Code Response     |                   |                      |
|          |                 |          |                       |                   |                      |
|          |                 |          | (G) Token             |                   |                      |
|          |                 |          |     Request           |+-----------------+|                      |
|          |                 |          |---------------------->||      Token      ||                      |
|          |                 |          |                       ||     Endpoint    ||                      |
|          |                 |          |<----------------------||                 ||                      |
|          |                 |          | (H) Access Token      |+-----------------+|                      |
|          |                 |          |                       |                   |                      |
+----------+                 +----------+                       +-------------------+                      |
```


(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. The client MUST provide `type` and `redirect_uri` to indicate that it is requesting an OID4VP Authorization Request for a same-device flow. The Client MAY provide these parameters in the initial or in a subsequent Authorization Challenge Request. See Section {{authorization-challenge-request}} for details.

(B) The Authorization Server responds with an authorization error containing the required/supported authentication methods, where at least one is of type "verifiable_presentation". The Client may be asked to send further Authorization Requests to provide sufficient data for the Authorization Server to issue an OID4VP Authorization Request (Presentation Request) as part of this Authorization Error Response. The Presentation Response SHOULD contain `response_mode=fragment` as defined in section 3.1. of {{OpenID4VP}}. See Section {{authorization-error-response}} for details.

(C) The Client invokes the wallet on the user’s device with the OID4VP Authorization Request, where user is expected to authenticate/authorize and consent to sharing the credentials with the verifier.

(D) The Client receives from the wallet the Presentation Response, which MAY include a `vp_token` or other presentation artifacts depending on the wallet invocation method and the Presentation Request used in (C). If a `response_code` is present, it indicates the wallet submitted the `vp_token` directly to the Verifier’s Response Endpoint.

(E) The Client forwards the wallet response from (D) in a subsequent Authorization Challenge Request to the Authorization Server.

(F) Upon successful validation of the Presentation Response and completion of authorization checks, the Authorization Server issues an Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 or OpenID Connect tokens.


## Cross Device Flow {#cross-device-flow}

The following figure illustrates the High Level use cases in which the Authorization Server (AS) requests a Verifiable Presentation from the Wallet as part of the authorization process and the wallet is on a different device from the client app. This does not represent the OID4VP over DC API, see section {{oid4vp-over-dc-api}}.


```text
                                                                  +-------------------+                +------------+ 
                                                                  |   Authorization   |                |  Verifier  |
                                                                  |      Server       |                +------------+
+----------+                   +----------+ (A) Authorization     |+-----------------+|                    |
|  Wallet  |                   |  Third   |     Challenge Request |+-----------------+| (A) Create         |
|          |                   |  Party-  |---------------------->||  Authorization  ||     Presentation   |
|          |                   |  Client  |          :            ||   Challenge     ||     Request        |
|          |                   |          |                       ||    Endpoint     ||------------------> |
|          |                   |          |                       ||                 ||                    |
|          |                   |          |                       ||                 ||<------------------ |
|          |                   |          |                       ||                 || (B) Presentation   |
|          |                   |          |<----------------------||                 ||     Request        |
|          |                   |          | (B) Authorization     ||                 ||                    |
|          |                   |          |     Error Respons     ||                 ||                    |
|          |                   |          |    (Presentation Req.)||                 ||                    |
|          |<------------------|          |                       ||                 ||                    |
|          |(C) Presentation   |          |          :            ||                 ||                    |
|          |    Request        |          |          :            ||                 ||                    |
|          |                   |          |                       ||                 ||                    |
|          |(D) Presentation   |          |                       ||                 ||                    |
|          |    Response       |          |                       ||                 ||                    |
|          |---------------------------------------------------------------------------------------------->|
|          |                   |          |                       ||                 ||                    |
|          |                   |          |                       ||                 ||(D) Get Presentation|
|          |                   |          |                       ||                 ||    Data            |
|          |                   |          |                       ||                 ||------------------->|
|          |                   |          |                       ||                 ||                    |
|          |                   |          |                       ||                 ||<-------------------|
|          |                   |          |<----------------------||                 ||(D) Presentation    |
|          |                   |          | (F) Authorization     |+-----------------+|     Data           |
|          |                   |          |     Code Response     |                   |                    |
|          |                   |          |                       |                   |                    |
|          |                   |          | (G) Token             |                   |                    |
|          |                   |          |     Request           |+-----------------+|                    |
|          |                   |          |---------------------->||      Token      ||                    |
|          |                   |          |                       ||     Endpoint    ||                    |
|          |                   |          |<----------------------||                 ||                    |
|          |                   |          | (H) Access Token      |+-----------------+|                    |
|          |                   |          |                       |                   |                    |
+----------+                   +----------+                       +-------------------+                    |
```


(A) (A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. The client MUST provide `type` and ommit the `redirect_uri` to indicate that it is requesting an OID4VP Authorization Request for a cross-device flow. The Client MAY provide these parameters in the initial or in a subsequent Authorization Challenge Request. See Section {{authorization-challenge-request}} for details.

(B) The Authorization Server responds with an authorization error containing the required authentication methods, where at least one is of type "verifiable_presentation". The Client may be asked to send further Authorization Requests to provide sufficient data for the Authorization Server to issue an OID4VP Authorization Request (Presentation Request) as part of this Authorization Error Response. The Presentation Response SHOULD contain `response_mode=direct_post` as defined in section 3.2. of {{OpenID4VP}}. See Section {{authorization-error-response}} for details.

(C) The Client generated the QR Code for the user to invoke the wallet on another device wich contains the OID4VP Authorization Request.

(D) The wallet sends the Presentation Response to the Verifier's Response Endpoint, which MAY include a `vp_token` or other presentation artifacts depending on the wallet invocation method and the Presentation Request used in (C). If a `response_code` is present, it indicates the wallet submitted the `vp_token` directly to the Verifier’s Response Endpoint.

(F) Upon successful validation of the Presentation Response and completion of authorization checks, the Authorization Server issues an Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 or OpenID Connect tokens.

# Authorization Challenge Request {#authorization-challenge-request}

## Initial authorization request {#init-authorization-challenge-request}

## Subsequest authorization challenge request {#subs-authorization-challenge-request}

# Authorization Error Response {#authorization-error-response}

# Example flows {#example-flows}

## OpenID4VP Same-Device Flow {#oid4vp-same-device-example}

## OpenID4VP Same-Device Flow using `direct_post` {#oid4vp-same-device-direct-post-example}

## OpenID4VP over DC API {#oid4vp-over-dc-api}

# Security Considerations {#security-considerations}

TODO Security

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
