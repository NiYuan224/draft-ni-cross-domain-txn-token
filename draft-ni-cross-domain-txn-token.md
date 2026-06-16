---
title: "Cross-domain Transaction Tokens"
category: info

docname: draft-ni-cross-domain-txn-token-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
  - Identity Chaining
  - Transaction Tokens
  - Cross-Domain

venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "NiYuan224/draft-ni-cross-domain-txn-token"
  latest: "https://NiYuan224.github.io/draft-ni-cross-domain-txn-token/draft-ni-cross-domain-txn-token.html"

author:
 -
  name: Yuan Ni
  organization: Huawei
  email: niyuan1@huawei.com

 -
  name: Chunchi Peter Liu
  organization: Huawei
  email: liuchunchi@huawei.com

normative:
  RFC2119:
  RFC8174:
  RFC6749:
  RFC7523:

informative:
  I-D.kiliram-agent-trust-auth-framework:
  I-D.ietf-oauth-transaction-tokens:
  I-D.ietf-oauth-identity-chaining:

...

--- abstract
This document describes a mechanism for Cross-Domain Transaction Tokens, which enables the safe maintainment and propagation of user identity, workload identities and authorization context across multiple trust domains.


--- middle

# Introduction

Due to the rise of collaboration service ecosystems and AI Agents, service interactions frequently cross domain boundaries and involve multiple service operators (e.g., enterprises, cloud providers, SaaS
platforms, etc.). As illustrated in {{?I-D.kiliram-agent-trust-auth-framework}}, workflows spanning multiple domains require workloads within each domain to perform subtasks and pass results to cooperatively accomplish an overall task. Moreover, to ensure end-to-end accountability, the entities and actions must remain auditable throughout the entire call chain.

This requires the secure preservation and propagation of user identity, workload identities and authorization context across different domains.

Transaction Token (Txn-Token) {{?I-D.ietf-oauth-transaction-tokens}} defines a short-lived, signed JWT that preserves immutable user identity, workload identity, and authorization context within a single trust domain's call chain. While this mechanism effectively preserves context, its applicability is currently limited to intra-domain call chains. Conversely, Identity Chaining {{?I-D.ietf-oauth-identity-chaining}} provides a framework for cross-domain delegation but does not specify how to leverage the rich, structured context carried by Txn-Tokens (i.e., the claims of txn, tctx, rctx, and req_wl in {{?I-D.ietf-oauth-transaction-tokens}}) .

This document bridges this gap by defining a mechanism to use a Txn-Token as the input of Identity Chaining. This approach enables the propagation of workflow-related claims, i.e., claims of txn, tctx, rctx, and req_wl, across multiple trust domains, ensuring the integrity and auditability of the entire call chain.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This document uses the terms "Workload", "Trust Domain", "External Endpoint", "Call Chain", and "Transaction Token Service" (TTS) defined by {{?I-D.ietf-oauth-transaction-tokens}}, as well as "Authorization Server" (AS) defined by {{RFC6749}}. Moreover, the following terms are extended or defined in this document.

* Intra-domain Transaction Token (Txn-Token). A short-lived, signed JWT that provides immutable information about the user, workloads, and specific contextual attributes of the call within a trust domain, as defined in {{?I-D.ietf-oauth-transaction-tokens}}.

* Cross-domain Transaction Token. A Txn-Token that preserves and propagates workflow-related claims from an upstream trust domain to a downstream one via identity chaining as defined in {{?I-D.ietf-oauth-identity-chaining}}.

* Transaction JWT Authorization Grant (Txn-JAG). A specialized JWT acting as an OAuth 2.0 authorization grant as defined in {{RFC7523}}. It carries the workflow-related claims, ensuring the integrity and auditability of the cross-domain call chain.

# Workflows of Cross-Domain Transaction Token

By combining Identity Chaining {{?I-D.ietf-oauth-identity-chaining}} and Transaction Token {{?I-D.ietf-oauth-transaction-tokens}}, this section describes two workflow modes to securely preserve and propagate workflow-related claims across different trust domains. Both modes utilize the Txn-JAG as the secure carrier between domains but differ in how the downstream domain processes it.

* Mode A: Indirect Txn-Token Exchange via Intermediate Access Token. Workload A in Trust Domain I first exchanges its local Txn-Token with its own AS for a Txn-JAG that targets the AS in Trust Domain II. Then, Workload A presents that Txn-JAG to the AS in Trust Domain II to obtain an access token, and uses the access token to invoke Endpoint B in Trust Domain II. Finally, Endpoint B exchanges the access token with the local TTS to acquire a new Txn-Token in Trust Domain II.

* Mode B: Direct Txn-Token Exchange. Workload A in Trust Domain I exchanges its local Txn-Token with its own AS for a Txn-JAG that targets the TTS in Trust Domain II. Then, Workload A presents the Txn-JAG directly to Endpoint B, and Endpoint B exchanges the Txn-JAG with the local TTS to obtain a new Txn-Token in Trust Domain II, thereby eliminating the intermediate round trips for the access token.

The following subsections focus on the logical orchestration and context propagation of these workflows. Definitions for the request and response formats are detailed in Section 4.

## Mode A: Indirect Txn-Token Exchange

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

(1) Workload A in Trust Domain I sends a token exchange request to the local AS to exchange the local Txn-Token for a cross-domain Txn-JAG. See Section 4.1.1 for the request format.

(2) The AS in Trust Domain I validates the request, generates and responds with the Txn-JAG, transcribing the workflow-related claims from the local Txn-Token into the Txn-JAG's claims. See Section 4.1.2 for the response format.

(3) Workload A in Trust Domain I presents the Txn-JAG to the AS of Trust Domain II to request an access token. See Section 4.2.1 for the request format.

(4) The AS of Trust Domain II validates the Txn-JAG and responds with an access token, transcribing the workflow-related claims from the Txn-JAG into the access token. See Section 4.2.2 for the response format.

(5) Workload A in Trust Domain I invokes Endpoint B of Trust Domain II with the received access token.

(6) Endpoint B presents the inbound access token to the TTS of Trust Domain II to request a Txn-Token. See Section 4.3.1 for the request format.

(7) The TTS of Trust Domain II mints a local Txn-Token, transcribing the workflow-related claims from the access token into the new Txn-Token. See Section 4.3.2 for the response format.

## Mode B: Direct Txn-Token Exchange
This mode reduces round trips by allowing the TTS in Trust Domain II to accept the Txn-JAG directly as the subject token in a Txn-Token request. This requires a pre-established trust relationship between the AS in Trust Domain I and the TTS in Trust Domain II. The workflow is illustrated in Figure 2.


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

(3) Workload A in Trust Domain I presents the Txn-JAG to Endpoint B of Trust Domain II. See Section 4.5 for the transmission method.

(4) Endpoint B uses Txn-JAG as the subject token to exchange for a local Txn-Token at its local TTS.

(5) The TTS of Trust Domain II mints a local Txn-Token, transcribing the workflow-related context from the Txn-JAG into the local Txn-Token's claims.


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
