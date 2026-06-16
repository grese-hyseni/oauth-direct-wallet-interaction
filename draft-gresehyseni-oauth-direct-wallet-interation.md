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

This document defines a protocol for Authorization Servers to request Verifiable Presentations as part of authentication and authorization processes.

--- middle

# Introduction

This specification provides a mechanism on top of First-Party Apps (FiPA) {{I-D.ietf-oauth-first-party-apps}} that enables Authorization Servers to request Verifiable Presentations as part of the authentication and authorization proccess.

OAuth for First-Party Apps (FiPA) is used as a base protocol as it extends the OAuth 2.0 Authorization Framework {{RFC6749}} with a new endpoint to support applications that want to control the process of obtaining authorization from the user using a native experience, which serves a smooth user experience when using digital credentials from a wallet app.

The scope of this document is not limited to First-Party Apps but extends to Third-Party Apps as defined by {{RFC6749}} while keeping the rest of the base model as defined in First-Party Apps (FiPA) {{I-D.ietf-oauth-first-party-apps}}.

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

~~~ ascii-art
+----------+                 +----------+                       +-------------------+                      +------------+
|  Wallet  |                 |  Third   |                       |   Authorization   |                      |  Verifier  |
|          |                 |  Party-  |                       |      Server       |                      |            |
|          |                 |  Client  |  (A) Authorization    |+-----------------+|                      |            |
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
|          | (reponse_code/  |          |---------------------->||                 ||     Response         |            |
|          |  vp_token)      |          |                       ||                 ||--------------------->|            |
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

(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. The client MUST provide `authentication_method` and MAY provide `redirect_uri` to request an OID4VP Authorization Request for a same-device flow. The Client MAY provide these parameters in the initial or in a subsequent Authorization Challenge Request. See Sections {{authorization-challenge-request}} for details.

(B) Once the Authorization Server determines the authentication method and has the necessary data, it responds with an Authorization Error Response containing the OID4VP Authorization Request (Presentation Request) suitable for a same-device flow. See Sections {{authorization-error-response}} for details.

(C) The Client invokes the wallet on the user’s device with the OID4VP Authorization Request, where user is promted to authenticate, authorize, and consent to share credentials.

(D) The Client receives the Presentation Response from the wallet, which MAY include a VP Token or other presentation artifacts depending on the Presentation Request used in (C). If a `oid4vp_response_code` is present, it indicates that wallet submitted the VP Token directly to the Verifier’s Response Endpoint (see {{oid4vp-same-device-direct-post}}).

(E) The Client forwards the wallet response from (D) in a subsequent Authorization Challenge Request to the Authorization Server.

(F) Upon successful validation of the Presentation Response by the verifier and completion of authorization checks, the Authorization Server issues an Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 or OpenID Connect tokens.


## Cross Device Flow {#cross-device-flow}

The following figure illustrates the High Level use cases in which the Authorization Server (AS) requests a Verifiable Presentation from the Wallet as part of the authorization process and the wallet is on a different device from the client app. This does not represent the OID4VP over DC API, see section {{oid4vp-over-dc-api}}.

~~~ ascii-art
+----------+                   +----------+                        +-------------------+                   +------------+
|  Wallet  |                   |  Third   |                       |   Authorization   |                    |  Verifier  |
|          |                   |  Party-  |                       |      Server       |                    |            |
|          |                   |  Client  | (A) Authorization     |+-----------------+|                    |            |
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

(A) The Client sends an Authorization Challenge Request to the Authorization Server’s Authorization Challenge Endpoint. The client MUST provide `authentication_method` and ommit the `redirect_uri` to request an OID4VP Authorization Request for a cross-device flow. The Client MAY provide these parameters in the initial or in a subsequent Authorization Challenge Request. See Sections {{authorization-challenge-request}} for details.

(B) Once the Authorization Server determines the authentication method and has the necessary data, it responds with an Authorization Error Response containing the OID4VP Authorization Request (Presentation Request) suitable for a cross-device flow. See Section {{authorization-error-response}} for details.

(C) The Client generated the QR Code for the user to invoke the wallet on another device which contains the OID4VP Authorization Request.

(D) The wallet sends the Presentation Response which contains the VP Token directly to the Verifier's Response Endpoint.

(F) Upon successful validation of the Presentation Response by the verifier and completion of authorization checks, the Authorization Server issues an Authorization Code.

(G) The Client sends a Token Request to the Authorization Server’s Token Endpoint to exchange the Authorization Code.

(H) The Authorization Server validates the Token Request and responds with an Access Token and optionally additional OAuth 2.0 or OpenID Connect tokens.

# Authorization Challenge Request {#authorization-challenge-request}

This document extends the FiPA {{I-D.ietf-oauth-first-party-apps}} Authorization Challenge Request by adding the following parameters:

"authentication_method":
:    OPTIONAL.  A string indicating the authentication method requested by the Client, e.g., a user-preselected method on the client side. Client MUST include this in order to request a OID4VP Authorization Request from the Authorization Server. This document defines the value `oid4vp` and `oid4vp_dc_api`. Parties using any other values must mutually agree on the values meanings, which may be context-specific.

"oid4vp_vp_token":
:    OPTIONAL.  A value the VP Token, as defined in section 8 of {{OpenID4VP}}, returned from the wallet as part of the Presentation Response (e.g., in DC API or `response_mode=fragment` flows).

"oid4vp_response_code":
:    OPTIONAL.  A value of the `response_code`, as defined in section 13.3. of {{OpenID4VP}}, returned from the wallet as part of the Presentation Response (for non-DC API, same-device, response_mode=direct_post flows).

Additionally, the client MAY add to the request URI the following parameters to the query component of the authorization challenge endpoint URI using the "application/x-www-form-urlencoded" format:

"redirect_uri":
:    OPTIONAL.  The OAuth redirect_uri defined in {{RFC6749}}. This parameter MUST be included in same-device flows along with `authentication_method` and represents the client URI to which the wallet redirects the user for the same-device flows. If the Authorization Server receives `authentication_method` without `redirect_uri`, it SHOULD treat the request as a request for cross-device flow. This parameter SHOULD NOT be sent when `authentication_method=oid4vp_dc_api`. If it is sent, the Authorization Server MUST ignore it.

"authorization_details":
:    OPTIONAL. A JSON object as defined in {{RFC9396}}, representing OAuth authorization details. See {{oid4vp-with-rar}} to understand its usage.


## Initial Authorization Challenge Request {#init-authorization-challenge-request}
The initial Authorization Challenge Request includes only the information available at that time. For example, if the client has no prior information on preferred user login methods, it can send a Authorization Challenge Request without specifying the `authentication_method` or any additional data relevant for it. See {#example-flows}. Similarly, in another time, if/when the client knows pre-flow such information, it can already send everything in the first authrization challenge request.

## Subsequest uthorization Challenge Requests {#subs-authorization-challenge-request}
The Authorization Server MAY respond with an Authorization Error Response containing `oid4vp` or/and `oid4vp_dc_api` as one of the `authentication_methods_supported`. In such case, the Client should provide in a subsequent Authorization Challenge Request additional parameters such as `authenticaion_method` and/or `redirect_uri`. See {#example-flows}.

# Authorization Error Response {#authorization-error-response}

This document extends FiPA's {{I-D.ietf-oauth-first-party-apps}} Authorization Error Response by adding the following attributes, used when the error code "insufficient_authorization" is returned:

"authentication_methods_supported":
:    OPTIONAL. A JSON array of strings identifying the authentication methods supported by the Authorization Server. support for OID4VP and OID4VP over DC API, respectively.  This document defines the value `oid4vp` and `oid4vp_dc_api`. Parties using any other values must mutually agree on the values meanings, which may be context-specific.

"authentication_method":
:    OPTIONAL.  A string indicating the authentication method requested by the Authorization Server. The value MUST be one of those listed in `authentication_methods_supported`. This parameter MUST be present when the Authorization Server lacks sufficient data to return the actual `oid4vp_authorization_request`.

"oid4vp_authorization_request":
:   OPTIONAL. An OpenID4VP {{OpenID4VP}} request URL.

"status":
:    OPTIONAL.  Indicates the status of the ongoing operation. This document uses the value "pending".

# Example flows {#example-flows}

The Client MAY send an Authorization Challenge Request using one of the following mechanisms:

* Including a `redirect_uri` + `response_mode = fragment` parameter to indicate a same-device flow.
* Omitting the `redirect_uri` + `response_mode = direct_post` parameter to indicate a cross-device flow.
* Including a `redirect_uri` + `response_mode = direct_post` parameter to indicate a same-device flow.
* Including `type=dc_api` to indicate that the Presentation Request SHOULD be conformant with OpenID4VP over DC API.

This section provides non-normative examples illustrating how this specification supports specific deployment models.

## OpenID4VP Same-Device Flow {#oid4vp-same-device}

In a same-device flow with `response_mode=fragment`, the Wallet returns the `vp_token` directly to the Client. The Client then includes the `vp_token` in an Authorization Challenge Request sent to the Authorization Server.

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
   "authentication_method": ['oid4vp'],
   "oid4vp_authorization_request": "https://wallet.example.org/universal-link?
   response_type=vp_token
   &client_id=redirect_uri%3Ahttps%3A%2F%2Fclient.example.org%2Fcb
   &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
   &dcql_query=...
   &nonce=n-0S6_WzA2Mj
   &client_metadata=%7B%22vp_formats_supported%22%3A%7B%22dc%2Bsd-jwt%22%3A%7B%22sd-jwt_alg_values%22%3A%20%5B%22ES256%22%5D%2C%22kb-jwt_alg_values%22%3A%20%5B%22ES256%22%5D%7D%7D%7D"
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

In a same-device flow where the OID4VP Authorization Request `response_mode` is set to `direct_post`, the Wallet submits the VP Token directly to the Verifier Response Endpoint as depicted on section 8.2 of {{OpenID4VP}}. This is the recommended flow for our scenario where the Client isn't the Verifier, and it might be a third-party application, and such flow avoids having to send a VP Token via the client when the client isn't its dedicated audience.

In a same-device flow with `response_mode=direct_post`, the Wallet submits the vp_token directly to the Authorization Server provided `response_uri`, which serves as the Verifier Response Endpoint as described in Section 8.2 of {{OpenID4VP}}. This flow is recommended when the Client is not the Verifier, such as when it is a third-party app, because it avoids sending the `vp_token` through the Client, which is not the intended audience for the token.

In this deployment model, the Authorization Server MUST recognize that the flow is same-device, as the Verifier Response Endpoint is expected to return a `redirect_uri` containing a `response_code` only for same-device flows. This `response_code` enables the Client to resume and complete the authorization flow.

The Client then sends a subsequent Authorization Challenge Request including the response_code to finalize the authorization process, as illustrated in the example below:

~~~ http-message
   POST /authorize-challenge HTTP/1.1
   Host: server.example.com
   Content-Type: application/x-www-form-urlencoded

   auth_session="bXlzZXNzaW9uMTIzNDU2"
   &oid4vp_response_code="091535f699ea575c7937fa5f0f454aee"
~~~

## OpenID4VP over DC API {#oid4vp-over-dc-api}

When the Client supports the DC API, it indicates this capability to the Authorization Server in the Authorization Challenge Request via the appropriate parameter (e.g., authentication_method=oid4vp_dc_api). The Authorization Server responds with a Presentation Request formatted for OpenID4VP over DC API processing.

In this deployment model, the `vp_token` is returned always returned to the Client via the DC API. The Client then forwards the received `vp_token` to the Authorization Server in a subsequent Authorization Challenge Request to complete the authorization process.


## OpenID4VP with RAR {#oid4vp-with-rar}

A useful use case is when the user is already authenticated but needs to provide explicit approval using their wallet.

In such a case, the Client MUST include the `authorization_details` parameter as defined in {{RFC9396}} within the Authorization Challenge Request. This allows the Authorization Server to convey the specific authorization requirements to the wallet, enabling the user to approve the requested action.

An non-normative example Authorization Challenge Request is shown below:

~~~  http-message
POST /authorize-challenge HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session="bXlzZXNzaW9uMTIzNDU2"
&oid4vp_response_code="091535f699ea575c7937fa5f0f454aee"
&authorization_details='{
  "type": "openid_credential",
  "actions": ["approve"],
  "locations": ["https://example.com/resource/123"]
}'
~~~

Upon receiving this request, the Authorization Server MAY use the contents of authorization_details to populate the transaction_data in the OID4VP Authorization Request. This enables the wallet to present the user with the relevant information and prompt for explicit consent for the specified action(s), as defined in {{OpenID4VP}}.

# Security Considerations

Implementations of this specification SHOULD consider the security and privacy considerations defined by {{OpenID4VP}}, in particular those described in the OpenID for Verifiable Presentations Security Considerations section.

Implementations SHOULD additionally follow OAuth 2.0 and OpenID Connect security best current practices, including recommendations related to authorization server validation, redirect URI validation, token protection, replay prevention, transaction binding, and secure handling of authorization artifacts.

In deployment models where Authorization Requests and Presentation Responses transit through a third-party application or intermediary component, additional integrity, confidentiality, and privacy risks may arise due to potential exposure of sensitive request parameters, credential data, or presentation artifacts.

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
