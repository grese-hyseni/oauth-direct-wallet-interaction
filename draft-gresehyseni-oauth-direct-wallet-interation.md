---
title: "OAuth 2.0 Direct Wallet interaction"
abbrev: "OAuth direct Wallet interaction"
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
 - Wallet
 - FiPA
 - OAuth
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "grese-hyseni/oauth-direct-wallet-interaction"
  latest: "https://grese-hyseni.github.io/oauth-direct-wallet-interaction/draft-gresehyseni-oauth-direct-wallet-interaction.html"

author:
 -
    fullname: "Gresë Hyseni"
    organization: Raiffeisen Bank International
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

This document defines a protocol extension that enables OAuth 2.0 Authorization Servers to request verifiable presentations of digital credentials as part of authorization flows.

--- middle

# Introduction

Digital credentials are increasingly used for authentication and authorization. Existing credential presentation protocols, such as {{OpenID4VP}}, produce presentation artifacts (e.g., VP Tokens) and not OAuth authorization artifacts like Authorization Codes or Access Tokens, limiting seamless integration with OAuth authorization flows.

This document specifies a method to integrate Presentation of Credentials into OAuth authorization flows, enabling OAuth 2.0 Authorization Servers to request digital credential presentations as part of the authentication and authorization process.

Specifically, this specification extends OAuth 2.0 for First-Party Applications (FiPA) {{I-D.ietf-oauth-first-party-apps}} by introducing:
* New parameters for the FiPA-defined Authorization Challenge Requests (see {{authorization-challenge-request}}).
* New fields for Authorization Error Responses (see {{authorization-error-response}}).

While building upon the FiPA model, this specification also applies to third-party applications as defined in {{RFC6749}}.

This document focuses on native applications. Existing browser-based OAuth authorization flows, such as the Authorization Code Grant defined in Section 4.1 of {{RFC6749}}, generally suffice for Wallet interactions and do not require the extensions defined here.

This document defines mechanisms that facilitate Wallet interactions across same-device and cross-device scenarios, supporting deployments based on OpenID4VP {{OpenID4VP}} as well as OpenID4VP over Digital Credentials APIs (DC API) {{DC.API}} or equivalent native SDKs.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This specification uses the following terms:

* Access Token, Authorization Code, Client, and Third-Party Application as defined by [RFC6749].

* Authorization Challenge Request, Authorization Challenge Error Response, Authorization Challenge Response as defined by [I-D.ietf-oauth-first-party-apps].

* Verifier, Wallet, Verifiable Presentation, VP Token, OID4VP Authorization Request (Presentation Request), and OID4VP Authorization Response (Presentation Response) as defined by [OpenID4VP].

# Protocol

This specification focuses on the OAuth 2.0 authorization flow based on the First-Party Applications (FiPA) model {{I-D.ietf-oauth-first-party-apps}}. The Client and Authorization Server (AS), as defined in {{RFC6749}}, are the primary actors depicted in the flow.

While the Verifier and Wallet play critical roles in credential presentation, their interactions and integration with the AS and Client are outside the scope of this document and are governed by {{OpenID4VP}}.

The Client acts as the orchestrator for Wallet interactions within the OAuth flow, and the Authorization Server remains responsible for authorization decisions and Verifier integration.

This abstraction allows the specification to focus on extending FiPA flows to support verifiable credential presentations without detailing Wallet or Verifier protocols. For an extended flow please see {{extended-oauth-fipa-oid4vp-flow}}.

## High Level Flow {#high-level-flow}

The following figure illustrates a high-level protocol flow in which the Authorization Server (AS) requests a Verifiable Presentation from the Wallet as part of the authorization process.

~~~ ascii-art
                  +----------+                       +-------------------+
                  |  Client  |                       |   Authorization  |
                  |          |                       |      Server      |
                  |          |  (A) Authorization    |+----------------+|
                  |          |  Challenge Request    ||  Authorization ||
                  |          |---------------------->||   Challenge    ||
                  |          |                       ||    Endpoint    ||
                  |          |<----------------------||                ||
                  |          | (B) Authorization     ||                ||
                  |          |     Error Response    ||                ||
                  |          |    (w/ auth_methods)  ||                ||
                  |          |                       ||                ||
                  |          |  (C) Authorization    ||                ||
                  |          |  Challenge Request    ||                ||
                  |          |  (w/ auth_method)     ||                ||
                  |          |---------------------->||                ||
                  |          |                       || +----------------------------------------+
                  |          |<----------------------|| | Note: AS creates Presentation Request  |
+--------------------------+ | (D) Authorization     || | with the Verifier                      |
| Note: Client interacts   | |     Error Response    || +----------------------------------------+
| with the Wallet          | |     (w/ Pres. Req.    ||                ||
+--------------------------+ |      response_mode)   ||                ||
                  |          |                       ||                ||
                  |          | (E) Authorization     ||                ||
                  |          |     Challenge Request ||                ||
                  |          |     (w/ Pres. Res.    ||                ||
                  |          |     or Poll)          ||                ||
                  |          |---------------------->||                ||
                  |          |                       || +--------------------------------------------------------+
                  |          |                       || | Note: AS receives and validates Presentation Response  |
                  |          |                       || | with the Verifier                                      |
                  |          |                       || +--------------------------------------------------------+
                  |          |<----------------------||                ||
                  |          | (F) Authorization     |+----------------+|
                  |          |     Code Response     |                  |
                  |          |                       |                  |
                  |          | (G) Token             |                  |
                  |          |     Request           |+----------------+|
                  |          |---------------------->||     Token      ||
                  |          |                       ||    Endpoint    ||
                  |          |<----------------------||                ||
                  |          | (H) Access Token      |+----------------+|
                  |          |                       |                  |
                  +----------+                       +-------------------+
~~~

(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. See Section {{authorization-challenge-request}} for details.

(B) The Authorization Server responds with an Authorization Error Response indicating the supported challenge methods (`challenge_methods`) to the Client. See {{authorization-error-response}} for details.

(C) The Client sends a subsequent Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint where it MUST specify the challenge method (`challenge_method`) and MAY provide the Client’s redirect URI (`oid4vp_redirect_uri`) to request an OID4VP Authorization Request.

(D) The Authorization Server determines the challenge method and responds with an Authorization Error Response containing the OID4VP Authorization Request (`oid4vp_request`) and `auth_session`. See {{authorization-error-response}} for details.

(E) Following the Wallet interaction, the Client submits a subsequent Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint to request the Authorization Code. If the Client retrieved the OID4VP Authorization Response directly from the Wallet, it includes it; if not, it polls using the `auth_session`. The means by which the Client knows whether this is a same or cross device flow, are outside of the scope for this specification, and are known based

(F) After the Authorization Server validates the OID4VP Authorization Response with the Verifier and performs authorization checks, it responds with an Authorization Challenge Response including the Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 tokens.

# Authorization Challenge Request {#authorization-challenge-request}

This document extends the FiPA {{I-D.ietf-oauth-first-party-apps}} Authorization Challenge Request by introducing the following parameters:

"challenge_method":
:    OPTIONAL.  A string indicating the challenge method requested by the Client, e.g., a user selected method on the client side. This document defines the value `oid4vp` and `oid4vp_dc_api`. Parties using other values must mutually agree on their meanings, which may be context-specific.

"oid4vp_redirect_uri":
:    OPTIONAL.  The Client URI to which the Wallet redirects the user in same-device flows. This parameter MUST be included to enable same-device flows when Client does not support DC API or an equivalent SDK. This parameter SHOULD NOT be sent when `challenge_method=oid4vp_dc_api`; if sent, the Authorization Server MUST ignore it.

"oid4vp_response_code":
:    OPTIONAL.  The value of `response_code`, as defined in section 13.3 of {{OpenID4VP}}, returned from the Wallet as part of the Presentation Response.

"oid4vp_vp_token":
:    OPTIONAL.  The VP Token value, as defined in section 8 of {{OpenID4VP}}, returned from the Wallet as part of the Presentation Response (e.g., in DC API or same-device redirect flows).

When the Client receives the OID4VP Authorization Response directly from the Wallet, it MAY include additional parameters in a subsequent Authorization Challenge Request to the Authorization Server to indicate an OID4VP Error Response, as defined in section 8.5 of {{OpenID4VP}}.

## Initial Authorization Challenge Request {#init-authorization-challenge-request}
The initial Authorization Challenge Request contains only the information available at that time. For example, if the Client lacks prior knowledge of preferred user login methods, it may omit the `challenge_method` and related parameters. See {{example-flows}}. Conversely, if such information is known, the Client can include it in the initial request.

## Subsequent Authorization Challenge Requests {#subs-authorization-challenge-request}
If the Client omits the `challenge_method` or specifies one not supported by the Authorization Server, the Authorization Server MAY respond with an Authorization Error Response listing supported `challenge_methods`. In this case, the Client SHOULD include an appropriate `challenge_method` and related parameters in subsequent Authorization Challenge Requests to request the desired challenge, e.g., `oid4vp`. See {{example-flows}}.

# Authorization Error Response {#authorization-error-response}
This document extends FiPA's {{I-D.ietf-oauth-first-party-apps}} Authorization Error Response by adding the following attributes, used when the error code "insufficient_authorization" is returned:

"challenge_methods":
:    OPTIONAL. A JSON array of strings identifying the challenge methods supported by the Authorization Server. This document defines the value `oid4vp` and `oid4vp_dc_api` for OID4VP and OID4VP over DC API, respectively. Parties using other values MUST mutually agree on their meanings, which may be context-specific.

"oid4vp_authorization_request":
:   OPTIONAL. An OpenID4VP {{OpenID4VP}} request URL.

"status":
:    OPTIONAL.  Indicates the status of the ongoing operation. This document uses the value "pending".

## OID4VP response_mode

This specification RECOMMENDS that the Authorization Server instruct the Verifier to use `response_mode=direct_post` or `response_mode=direct_post.jwt` in the OID4VP Authorization Request returned within the Authorization Error Response for non-DC API same-device and cross-device flows. This eliminates ambiguity for both the Authorization Server and the Verifier by removing the need to determine the Wallet’s device context, thereby avoiding restrictions to same-device interactions (e.g., `response_mode=fragment`).

Using these response modes also provides security benefits as detailed in Section {{security-considerations}}.

However, when the Authorization Server and Verifier have prior knowledge that the Wallet is on the same device (e.g., based on user preferences), they may use response_mode=fragment in the OID4VP Authorization Request (e.g. the Client is a First-Party Application to the Authorization Server).

The full set of `response_mode` values is defined in {{OpenID4VP}} and referenced specifications.

# Example flows {#example-flows}

This section provides non-normative examples illustrating how this specification supports various flows and deployment models.

## OpenID4VP Same-Device Flow with `response_mode=direct_post` {#oid4vp-same-device}

In this example, the Client requests `challenge_method=oid4vp` and includes `oid4vp_redirect_uri` (which does not explicitly indicate a same-device flow). The Authorization Server generates an OID4VP Authorization Request with `response_mode=direct_post`, instructing the Wallet to submit the VP Token directly to the Verifier’s Response Endpoint, as defined in section 8.2 of {{OpenID4VP}}. The Wallet is invoked on the same device (e.g., when the user clicks a wallet invocation link instead of scanning a QR code), so the Client waits for a `response_code` from the Wallet. The Client forwards this `response_code` in a subsequent Authorization Challenge Request to the Authorization Server via the `oid4vp_response_code` parameter.

Example: Initial Authorization Challenge Request (no preferred authentication):

~~~ http-message
   POST /authorize-challenge HTTP/1.1
   Host: server.example.com
   Content-Type: application/x-www-form-urlencoded

   scope=openid
   &client_id=bb16c14c73415
~~~

Example: Authorization Error Response indicating supported challenge methods

~~~ http-message
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store

{
   "error": "insufficient_authorization",
   "auth_session": "uY29tL2F1dGhlbnRpY",
   "challenge_methods": ["oid4vp","oid4vp_dc_api"]
}
~~~

Example: Subsequent Authorization Challenge Request requesting OID4VP Authorization Request (non-DC API)

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=uY29tL2F1dGhlbnRpY
&challenge_method=oid4vp
&oid4vp_redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
~~~


Example: Authorization Error Response containing OID4VP Authorization Request

~~~ http-message
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store

{
   "error": "insufficient_authorization",
   "auth_session": "uY29tL2F1dGhlbnRpY",
   "challenge_method": ["oid4vp"],
   "oid4vp_authorization_request": "https://Wallet.example.org/universal-link?
   response_type=vp_token
   &response_mode=direct_post
   &client_id=redirect_uri%3Ahttps%3A%2F%2Fclient.example.org%2Fcb
   &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
   &dcql_query=...
   &nonce=n-0S6_WzA2Mj
   &client_metadata=%7B%22vp_formats_supported%22%3A%7B%22dc%2Bsd-jwt%22%3A%7B%22sd-jwt_alg_values%22%3A%5B%22ES256%22%5D%2C%22kb-jwt_alg_values%22%3A%5B%22ES256%22%5D%7D%7D%7D"
}
~~~

Example: Subsequent Authorization Challenge Request forwarding the `response_code` returned from the Wallet

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=bXlzZXNzaW9uMTIzNDU2
&oid4vp_response_code=091535f699ea575c7937fa5f0f454aee
~~~

Example: Successful Authorization Challenge Response returning the `authorization_code`

~~~ http-message
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
   "authorization_code": "c2VjdXJlL2F1dGgvY29kZQ=="
}
~~~

## OpenID4VP Cross-Device Flow using `direct_post` {#oid4vp-cross-device-direct-post}

In this example, the Client omits `oid4vp_redirect_uri`, indicating a cross-device flow (e.g., based on pre-stored user preferences). The Authorization Server issues an OID4VP Authorization Request with `response_mode=direct_post`.

The Client polls for the presentation result by sending subsequent Authorization Challenge Requests including only the `auth_session` parameter:

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=bXlzZXNzaW9uMTIzNDU2
~~~

Example: Authorization Error Response indicating that Wallet presentation and/or necessary Authorization Server checks are still in progress:

~~~ http-message
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store

{
   "error": "insufficient_authorization",
   "auth_session": "bXlzZXNzaW9uMTIzNDU2",
   "status": "pending"
}
~~~

## OpenID4VP over DC API {#oid4vp-over-dc-api}

The Client indicates support for the DC API by including `challenge_method=oid4vp_dc_api` in the Authorization Challenge Request. The Authorization Server responds with a Presentation Request formatted for OpenID4VP over DC API.

In this model, the OID4VP Authorization Response is always returned to the Client via the DC API. The Client then forwards the received `oid4vp_vp_token` to the Authorization Server in a subsequent Authorization Challenge Request.

Example: Subsequent Authorization Challenge Request requesting OID4VP Authorization Request over DC API:

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=uY29tL2F1dGhlbnRpY
&challenge_method=oid4vp_dc_api
~~~

Example: Authorization Error Response containing OID4VP Authorization Request over DC API

~~~ http-message
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store

{
   "error": "insufficient_authorization",
   "auth_session": "uY29tL2F1dGhlbnRpY",
   "oid4vp_authorization_request": "https://Wallet.example.org/universal-link?
   response_type=dc_api
   &response_mode=fragment
   &client_id=redirect_uri%3Ahttps%3A%2F%2Fclient.example.org%2Fcb
   &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
   &dcql_query=...
   &nonce=n-0S6_WzA2Mj
   &client_metadata=%7B%22vp_formats_supported%22%3A%7B%22dc%2Bsd-jwt%22%3A%7B%22sd-jwt_alg_values%22%3A%5B%22ES256%22%5D%2C%22kb-jwt_alg_values%22%3A%5B%22ES256%22%5D%7D%7D%7D"
}
~~~

Example: Subsequent Authorization Challenge Request forwarding the Presentation Response:

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=uY29tL2F1dGhlbnRpY
&oid4vp_vp_token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMTIzIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNjI3MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
~~~


## OpenID4VP with RAR {#oid4vp-with-rar}

A common use case is when the user is already authenticated but must provide explicit approval via their Wallet.

In this scenario, the Client MUST include the `authorization_details` parameter, as defined in {{RFC9396}}, within the Authorization Challenge Request. This enables the Authorization Server to communicate specific authorization requirements to the Wallet, allowing the user to approve the requested action.

A non-normative example Authorization Challenge Request is shown below:

~~~  http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=bXlzZXNzaW9uMTIzNDU2
&oid4vp_response_code=091535f699ea575c7937fa5f0f454aee
&authorization_details='{
  "type": "openid_credential",
  "actions": ["approve"],
  "locations": ["https://example.com/resource/123"]
}'
~~~

Upon receiving this request, the Authorization Server MAY use the contents of `authorization_details` to populate the `transaction_data` in the OID4VP Authorization Request. This enables the Wallet to present the user with the relevant information and prompt for explicit consent for the specified action(s), as defined in {{OpenID4VP}}.

# Security Considerations {#security-considerations}

Implementations SHOULD comply with the security and privacy considerations outlined in all referenced specifications relevant to this document, including but not limited to OAuth 2.0 Authorization Framework {{RFC6749}}, OpenID4VP {{OpenID4VP}}, and Rich Authorization Requests (RAR) {{RFC9396}}.

In deployments where Authorization Requests and Presentation Responses transit through third-party applications or intermediaries, additional risks to integrity, confidentiality, and privacy arise due to potential exposure of sensitive data.

To mitigate these risks:

* Authorization Requests are RECOMMENDED to be signed by the Verifier to ensure authenticity and integrity.
* Presentation Responses RECOMMENDED to be signed and encrypted to protect confidentiality and privacy, especially in flows where responses pass through third-party components, such as OpenID4VP over DC API or OpenID4VP with `response_mode=fragment`.

Consistent with FiPA guidance, when third-party Clients are involved, Authorization Servers MUST NOT require Clients to handle sensitive user credential data directly. Instead, the `response_mode=direct_post` flow SHOULD be used, allowing the Wallet to submit the Verifiable Presentation directly to the Verifier’s Response Endpoint. This minimizes exposure of sensitive data to third-party Clients and reduces associated risks.

## Signed Authorization Requests

To mitigate tampering and unauthorized disclosure risks, it is RECOMMENDED that the Authorization Server, acting as the OpenID4VP Verifier, signs Authorization Requests to ensure authenticity and integrity.

## Signed and Encrypted Presentation Responses

To protect the integrity, confidentiality, and privacy of Presentation Responses, it is RECOMMENDED that the Authorization Server require signed Presentation Responses and support encrypted responses intended for the Authorization Server.

In particular, this is relevant for deployment models such as OpenID4VP over DC API, where Presentation Responses may transit through a third-party application before reaching the Authorization Server.


# IANA Considerations

OAuth Parameters Registration

IANA has (TBD) registered the following values in the IANA "OAuth Parameters" registry of [IANA.oauth-parameters] established by [RFC6749].

   *Parameter name*: challenge_method

   *Parameter usage location*: authorization request

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: challenge_methods

   *Parameter usage location*: authorization response

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: oid4vp_authorization_request

   *Parameter usage location*: authorization response

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: oid4vp_response_code

   *Parameter usage location*: authorization request

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: oid4vp_vp_token

   *Parameter usage location*: authorization request

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification

--- back

# Appendix A: Extended OAuth FiPA Authorization Flow with OpenID4VP (including over DC API) {#extended-oauth-fipa-oid4vp-flow}

This appendix illustrates an extended flow combining OAuth 2.0 for First-Party Applications (FiPA) {{I-D.ietf-oauth-first-party-apps}} with OpenID4VP {{OpenID4VP}} verifiable presentation requests. It distinguishes between the Authorization Server (AS), as defined in {{RFC6749}}, and the Verifier, as defined in {{OpenID4VP}}, treating them as separate logical components. The integration and communication between these components remain outside the scope of this document.

The flow includes optional steps that differ for OID4VP same-device, cross-device, or OID4VP over DC API flows.

~~~ ascii-art
+----------+                 +----------+                       +-------------------+                      +------------+
|  Wallet  |                 |  Client  |                       |   Authorization   |                      |  Verifier  |
|          |                 |          |                       |      Server       |                      | (Internal/ |
|          |                 |          |                       |                   |                      |  External) |
|          |                 |          |  (A) Authorization    |+-----------------+|                      |            |
|          |                 |          |  Challenge Request    |+-----------------+| (A1) Create          |            |
|          |                 |          |---------------------->||  Authorization  ||     Presentation     |            |
|          |                 |          |                       ||   Challenge     ||     Request          |            |
|          |                 |          |          .            ||    Endpoint     ||--------------------->|            |
|          |                 |          |          .            ||                 ||                      |            |
|          |                 |          |          .            ||                 ||<---------------------|            |
|          |                 |          |                       ||                 || (A2) Presentation    |            |
|          |                 |          |<----------------------||                 ||     Request          |            |
|          |                 |          | (B) Authorization     ||                 ||                      |            |
|          |                 |          | Error Response        ||                 ||                      |            |
|          |                 |          | (Presentation Req.)   ||                 ||                      |            |
|          |<----------------|          |                       ||                 ||                      |            |
|          |(C) Presentation |          |          .            ||                 ||                      |            |
|          |    Request      |          |          .            ||                 ||                      |            |
|          | (response_mode) |          |          .            ||                 ||                      |            |
|          |                 |          |                       ||                 ||                      |            |
|        +--OPT: SD / CD + 'direct_post'---------------------------------------------------------------------+          |
|        | |(C1) VP Token    |          |                       ||                 ||                      | |          |
|        | |---------------------------------------------------------------------------------------------->| |          |
|        | |                 |          |                       ||                 ||                      | |          |
|        +---------------------------------------------------------------------------------------------------+          |
|          |                 |          |                       ||                 ||                      |            |
|        +--OPT: SD + 'direct_post'---------------------------------------------------------------------------------------+
|        | |(C2) redirect_uri + response_code                   ||                 ||                      |+----------+| |
|        | |<----------------------------------------------------------------------------------------------|| Response || |
|        | |                 |          |                       ||                 ||                      || URI      || |
|        | |                 |          |                       ||                 ||                      |+----------+| |
|        +----------------------------------------------------------------------------------------------------------------+
|          |                 |          |                       ||                 ||                      |            |
|        +--OPT: SD / DC API---+        |                       ||                 ||                      |            |
|        | |---------------->| |        |                       ||                 ||                      |            |
|        | |(D) Presentation | |        |                       ||                 ||                      |            |
|        | |    Response w/  | |        |                       ||                 ||                      |            |
|        | |    response code| |        |                       ||                 ||                      |            |
|        | |    or VP Token  | |        |                       ||                 ||                      |            |
|        | |                 | |        |                       ||                 ||                      |            |
|        +---------------------+        |                       ||                 ||                      |            |
|          |                 |          | (E) Authorization     ||                 ||                      |            |
|          |                 |          | Challenge Request     ||                 || (E1) Validate        |            |
|          |                 |          | (opt. w/ Pres. Res)   ||                 ||      Presentation    |            |
|          |                 |          |---------------------->||                 ||      Response        |            |
|          |                 |          |                       ||                 ||--------------------->|            |
|          |                 |          |                       ||                 ||                      |            |
|          |                 |          |                       ||                 ||<---------------------|            |
|          |                 |          |<----------------------||                 || (E2) Presentation    |            |
|          |                 |          | (F) Authorization     |+-----------------+|     Data             |            |
|          |                 |          |     Code Response     |                   |                      |            |
|          |                 |          |                       |                   |                      |            |
|          |                 |          | (G) Token             |                   |                      |            |
|          |                 |          |     Request           |+-----------------+|                      |            |
|          |                 |          |---------------------->||      Token      ||                      |            |
|          |                 |          |                       ||     Endpoint    ||                      |            |
|          |                 |          |<----------------------||                 ||                      |            |
|          |                 |          | (H) Access Token      |+-----------------+|                      |            |
|          |                 |          |                       |                   |                      |            |
+----------+                 +----------+                       +-------------------+
~~~

(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint specifying the challenge method (`challenge_method`) to request an OID4VP Authorization Request.

(A1–A2) The Authorization Server determines the challenge method and generates an OID4VP Authorization Request (Presentation Request) with `response_mode` in coordination with the Verifier component.

(B) The Authorization Server and responds with the Authorization Error Response containing the OID4VP Authorization Request from (A1-A2), the respective `response_mode` and `auth_session`.

(C) The Client invokes the Wallet via a link, QR Code, or DC API, depending on the OID4VP Authorization Request received in (B).

(C1) Optionally, for `response_mode:direct_post`, the Wallet submits the VP Token directly to the Verifier’s Response Endpoint.

(C2) Optionally, for `response_mode:direct_post` in same-device flows, the Verifier’s Endpoint responds to the Wallet with `redirect_uri` and `response_code`.

(D) Optionally, for DC API flows or same-device flows, the Client receives the OID4VP Authorization Response (Presentation Response) from the Wallet, which MAY include a VP Token or presentation artifacts from (C2). Otherwise, the Client SHOULD skip this step and continue with (E).

(E) The Client submits a subsequent Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint to request the Authorization Code. If the Client retrieved the OID4VP Authorization Response in (D), it includes it; if not, it polls using the `auth_session`.

(E1–E2) The Authorization Server validates the Presentation Response with the Verifier component.

(F) After a successful validation in (E1-E2) and performing authorization checks, the Authorization Server responds with an Authorization Challenge Response including the Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 tokens.

# Acknowledgments
{:numbered="false"}

The authors would like to thank all colleagues and contributors who provided valuable feedback and support during the development of this document. Special thanks to Henrik Kroll and Yaron Zehavi for their insightful contributions.
