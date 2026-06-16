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
  I-D.ietf-kiliram-agent-trust-auth-framework:
  I-D.ietf-oauth-transaction-tokens:
  I-D.ietf-oauth-identity-chaining:

...

--- abstract
This document describes a mechanism for Cross-Domain Transaction Tokens, which enables the safe maintainment and propagation of user identity, workload identities and authorization context across multiple trust domains.


--- middle

# Introduction

Due to the rise of collaboration service ecosystems and AI Agents, service interactions frequently cross domain boundaries and involve multiple service operators (e.g., enterprises, cloud providers, SaaS
platforms, etc.). As illustrated in {{?I-D.ietf-kiliram-agent-trust-auth-framework}}, workflows spanning multiple domains require workloads within each domain to perform subtasks and pass results to cooperatively accomplish an overall task. Moreover, to ensure end-to-end accountability, the entities and actions must remain auditable throughout the entire call chain.

This requires the secure preservation and propagation of user identity, workload identities and authorization context across different domains.

Transaction Token (Txn-Token) {{?I-D.ietf-oauth-transaction-tokens}} defines a short-lived, signed JWT that preserves immutable user identity, workload identity, and authorization context within a single trust domain's call chain. While this mechanism effectively preserves context, its applicability is currently limited to intra-domain call chains. Conversely, Identity Chaining {{?I-D.ietf-oauth-identity-chaining}} provides a framework for cross-domain delegation but does not specify how to leverage the rich, structured context carried by Txn-Tokens (i.e., the claims of txn, tctx, rctx, and req_wl in {{?I-D.ietf-oauth-transaction-tokens}}) .

This document bridges this gap by defining a mechanism to use a Txn-Token as the input of Identity Chaining. This approach enables the propagation of workflow-related claims, i.e., claims of txn, tctx, rctx, and req_wl, across multiple trust domains, ensuring the integrity and auditability of the entire call chain.


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
