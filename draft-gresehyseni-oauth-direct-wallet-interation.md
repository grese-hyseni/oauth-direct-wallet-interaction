---
title: "OAuth 2.0 direct Wallet interaction"
abbrev: "OAuth direct Wallet interaction"
category: info

docname: draft-gresehyseni-oauth-direct-Wallet-interation-latest
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
  github: "grese-hyseni/oauth-direct-Wallet-interaction"
  latest: "https://grese-hyseni.github.io/oauth-direct-Wallet-interaction/draft-gresehyseni-oauth-direct-Wallet-interation.html"

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

This document defines a protocol for Authorization Servers to request Presentation of Credentials, as part of authentication and authorization processes.

--- middle

# Introduction

This specification builds on the First-Party Apps (FiPA) framework {{I-D.ietf-oauth-first-party-apps}} to enable Authorization Servers to request Presentation of Credentials as part of the authentication and authorization processes in OAuth flows.

OAuth for First-Party Apps (FiPA) is used as a base protocol as it extends the OAuth 2.0 Authorization Framework {{RFC6749}} with a new endpoint to support applications that want to control the process of obtaining authorization from the user using a native experience. This approach facilitates a smooth user experience when leveraging digital credentials from a Wallet app.

The scope of this document extends to Third-Party Apps as defined in {{RFC6749}}, maintaining the core model established by FiPA {{I-D.ietf-oauth-first-party-apps}}.

This specification supports use cases such as user login, transaction approval, and credential presentation during issuance, based on {{OpenID4VP}} and {{OpenID4VCI}}.

The integration of OpenID4VP Verifier functionality within an Authorization Server, as well as the internal communication between an Authorization Server and a Verifier, are outside the scope of this specification.

Browser-based Wallet interactions are not the focus of this document, as such flows are expected to function effectively in web environments without extensions to existing authorization models, such as the Authorization Code Flow defined in section 4.1 of {{RFC6749}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol

In the model introduced in this document, the Client acts as the orchestrator of Wallet interactions, while the Authorization Server remains responsible for authorization decisions and for integrating Verifier capabilities.

The mechanisms defined in this document support:

* Same-device Wallet interactions;
* Cross-device Wallet interactions;
* Deployments using OpenID4VP {{OpenID4VP}}; and
* Deployments using platform Digital Credentials APIs {{DC.API}} or equivalent native SDKs.

The Verifier is a separate logical component with its own endpoints and functionality, as defined in OpenID4VP {{OpenID4VP}}.


## Same Device Flow {#same-device-flow}

The figure below illustrates a high-level use case where the Authorization Server (AS) requests a Verifiable Presentation from the Wallet as part of the authorization process, and the Wallet is on the same device as the Client app. This flow does not represent OID4VP over DC API; see section {{oid4vp-over-dc-api}}.

~~~ ascii-art
+----------+                 +----------+                       +-------------------+                      +------------+
|  Wallet  |                 |  Client  |                       |   Authorization   |                      |  Verifier  |
|          |                 |          |                       |      Server       |                      |            |
|          |                 |          |  (A) Authorization    |+-----------------+|                      |            |
|          |                 |          |  Challenge Request    |+-----------------+| (A) Create           |            |
|          |                 |          |---------------------->||  Authorization  ||     Presentation     |            |
|          |                 |          |                       ||   Challenge     ||     Request          |            |
|          |                 |          |          :            ||    Endpoint     ||--------------------->|            |
|          |                 |          |                       ||                 ||                      |            |
|          |                 |          |                       ||                 ||<---------------------|            |
|          |                 |          |                       ||                 || (B) Presentation     |            |
|          |                 |          |<----------------------||                 ||     Request          |            |
|          |                 |          | (B)Authorization      ||                 ||                      |            |
|          |                 |          | Error Response        ||                 ||                      |            |
|          |                 |          | (Presentation Req.)   ||                 ||                      |            |
|          |<----------------|          |                       ||                 ||                      |            |
|          |(C) Presentation |          |          :            ||                 ||                      |            |
|          |    Request      |          |                       ||                 ||                      |            |
|          | (response_mode) |          |                       ||                 ||                      |            |
|          |                 |          |                       ||                 ||                      |            |
|          |---------------->|          | (E) Authorization     ||                 ||                      |            |
|          |(D) Presentation |          | Challenge Request     ||                 || (E) Validate         |            |
|          |    Response     |          | (Presentation Res.)   ||                 ||     Presentation     |            |
|          | w/ reponse_code |          |---------------------->||                 ||     Response         |            |
|          | or vp_token     |          |                       ||                 ||--------------------->|            |
|          |                 |          |                       ||                 ||                      |            |
|          |                 |          |                       ||                 ||<---------------------|            |
|          |                 |          |<----------------------||                 || (F) Presentation     |            |
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
+----------+                 +----------+                       +-------------------+                      +------------+
~~~

(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. The Client MUST provide authentication_method and MAY provide redirect_uri to request an OID4VP Authorization Request for a same-device flow. These parameters may be included in the initial or a subsequent Authorization Challenge Request. See Section {{authorization-challenge-request}} for details.

(B) Once the Authorization Server determines the authentication method and has the necessary data, it responds with an Authorization Error Response containing the OID4VP Authorization Request (Presentation Request) suitable for a same-device flow. See Sections {{authorization-error-response}} for details.

(C) The Client invokes the Wallet on the user’s device with the OID4VP Authorization Request, as defined in {{OID4VP}}, where the user is prompted to authenticate, authorize, and consent to share credentials.

(D) The Client receives the Presentation Response from the Wallet, which MAY include a VP Token or other presentation artifacts depending on the Presentation Request used in (C). If a `oid4vp_response_code` is present, it indicates that Wallet submitted the VP Token directly to the Verifier’s Response Endpoint (see {{oid4vp-same-device-direct-post}}).

(E) The Client forwards the Wallet response from (D) in a subsequent Authorization Challenge Request to the Authorization Server.

(F) Upon successful validation of the Presentation Response by the verifier and completion of authorization checks, the Authorization Server issues an Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 or OpenID Connect tokens.


## Cross Device Flow {#cross-device-flow}

The figure below illustrates a high-level use case where the Authorization Server (AS) requests a Verifiable Presentation from the Wallet as part of the authorization process, and the Wallet is on a different device from the Client app. This flow does not represent OID4VP over DC API; see section {{oid4vp-over-dc-api}}.

~~~ ascii-art
+----------+                   +----------+                        +-------------------+                   +------------+
|  Wallet  |                   |  Client  |                       |   Authorization   |                    |  Verifier  |
|          |                   |          |                       |      Server       |                    |            |
|          |                   |          | (A) Authorization     |+-----------------+|                    |            |
|          |                   |          |     Challenge Request |+-----------------+| (A) Create         |            |
|          |                   |          |---------------------->||  Authorization  ||     Presentation   |            |
|          |                   |          |                       ||   Challenge     ||     Request        |            |
|          |                   |          |          :            ||    Endpoint     ||------------------> |            |
|          |                   |          |                       ||                 ||                    |            |
|          |                   |          |                       ||                 ||<------------------ |            |
|          |                   |          |                       ||                 || (B) Presentation   |            |
|          |                   |          |<----------------------||                 ||     Request        |            |
|          |                   |          | (B) Authorization     ||                 ||                    |            |
|          |                   |          |     Error Respons     ||                 ||                    |            |
|          |                   |          |    (Presentation Req.)||                 ||                    |            |
|          |<------------------|          |                       ||                 ||                    |            |
|          |(C) Presentation   |          |          :            ||                 ||                    |            |
|          |    Request        |          |                       ||                 ||                    |            |
|          |                   |          |                       ||                 ||                    |            |
|          |(D) Presentation   |          |                       ||                 ||                    |            |
|          |    Response       |          |                       ||                 ||                    |            |
|          |    w/ VP Token    |          |                       ||                 ||                    |            |
|          |---------------------------------------------------------------------------------------------->|            |
|          |                   |          |                       ||                 ||                    |            |
|          |                   |          |                       ||                 ||(D) Get Presentation|            |
|          |                   |          |                       ||                 ||    Data            |            |
|          |                   |          |                       ||                 ||------------------->|            |
|          |                   |          |                       ||                 ||                    |            |
|          |                   |          |                       ||                 ||<-------------------|            |
|          |                   |          |<----------------------||                 ||(D) Presentation    |            |
|          |                   |          | (F) Authorization     |+-----------------+|     Data           |            |
|          |                   |          |     Code Response     |                   |                    |            |
|          |                   |          |                       |                   |                    |            |
|          |                   |          | (G) Token             |                   |                    |            |
|          |                   |          |     Request           |+-----------------+|                    |            |
|          |                   |          |---------------------->||      Token      ||                    |            |
|          |                   |          |                       ||     Endpoint    ||                    |            |
|          |                   |          |<----------------------||                 ||                    |            |
|          |                   |          | (H) Access Token      |+-----------------+|                    |            |
|          |                   |          |                       |                   |                    |            |
+----------+                   +----------+                       +-------------------+                    +------------+
~~~

(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. The Client MUST provide `authentication_method` and omit the `redirect_uri` to request an OID4VP Authorization Request for a cross-device flow. These parameters may be included in the initial or a subsequent Authorization Challenge Request. See Section {{authorization-challenge-request}} for details.

(B) Same as for the {{same-device-flow}}.

(C) The Client generates a QR Code containing the OID4VP Authorization Request for the user to invoke the Wallet on another device.

(D) The Wallet sends the Presentation Response containing the VP Token directly to the Verifier's Response Endpoint.

(F) Same as for the {{same-device-flow}}.

(G) Same as for the {{same-device-flow}}.

(H) Same as for the {{same-device-flow}}.

# Authorization Challenge Request {#authorization-challenge-request}

This document extends the FiPA {{I-D.ietf-oauth-first-party-apps}} Authorization Challenge Request by adding the following parameters:

"authentication_method":
:    OPTIONAL.  A string indicating the authentication method requested by the Client, e.g., a user-preselected method on the client side. The Client MUST include this to request an OID4VP Authorization Request from the Authorization Server. This document defines the value `oid4vp` and `oid4vp_dc_api`. Parties using other values must mutually agree on their meanings, which may be context-specific.

"oid4vp_vp_token":
:    OPTIONAL.  The VP Token value, as defined in section 8 of {{OpenID4VP}}, returned from the Wallet as part of the Presentation Response (e.g., in DC API or `response_mode=fragment` flows).

"oid4vp_response_code":
:    OPTIONAL.  The `response_code` value, as defined in section 13.3 of {{OpenID4VP}}, returned from the Wallet as part of the Presentation Response (for non-DC API, same-device, response_mode=direct_post flows).

Additionally, the Client MAY add the following parameters to the query component of the Authorization Challenge Endpoint URI using the "application/x-www-form-urlencoded" format:

"redirect_uri":
:    OPTIONAL.  The OAuth redirect_uri defined in {{RFC6749}}. This parameter MUST be included in same-device flows along with `authentication_method` and represents the Client URI to which the Wallet redirects the user in same-device flows. If the Authorization Server receives `authentication_method` without `redirect_uri`, it SHOULD treat the request as a cross-device flow. This parameter SHOULD NOT be sent when `authentication_method=oid4vp_dc_api`; if sent, the Authorization Server MUST ignore it.

"authorization_details":
:    OPTIONAL. A JSON object as defined in {{RFC9396}}, representing OAuth authorization details. See {{oid4vp-with-rar}} for usage details.


## Initial Authorization Challenge Request {#init-authorization-challenge-request}
The initial Authorization Challenge Request includes only the information available at that time. For example, if the Client has no prior knowledge of preferred user login methods, it can send an Authorization Challenge Request without specifying `authentication_method` or any related parameters. See {{example-flows}}. Conversely, if the Client has such information beforehand, it can include it in the initial Authorization Challenge Request.

## Subsequest uthorization Challenge Requests {#subs-authorization-challenge-request}
The Authorization Server MAY respond with an Authorization Error Response containing `oid4vp` or/and `oid4vp_dc_api` as one of the supported authentication methods (`authentication_methods_supported`). In such cases, the Client SHOULD provide additional parameters such as `authentication_method` and/or `redirect_uri` in a subsequent Authorization Challenge Request. See {{example-flows}}

# Authorization Error Response {#authorization-error-response}
This document extends FiPA's {{I-D.ietf-oauth-first-party-apps}} Authorization Error Response by adding the following attributes, used when the error code "insufficient_authorization" is returned:

"authentication_methods_supported":
:    OPTIONAL. A JSON array of strings identifying the authentication methods supported by the Authorization Server. This document defines the value `oid4vp` and `oid4vp_dc_api` for OID4VP and OID4VP over DC API respectively. Parties using other values must mutually agree on their meanings, which may be context-specific.

"authentication_method":
:    OPTIONAL.  A string indicating the authentication method requested by the Authorization Server. The value MUST be one of those listed in `authentication_methods_supported`. This parameter MUST be present when the Authorization Server lacks sufficient data to return the actual `oid4vp_authorization_request`.

"oid4vp_authorization_request":
:   OPTIONAL. An OpenID4VP {{OpenID4VP}} request URL.

"status":
:    OPTIONAL.  Indicates the status of the ongoing operation. This document uses the value "pending".

# Example flows {#example-flows}

The Client MAY send an Authorization Challenge Request indicating whether it supports same-device or cross-device flows, and whether it supports the DC API. For same-device flows, the Authorization Server decides whether the VP Token should be returned to the Client for forwarding (using `response_mode=fragment`), or submitted directly from the Wallet to the Verifier (using response_mode=direct_post). For details on response_mode, see {{OpenID4VP}}.

This section provides non-normative examples illustrating how this specification supports various deployment models.

## OpenID4VP Same-Device Flow {#oid4vp-same-device}

In this example, the Client includes a `redirect_uri` and `authentication_method=oid4vp` to indicate a same-device flow. The Authorization Server generates an OID4VP Authorization Request containing `response_mode=fragment`, meaning the Wallet returns the VP Token directly to the Client. The Client then forwards the VP Token via the `oid4vp_vp_token` parameter in an Authorization Challenge Request sent to the Authorization Server.

Example: Initial Authorization Challenge Request (no preferred authentication):

~~~ http-message
   POST /authorize-challenge HTTP/1.1
   Host: server.example.com
   Content-Type: application/x-www-form-urlencoded

   scope=openid
   &client_id=bb16c14c73415
~~~

Example: Authorization Error Response indicating supported authentication methods

~~~ http-message
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store

{
   "error": "insufficient_authorization",
   "auth_session": "uY29tL2F1dGhlbnRpY",
   "authentication_methods_supported": ['oid4vp','oid4vp_dc_api']
}
~~~

Example: Subsequent Authorization Challenge Request requesting OID4VP Authorization Request (non-DC API same-device flow)

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=uY29tL2F1dGhlbnRpY
&authentication_method=oid4vp
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
~~~

Example: Authorization Error Response containing OID4VP Authorization Request
(Whitespace and line breaks added for readability only)

~~~ http-message
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store

{
   "error": "insufficient_authorization",
   "auth_session": "uY29tL2F1dGhlbnRpY",
   "authentication_method": ["oid4vp"],
   "oid4vp_authorization_request": "https://Wallet.example.org/universal-link?
   response_type=vp_token
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
&vp_token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMTIzIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNjI3MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
~~~

Example: Successful Authorization Challenge Response returning the authorization code

~~~ http-message
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
   "authorization_code": "c2VjdXJlL2F1dGgvY29kZQ=="
}
~~~

## OpenID4VP Same-Device Flow using `direct_post` {#oid4vp-same-device-direct-post}

In this example, the Client includes a `redirect_uri` to indicate a same-device flow. The Authorization Server issues an OID4VP Authorization Request with `response_mode=direct_post`, instructing the Wallet to submit the VP Token directly to the Verifier’s Response Endpoint, per section 8.2 of {{OpenID4VP}}. This avoids sending the VP Token through the Client when it is not the intended audience.

The Authorization Server MUST inform the Verifier that this is a same-device flow, so the Verifier Response Endpoint knows to return a `redirect_uri` with a `response_code` only in this scenario. The Client then uses this `response_code` to complete the authorization

The Client then sends a subsequent Authorization Challenge Request including the `oid4vp_response_code`:

~~~ http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=bXlzZXNzaW9uMTIzNDU2
&oid4vp_response_code=091535f699ea575c7937fa5f0f454aee
~~~

## OpenID4VP over DC API {#oid4vp-over-dc-api}

The Client indicates support for the DC API by including `authentication_method=oid4vp_dc_api` in the Authorization Challenge Request. The Authorization Server responds with a Presentation Request formatted for OpenID4VP over DC API.

In this model, the vp_token is always returned to the Client via the DC API. The Client then forwards the received vp_token to the Authorization Server in a subsequent Authorization Challenge Request to complete the authorization process.

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

Upon receiving this request, the Authorization Server MAY use the contents of authorization_details to populate the transaction_data in the OID4VP Authorization Request. This enables the Wallet to present the user with the relevant information and prompt for explicit consent for the specified action(s), as defined in {{OpenID4VP}}.

# Security Considerations

Implementations SHOULD adhere to the security and privacy considerations specified in all referenced specifications used in this document, including but not limited to OAuth 2.1 {{RFC9126}}, OpenID Connect {{OIDC-Core}}, OpenID4VP {{OpenID4VP}}, and Rich Authorization Requests (RAR) {{RFC9396}}.

In deployments where Authorization Requests and Presentation Responses transit through third-party applications or intermediaries, additional risks to integrity, confidentiality, and privacy arise due to potential exposure of sensitive data.

To mitigate these risks:

* Authorization Requests are RECOMMENDED to be signed by the Authorization Server (acting as Verifier) to ensure authenticity and integrity.
* Presentation Responses are RECOMMENDED to be signed and encrypted to protect confidentiality and privacy, especially in flows like OpenID4VP over DC API where responses pass through third-party components.

Consistent with FiPA guidance, when third-party Clients are involved, Authorization Servers MUST NOT require Clients to handle sensitive user credential data directly. Instead, the `response_mode=direct_post` flow SHOULD be used, allowing the Wallet to submit the Verifiable Presentation directly to the Verifier’s Response Endpoint. This minimizes exposure of sensitive data to third-party Clients and reduces associated risks.

## Signed Authorization Requests

To mitigate tampering and unauthorized disclosure risks, it is RECOMMENDED that the Authorization Server, acting as the OpenID4VP Verifier, signs Authorization Requests to ensure authenticity and integrity.

## Signed and Encrypted Presentation Responses

To protect the integrity, confidentiality, and privacy of Presentation Responses, it is RECOMMENDED that the Authorization Server require signed Presentation Responses and support encrypted responses intended for the Authorization Server.

In particular, this is relevant for deployment models such as OpenID4VP over DC API, where Presentation Responses may transit through a third-party application before reaching the Authorization Server.


# IANA Considerations

OAuth Parameters Registration

IANA has (TBD) registered the following values in the IANA "OAuth Parameters" registry of [IANA.oauth-parameters] established by [RFC6749].

   *Parameter name*: authentication_method

   *Parameter usage location*: authorization request

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: vp_token

   *Parameter usage location*: authorization request

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: vp_response_code

   *Parameter usage location*: authorization request

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: oid4vp_authorization_request

   *Parameter usage location*: authorization response

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: authentication_methods_supported

   *Parameter usage location*: authorization response

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


   *Parameter name*: authentication_method

   *Parameter usage location*: authorization request, authorization response

   *Change Controller*: IETF

   *Specification Document*: Section {#authorization-challenge-request} of this specification


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank all colleagues and contributors who provided valuable feedback and support during the development of this document. Special thanks to Henrik Kroll and Yaron Zehavi for their insightful contributions.
