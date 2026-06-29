---
title: "Cross-domain Transaction Tokens"
category: info

docname: draft-liu-cross-domain-txn-token-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: OAuth Working Group

keyword:
  - Identity Chaining
  - Transaction Tokens
  - Cross-Domain

venue:
#  group: OAuth
#  type: Working Group
#  mail: oauth@ietf.org
#  arch: https://example.com/WG
  github: "NiYuan224/draft-liu-cross-domain-txn-token"
  latest: "https://NiYuan224.github.io/draft-liu-cross-domain-txn-token/draft-liu-cross-domain-txn-token.html"

author:
 -
  name: Chunchi Peter Liu
  organization: Huawei
  email: liuchunchi@huawei.com

 -
  name: Yuan Ni
  organization: Huawei
  email: niyuan1@huawei.com



normative:
  RFC2119:
  RFC8174:
  RFC6749:
  RFC7523:
  RFC8693:
  RFC6750:

informative:
  I-D.kiliram-agent-trust-auth-framework:
  I-D.ietf-oauth-transaction-tokens:
  I-D.ietf-oauth-identity-chaining:

...

--- abstract
This document describes a mechanism for Cross-Domain Transaction Tokens, which enables the safe maintenance and propagation of user identity, workload identities, and authorization context across multiple trust domains.


--- middle

# Introduction

Due to the rise of collaborative service ecosystems and AI agents, services frequently interact across domain boundaries and involve multiple service operators (e.g., enterprises, cloud providers, SaaS
platforms, etc.). As illustrated in {{?I-D.kiliram-agent-trust-auth-framework}}, workflows spanning multiple domains require workloads within each domain to perform subtasks and pass results to cooperatively accomplish an overall task. Moreover, to ensure end-to-end accountability, the entities and actions must remain auditable throughout the entire call chain.

This requires the secure preservation and propagation of user identity, workload identities and authorization context across different domains.

Transaction Token (Txn-Token) {{?I-D.ietf-oauth-transaction-tokens}} defines a short-lived, signed JWT that preserves immutable user identity, workload identity, and authorization context within a single trust domain's call chain. While this mechanism effectively preserves context, its applicability is currently limited to single-domain call chains. Conversely, Identity Chaining {{?I-D.ietf-oauth-identity-chaining}} provides a framework for cross-domain delegation but does not specify how to leverage the rich, structured context carried by Txn-Tokens (i.e., the claims of `txn`, `tctx`, `rctx`, and `req_wl` in {{?I-D.ietf-oauth-transaction-tokens}}) .

This document bridges this gap by defining a mechanism to use a Txn-Token (from domain I) as the input of Identity Chaining, in order to create another Txn-Token at the other domain II. This approach enables the propagation of workflow-related claims, i.e., claims of `txn`, `tctx`, `rctx`, and `req_wl`, across multiple trust domains, ensuring the integrity and auditability of the entire call chain.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This document uses the terms "Workload", "Trust Domain", "External Endpoint", "Call Chain", and "Transaction Token Service" (TTS) defined by {{?I-D.ietf-oauth-transaction-tokens}}, as well as "Authorization Server" (AS) defined by {{RFC6749}}. Moreover, the following terms are extended or defined in this document.

* Single-domain Transaction Token. A short-lived, signed JWT that provides immutable information about the user, workloads, and specific contextual attributes of the call within a trust domain, as defined in {{?I-D.ietf-oauth-transaction-tokens}}.

* Cross-domain Transaction Token. A Txn-Token that preserves and propagates workflow-related claims from an upstream trust domain to a downstream domain via identity chaining as defined in {{?I-D.ietf-oauth-identity-chaining}}.

* Transaction JWT Authorization Grant (Txn-JAG). A newly defined, specialized JWT acting as an OAuth 2.0 authorization grant (as defined in {{RFC7523}}), which carries the workflow-related claims, ensuring the integrity and auditability of the cross-domain call chain.

* Workflow-related Claims: `txn`, `tctx`, `rctx`, and `req_wl` claims, as defined in {{?I-D.ietf-oauth-transaction-tokens}}.

# Workflows of Cross-Domain Transaction Token {#workflow}

By combining Identity Chaining {{?I-D.ietf-oauth-identity-chaining}} and Transaction Token {{?I-D.ietf-oauth-transaction-tokens}}, this section describes two workflow modes to securely preserve and propagate workflow-related claims across different trust domains. Both modes utilize the newly defined Txn-JAG as the secure carrier between domains but differ in how the downstream domain processes it.

* Mode A: Indirect Txn-Token Exchange via Intermediate Access Token. Workload A in Trust Domain I first exchanges its local Txn-Token with its own AS for a Txn-JAG that targets the AS in Trust Domain II. Then, Workload A presents that Txn-JAG to the AS in Trust Domain II to obtain an access token, and uses the access token to invoke Endpoint B in Trust Domain II. Finally, Endpoint B exchanges the access token with the local TTS to acquire a new Txn-Token in Trust Domain II.

* Mode B: Direct Txn-Token Exchange. Workload A in Trust Domain I exchanges its local Txn-Token with its own AS for a Txn-JAG that targets the TTS in Trust Domain II. Then, Workload A presents the Txn-JAG directly to Endpoint B, and Endpoint B exchanges the Txn-JAG with the local TTS to obtain a new Txn-Token in Trust Domain II, thereby eliminating the intermediate round trips for the access token.

The following subsections focus on the logical orchestration and context propagation of these workflows. Definitions for the request and response formats are detailed in {{reqs}}.

## Mode A: Indirect Txn-Token Exchange {#modea}

This mode follows the classic identity chaining workflow {{?I-D.ietf-oauth-identity-chaining}}. The workflow is illustrated in Figure 1.

~~~
+----------+ +--------+ +---------+ +----------+ +---------+
|Workload A| |   AS   | |    AS   | |Endpoint B| |   TTS   |
|Domain I  | |Domain I| |Domain II| |Domain II | |Domain II|
+----------+ +--------+ +---------+ +----------+ +---------+
 |                 |          |      |                 |
 |(1)Token Exchange Request   |      |                 |
 |---------------->|          |      |                 |
 | <Txn-Token I>   |          |      |                 |
 |                 |          |      |                 |
 |(2)Token Exchange Response  |      |                 |
 |<----------------|          |      |                 |
 |     <Txn-JAG>   |          |      |                 |
 |                 |          |      |                 |
 |(3)Present Txn-JAG          |      |                 |
 |--------------------------->|      |                 |
 |                 |          |      |                 |
 |(4)<Access Token>           |      |                 |
 |<---------------------------|      |                 |
 |                 |          |      |                 |
 |(5)Invoke with Access Token |      |                 |
 |----------------------------|----->|                 |
 |                 |          |      |                 |
 |                 |          |      |(6)Txn-Token Request
 |                 |          |      |---------------->|
 |                 |          |      | <Access Token>  |
 |                 |          |      |                 |
 |                 |          |      |(7)Txn-Token Response
 |                 |          |      |<----------------|
 |                 |          |      | <Txn-Token II>  |

~~~

*Figure 1: Indirect Txn-Token Exchange*

(1) Workload A in Trust Domain I sends a token exchange request to the local AS to exchange the local Txn-Token for a cross-domain Txn-JAG. See {{exchangeforJAG}} for the request format.

(2) The AS in Trust Domain I validates the request, generates and responds with the Txn-JAG, transcribing the workflow-related claims from the local Txn-Token into the Txn-JAG's claims. See {{exchangeforJAG}} for the response format.

(3) Workload A in Trust Domain I presents the Txn-JAG to the AS of Trust Domain II to request an access token. See {{exchangeforAT}} for the request format.

(4) The AS of Trust Domain II validates the Txn-JAG and responds with an access token, transcribing the workflow-related claims from the Txn-JAG into the access token. See {{exchangeforAT}} for the response format.

(5) Workload A in Trust Domain I invokes Endpoint B of Trust Domain II with the received access token.

(6) Endpoint B presents the inbound access token to the TTS of Trust Domain II to request a Txn-Token. See {{exchangeforTxn}} for the request format.

(7) The TTS of Trust Domain II mints a local Txn-Token, transcribing the workflow-related claims from the access token into the new Txn-Token. See {{exchangeforTxn}} for the response format.

## Mode B: Direct Txn-Token Exchange {#modeb}
While Mode A provides a straightforward integration of Identity Chaining procedures with Transaction Tokens procedures, which does not create protocol-level changes, it introduces multiple cross-domain round trips that can significantly increase latency. As illustrated in Figure 1, steps (3) through (5) require at least three cross-domain round-trips: presenting the Txn-JAG to the downstream AS to obtain an access token, invoking the target endpoint with that token, and subsequently exchanging it for a local Txn-Token. In distributed and high-throughput environments, this operational overhead can become prohibitive.

An opportunity for optimization arises from the flexibility defined in Section 5.1 of the Transaction Token{{?I-D.ietf-oauth-transaction-tokens}}, which allows the subject_token to be "any other format that is understood by the TTS". This enables a TTS to directly accept a Txn-JAG issued by the upstream AS as the subject token in a Txn-Token request.

Thus, Mode B reduces cross-domain round trips from three to one by allowing the TTS in Trust Domain II to accept the Txn-JAG directly as the subject token in a Txn-Token request. This requires a pre-established trust relationship between the AS in Trust Domain I and the TTS in Trust Domain II. The workflow is illustrated in Figure 2.


~~~
+----------+   +--------+ +----------+     +---------+
|Workload A|   |   AS   | |Endpoint B|     |   TTS   |
| Domain I |   |Domain I| |Domain II |     |Domain II|
+----------+   +--------+ +----------+     +---------+
     |                 |        |                 |
     |(1)Token Exchange Request |                 |
     |---------------->|        |                 |
     |<Txn-Token I>    |        |                 |
     |                 |        |                 |
     |(2)Token Exchange Response|                 |
     |<----------------|        |                 |
     |<Txn-JAG>        |        |                 |
     |                 |        |                 |
     |(3)Present Txn-JAG        |                 |
     |------------------------->|                 |
     |                 |        |                 |
     |                 |        |(4)Txn-Token Request
     |                 |        |---------------->|
     |                 |        | <Txn-JAG>       |
     |                 |        |(5)Txn-Token Response
     |                 |        |<----------------|
     |                 |        | <Txn-Token II>  |

~~~

*Figure 2: Direct Txn-Token Exchange*

Steps (1) and (2) are the same as those in Mode A.

(3) Workload A in Trust Domain I presents the Txn-JAG to Endpoint B of Trust Domain II. See {{transmission}} for the transmission method.

(4) Endpoint B uses Txn-JAG as the subject token to exchange for a local Txn-Token at its local TTS.

(5) The TTS of Trust Domain II mints a local Txn-Token, transcribing the workflow-related context from the Txn-JAG into the local Txn-Token's claims.

# Requests and Responses {#reqs}
The formats of requests and responses included in {{workflow}} are detailed in this section. To avoid redundancy, all illustrative examples are provided in Appendix A.

## Mode A: Indirect Txn-Token Exchange

This section defines the requests and responses for Mode A as described in {{modea}}.

### Token Exchange for Txn-JAG  {#exchangeforJAG}
Workload A in Trust Domain I performs token exchange with the AS in Trust Domain I to obtain a Txn-JAG that can be used at the AS in Trust Domain II.

#### Txn-JAG Request
The parameters for the Txn-JAG request build upon the definitions in Section 2.3.1 of {{?I-D.ietf-oauth-identity-chaining}} and {{RFC8693}}. While claim types remain consistent with these specifications, the values of certain claims are required as follows:

**resource**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED** if audience is not set. It MUST be the URI of the AS in Trust Domain II.

**audience**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED** if resource is not set. It MUST be the well-known/logical name of the AS in Trust Domain II.

**subject_token**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** MUST be a valid local Txn-Token issued within Trust Domain I.

**subject_token_type**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** MUST be urn:ietf:params:oauth:token-type:txn_token.


#### Txn-JAG Response
The processing rules and response format defined in Sections 2.3.2 and 2.3.3 of {{?I-D.ietf-oauth-identity-chaining}} apply, with the following modifications:

* The AS in Trust Domain I SHOULD transcribe the workflow-related claims from the Txn-Token to the Txn-JAG's claims. During this transcription, The AS in Trust Domain I MAY add, remove, or change the claims. See Claims Transcription ({{trans}}).

### Cross-Domain Assertion {#exchangeforAT}

Workload A in Trust Domain I uses the Txn-JAG obtained from the AS in Trust Domain I as an assertion to request an access token from the AS in Trust Domain II.

#### Access Token Request

The parameters described in Section 2.4.1 of {{?I-D.ietf-oauth-identity-chaining}} apply here with the following requirements:

**assertion**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** The Txn-JAG returned by the AS in Trust Domain I.

#### Access Token Response
The processing rules and response formats defined in Sections 2.4.2 and 2.4.3 of {{?I-D.ietf-oauth-identity-chaining}} apply, with the following modifications:

* The AS in trust domain II SHOULD transcribe the workflow-related claims from the Txn-JAG to the access token's claims. See Claims Transcription ({{trans}}).


### Token Exchange for Txn-Token {#exchangeforTxn}

Workload A in Trust Domain I performs a token exchange with the TTS in Trust Domain II to obtain a local Txn-Token in Trust Domain II.

#### Txn-Token Request

The parameters for the Txn-Token request follow the definitions in Section 12.1 of {{?I-D.ietf-oauth-transaction-tokens}}, the following requirements apply to the subject_token and subject_token_type:

**subject_token**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** MUST be the access token issued by the AS in Trust Domain II, as obtained {{exchangeforTxn}}.

**subject_token_type**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** MUST be urn:ietf:params:oauth:token-type:access_token.

#### Txn-Token Response
The processing rules and response formats defined in Sections 12.3 and 12.4 of {{?I-D.ietf-oauth-transaction-tokens}} apply, with the following modifications:

* For subject token validation, the TTS in Trust Domain II MUST validate the access token issued by its local AS.

* The TTS in trust domain II SHOULD transcribe the workflow-related claims from the subject token to the claims of the issued Txn-Token.  See Claims Transcription ({{trans}}).

Domain II will proceed to use the obtained Txn-Token II normally, as defined in {{?I-D.ietf-oauth-transaction-tokens}}.

## Mode B: Direct Txn-Token Exchange

This section defines the requests and responses for Mode B as described in {{modeb}}.

### Token Exchange for Txn-JAG
Workload A in Trust Domain I performs token exchange with the AS in Trust Domain I to obtain a Txn-JAG that can be used with the TTS in Trust Domain II.

#### Txn-JAG Request
The request follows the same format as defined in {{exchangeforJAG}}, except for the resource or audience claim:

**resource**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** if audience is not set. It MUST be the URI of the TTS in Trust Domain II.

**audience**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** if resource is not set. It MUST be the well-known/logical name of the TTS in Trust Domain II.


#### Txn-JAG Response
The processing rules and response format are identical to those described in {{exchangeforJAG}}, with the exception that the `aud` claim in the issued Txn-JAG MUST identify the TTS of Trust Domain II.

#### Txn-JAG Transmission Methods {#transmission}

When a Txn-JAG is presented directly to an endpoint, the workload MUST include the Txn-JAG parameter in HTTP named `Txn-JAG`. This dedicated parameter avoids ambiguity with the Authorization header, which is conventionally associated with access tokens per {{RFC6750}}.

In {{RFC7523}} and {{?I-D.ietf-oauth-identity-chaining}}, the JAG is carried by the `assertion` field in HTTP, but it is used to request an access token. These drafts have not listed the usage of `assertion` field to request a Txn-Token, as described in this document. As a result, this document has designed a new HTTP parameter `Txn-JAG`. But alternatively, it could also be transmitted as a HTTP header field. The most appropriate design should subject to Working Group's discretion.

### Token Exchange for Txn-Token
Endpoint B performs a token exchange with the TTS in Trust Domain II to obtain a local Txn-Token in Trust Domain II, using the Txn-JAG as the subject_token.

#### Txn-Token Request
The parameters for the Txn-Token request follow the definitions in Section 12.1 of {{?I-D.ietf-oauth-transaction-tokens}}. The following requirements apply:

**subject_token**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.** MUST be the Txn-JAG issued by the AS in Trust Domain I, as obtained in Section 4.2.1 and presented to Endpoint B via the method in Section 4.2.1.3.

**subject_token_type**<br>
&nbsp;&nbsp;&nbsp;&nbsp;**REQUIRED.**  MUST be urn:ietf:params:oauth:token-type:jwt-bearer.

#### Txn-Token Response
The processing rules and response format are identical to those described in {{exchangeforTxn}}, with the following modification for subject token validation:

For subject token validation, the TTS in Trust Domain II MUST validate the Txn-JAG issued by the AS in Trust Domain I. This requires a pre-established trust relationship between the AS in Trust Domain I and the TTS in Trust Domain II. Such a trust relationship typically manifests as the exchange of key material.

The transcription of workflow-related claims from the subject token (the Txn-JAG) to the issued Txn-Token follows the same rules defined in {{trans}}.

## Claims Transcription {#trans}

Claims transcription across trust domains SHOULD ensure that the workflow-related claims are preserved for auditability and accountability. This builds upon the principles defined in Section 2.5 of {{?I-D.ietf-oauth-identity-chaining}}. Specific transcription rules for workflow-related claims are defined as follows:


* Preserving the `txn` claim. The `txn` claim serves as the immutable unique identifier for the cross-domain transaction. Both the AS and TTS SHOULD NOT modify or regenerate the `txn` value during transcription. It SHOULD be copied from the subject_token to the issued token to ensure auditability and accountability across different domains. To avoid collisions without a centralized namespace, the `txn` value can be generated as a high-entropy string or prefixed with a domain identifier.

* Evolving the `req_wl` claim. The `req_wl` SHOULD identify all the workloads that requested or exchanged tokens throughout the cross-domain transaction. Specifically, the AS in Trust Domain I SHOULD add the identifier of Workload A to the `req_wl` in the issued Txn-JAG. The TTS in Trust Domain II SHOULD add the identifier of Endpoint B to `req_wl` in the issued Txn-Token in Trust Domain II. This ensures that every point where claims may change is recorded, providing a trail of how the claims reached its current state.

* Data Minimization. The processing or `req_wl` may exist privacy concerns that exposing topology of Domain I. The AS in Trust Domain I MAY apply security and privacy strategies to workflow-related claims when issuing the Txn-JAG. Such measures include but not limited to Removal.

    * Removal. Claims related to completed tasks or not required by downstream trust domains COULD be removed or redacted. If certain claims are required for end-to-end auditing, Domain I MAY take proper logs before removal.

# Operational Considerations {#ops}

* AS and TTS may be operated by the same service, or not. In the former case, some procedures listed in {{workflow}} and {{reqs}} could be merged.

# Security and Privacy Considerations

* As Domain I and II may have public Internet in between, the correct and confidential passing of `tctx` and `rctx` may require encryption or masking techniques.


# IANA Considerations

This document has no IANA actions.



--- back

# Examples
Two examples are provided in Appendix A, including the protocol framing and decoded JSON payloads.

## Example Mode A: Indirect Txn-Token Exchange

### Txn-JAG Request and Response
Workload A in Trust Domain I performs token exchange with the AS in Trust Domain I (https://as.domain1.example/auth) to receive a Txn-JAG targeted at the AS in trust domain II (https://as.domain2.example/auth).

~~~
   POST /auth HTTP/1.1
   Host: as.domain1.example
   Content-Type: application/x-www-form-urlencoded

   grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Atoken-exchange
   &resource=https%3A%2F%2Fas.domain2.example%2Fauth
   &subject_token=[Encoded Txn-Token-I]
   &subject_token_type=urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Atxn_token
~~~
*Figure 3: Txn-JAG Request*

The subject_token in the prior request is the Txn-Token issued by the TTS in Trust Domain I, and the decoded JWT payload is shown here.

~~~
{
    "iat": 1777724800,
    "exp": 1777724860,
    "aud": "https://domain1.example",
    "iss": "https://tts.domain1.example",
    "txn": "97053963-771d-49cc-a4e3-20aad399c312",
    "sub": "john_doe@a.org",
    "req_wl": "apigateway.domain1.example",
    "rctx": {
        "req_ip": "69.151.72.123",
        "authn": "urn:ietf:rfc:6749"
    },
    "scope": "trade.stocks",
    "tctx": {
        "action": "BUY",
        "ticker": "MSFT",
        "quantity": "100",
        "customer_type": {
            "geo": "US",
            "level": "VIP"
        }
    }
}
~~~
*Figure 4: Txn-Token-I Payload*

The `access_token` parameter of the token exchange response contains the Txn-JAG requested by Workload A.

~~~
HTTP/1.1 200 OK
   Content-Type: application/json
   Cache-Control: no-cache, no-store

   {
     "access_token":[Encoded Txn-JAG],
     "token_type":"N_A",
     "issued_token_type":"urn:ietf:params:oauth:token-type:jwt",
     "expires_in":60
   }
~~~
*Figure 5: Txn-JAG Response*

~~~
{
  "iat": 1777724830,
  "exp": 1777728430,
  "aud": "https://as.domain2.example/auth",
  "iss": "https://as.domain1.example",
  "sub": "john_doe@a.org",
  "txn": "97053963-771d-49cc-a4e3-20aad399c312",
  "req_wl": "apigateway.domain1.example,workload_a",
  "rctx": {
    "authn": "urn:ietf:rfc:6749"
    [Encrypted_Information]
  },
  "scope": "trade.stocks",
  "tctx": {
    "action": "BUY",
    "ticker": "MSFT",
    "quantity": "100"
    [Encrypted_Information]
  }
}
~~~
*Figure 6: Txn-JAG Payload*

As shown in Figure 4 and Figure 6, the AS in Trust Domain I protects and evolves the claims during Txn-JAG issuance: for immutability, the `txn` claim is copied verbatim; for evolution, workload_a is appended to `req_wl` to record the exchange point; and for security, sensitive information like `req_ip` and `customer_type` are encrypted.

### Access Token Request and Response
Workload A presents the Txn-JAG as an assertion to the AS of Trust Domain II to request an access token.

~~~
    POST /auth HTTP/1.1
    Host: as.domain2.example
    Content-Type: application/x-www-form-urlencoded

    grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
    &assertion=[Encoded Txn-JAG]
~~~
*Figure 7: Access Token Request*

~~~
   HTTP/1.1 200 OK
   Content-Type: application/json
   Cache-Control: no-cache, no-store

   {
     "access_token":"ey...",
     "token_type":"N_A",
     "issued_token_type":"urn:ietf:params:oauth:token-type:jwt",
     "expires_in":60
   }
~~~
*Figure 8: Access Token Response*

~~~
{
  "iat": 1777724890,
  "exp": 1777728490,
  "aud": "https://endpointb.domain2.example",
  "iss": "https://as.domain2.example",
  "sub": "john.doe.123",
  "txn": "97053963-771d-49cc-a4e3-20aad399c312",
  "req_wl": "apigateway.domain1.example,workload_a",
  "rctx": {
    "authn": "urn:ietf:rfc:6749"
    [Encrypted_Information]
  },
  "scope": "trade.stocks",
  "tctx": {
    "action": "BUY",
    "ticker": "MSFT",
    "quantity": "100"
    [Encrypted_Information]
  }
}
~~~
*Figure 9: Access Token Payload*

### Txn-Token Request and Response
Upon receiving the access token, Workload A in Trust Domain II exchanges it for a local Txn-Token from its TTS. The payload of the Txn-Token in Trust Domain II is shown below.

~~~
{
    "iat": 1777724900,
    "exp": 1777724960,
    "iss": "https://tts.domain2.example",
    "aud": "https://domain2.example",
    "txn": "97053963-771d-49cc-a4e3-20aad399c312",
    "sub": "john_doe@a.org",
    "req_wl": "apigateway.domain1.example,workload_a,endpoint_b",
    "rctx": {
        "req_ip": "69.151.72.123",
        "authn": "urn:ietf:rfc:6749"
    },
    "scope": "trade.stocks",
    "tctx": {
        "action": "BUY",
        "ticker": "MSFT",
        "quantity": "100",
        "customer_type": {
            "geo": "US",
            "level": "VIP"
        }
    }
}
~~~
*Figure 10: Txn-Token-II Payload*

As can be seen from Figure 10, the TTS preserves the `txn` claim verbatim and evolves the `req_wl` claim, appending the identifier of Endpoint B.

## Example Mode B: Direct Txn-Token Exchange

### Txn-JAG Request and Response

Workload A in Trust Domain I exchanges its local Txn-Token for a Txn-JAG, where the resource is set to the TTS of Domain II.

~~~
POST /auth HTTP/1.1
Host: as.domain1.example
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&resource=https://tts.domain2.example
&subject_token=[Encoded Txn-Token-I the same in Figure 4]
&subject_token_type=urn:ietf:params:oauth:token-type:txn_token
~~~
*Figure 11: Txn-JAG Request*

The AS in Domain I transcribes the claims. In the issued Txn-JAG, the `aud` is set to the Domain II TTS.

~~~
{
  "iat": 1777724830,
  "exp": 1777728430,
  "aud": "https://tts.domain2.example",
  "iss": "https://as.domain1.example",
  "sub": "john_doe@a.org",
  "txn": "97053963-771d-49cc-a4e3-20aad399c312",
  "req_wl": "workload_a",
  "rctx": {
        "req_ip": "69.151.72.123",
        "authn": "urn:ietf:rfc:6749"
  },
  "scope": "trade.stocks",
  "tctx": {
        "action": "BUY",
        "ticker": "MSFT",
        "quantity": "100",
        "customer_type": {
            "geo": "US",
            "level": "VIP"
        }
    }
}
~~~
*Figure 12: Txn-JAG Payload*

As defined in {{trans}}, the AS in Trust Domain I applies an removal strategy to the `req_wl` claim. The internal path preceding workload_a is removed to protect the internal topology of Domain I.

### Txn-Token Request and Response

Workload A presents the Txn-JAG directly to Endpoint B. Endpoint B then uses this Txn-JAG as the `subject_token` to request a local Txn-Token from the TTS in Trust Domain II.

~~~
POST /token HTTP/1.1
Host: tts.domain2.example
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&requested_token_type=urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Atxn-token
&subject_token=[Encoded Txn-JAG]
&scope=trade.stocks
&subject_token_type=urn:ietf:params:oauth:token-type:jwt-bearer
~~~
*Figure 13: Txn-Token Request*

The TTS validates the cross-domain Txn-JAG based on the pre-established trust relationship with AS in Trust Domain I. It transcribes the claims and evolves the `req_wl` to include Endpoint B.

~~~
{
  "iat": 1777724900,
  "exp": 1777724960,
  "iss": "https://tts.domain2.example",
  "aud": "https://domain2.example",
  "txn": "97053963-771d-49cc-a4e3-20aad399c312",
  "sub": "john_doe@a.org",
  "req_wl": "workload_a,endpoint_b",
  "rctx": {
        "req_ip": "69.151.72.123",
        "authn": "urn:ietf:rfc:6749"
  },
  "scope": "trade.stocks",
  "tctx": {
        "action": "BUY",
        "ticker": "MSFT",
        "quantity": "100",
        "customer_type": {
            "geo": "US",
            "level": "VIP"
        }
    }
}
~~~
*Figure 14: Txn-Token-II Payload*

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
