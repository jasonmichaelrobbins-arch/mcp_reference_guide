[MCP_Security_Reference_Guide.md](https://github.com/user-attachments/files/24315664/MCP_Security_Reference_Guide.md)
# MCP Security Reference Guide

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

A side-by-side reference for securing internal (self-hosted) versus external (vendor-hosted) MCP server deployments—with prioritized controls, architecture diagrams, validation questions, and a decision framework.

*Last Updated: February 2026*

---

## Why This Exists

MCP (Model Context Protocol) servers are becoming critical infrastructure for AI agent deployments, but security teams lack clear guidance on how to evaluate and secure them. Questions pile up:

* Should we self-host or use a vendor?
* What controls are non-negotiable before production?
* How do internal and external deployments differ from a security perspective?
* What questions should we ask during a security assessment?

This guide answers all of that with an opinionated, prioritized framework. It's designed to be used alongside the [Enterprise AI Security Governance Framework](https://github.com/jasonmichaelrobbins-arch/-ai-security-governance-workbook) for organizations deploying MCP infrastructure.

**This is a living document.** MCP security practices are evolving rapidly. Contributions welcome.

---

## Table of Contents

* [What's Included](#whats-included)
* [Quick Start](#quick-start)
* [Priority Legend](#priority-legend)
* [Architecture Overview](#architecture-overview)
  * [Why These Diagrams Matter](#why-these-diagrams-matter)
  * [External MCP Server Architecture](#external-mcp-server-architecture)
  * [Internal MCP Server Architecture](#internal-mcp-server-architecture)
* [Shared Concerns](#shared-concerns)
* [Internal vs. External MCP Servers](#internal-vs-external-mcp-servers)
* [Security Validation Questions](#security-validation-questions)
* [Decision Framework](#decision-framework)
* [Related Resources](#related-resources)
* [Contributing](#contributing)
* [License](#license)
* [Author](#author)

---

## What's Included

| Component | Purpose |
|-----------|---------|
| **Priority Legend** | Four-tier severity model (P1-P4) for control prioritization |
| **Architecture Diagrams** | Visual data flow for internal and external MCP deployments |
| **Shared Concerns** | Controls required regardless of deployment model |
| **Comparison Matrix** | Side-by-side security controls for internal vs. external |
| **Validation Questions** | Quick-hit questions for security assessments |
| **Decision Framework** | Criteria for choosing between deployment models |

---

## Quick Start

1. **Determine your deployment model** — Use the [Decision Framework](#decision-framework) to evaluate internal vs. external hosting
2. **Review shared concerns first** — These P1-P3 controls apply to both models
3. **Use the comparison matrix** — Identify specific controls for your chosen model
4. **Validate with assessment questions** — Use [Security Validation Questions](#security-validation-questions) during vendor or architecture reviews
5. **Prioritize by severity** — Start with P1 (Severe), then P2 (High), etc.

---

## Priority Legend

| Priority | Level | Description |
|----------|-------|-------------|
| **P1** | Severe | Must have before production. Fundamental security controls. |
| **P2** | High | Should have for production. Important defense-in-depth. |
| **P3** | Medium | Plan for implementation. Governance and operational maturity. |
| **P4** | Recommended | Nice to have. Advanced capabilities for mature programs. |

---

## Architecture Overview

Before diving into specific security controls, it is essential to understand the fundamental architectural differences between internal and external MCP server deployments. The diagrams below illustrate the data flow, trust boundaries, and key security control points for each model.

### Why These Diagrams Matter

| Concept | Why It Matters |
|---------|----------------|
| **Trust boundaries** | External deployments introduce a trust boundary at the public internet, requiring different controls than internal deployments where traffic stays within your network perimeter. |
| **Control points** | The gateway layer serves different purposes—for external MCP, it injects API keys and sanitizes outbound data; for internal MCP, it provides defense-in-depth before traffic enters the private subnet. |
| **Visibility gaps** | With external vendors, the MCP server is a "black box"—you control only what enters and exits. Internal deployments give full visibility into tool execution, container isolation, and backend access. |
| **Logging strategy** | Both models require comprehensive logging, but the sources differ. External deployments depend heavily on gateway-side logging since vendor logs may have retention limits. |
| **Credential management** | External deployments inject vendor API keys at the gateway; internal deployments use managed identities and service principals with per-MCP/per-agent scoping. |

### External MCP Server Architecture

The external (vendor-hosted) model routes all traffic through your API gateway before reaching the vendor's MCP server over the public internet. Your gateway handles authentication, DLP/redaction, API key injection, and comprehensive logging.

![External MCP Architecture](images/external-mcp-architecture.png)

### Internal MCP Server Architecture

The internal (self-hosted) model keeps all MCP infrastructure within your private network. Traffic flows through your gateway, across a VPN or private link, into a private subnet with no public IPs. You maintain full control over container isolation, secrets management, immutable logging (WORM - Write Once Read Many), and oversight agents.

![Internal MCP Architecture](images/internal-mcp-architecture.png)

---

## Shared Concerns

The concerns below are essential regardless of whether you run internal or external MCP servers.

| Priority | Area | Control | Key Question |
|----------|------|---------|--------------|
| **P1** | Kill switches | Ability to quickly disable tools, endpoints or entire MCP connections | Can you disable any agent/tool within five minutes? |
| **P1** | Authentication | Strong identity verification for all callers (humans, agents, services) | Can an unauthenticated request ever reach the MCP? |
| **P1** | Logging | Capture prompts, tool calls and responses with appropriate redaction | Can you reconstruct what an agent did and why? |
| **P1** | Prompt injection defense | Input sanitization, injection detection, output validation | What happens if malicious content is in retrieved documents? |
| **P2** | Human-in-the-loop | Require approval for high-risk, irreversible or financial actions | Which actions can an agent take without human approval? |
| **P2** | Data classification | Label and control data by sensitivity before it reaches MCP | Do you know what data sensitivity levels the MCP can access? |
| **P2** | Tool minimization | Only enable tools necessary for the use case; treat each tool as a risk | Is there a tool enabled that isn't actively needed? |
| **P2** | Agent orchestration | Secure agent-to-agent communication; validate delegated actions; prevent privilege escalation across agents | How do you secure communication between agents in multi-agent workflows? |
| **P2** | Cost controls | Set budget limits, monitor token consumption, alert on runaway usage, implement circuit breakers | What stops a misbehaving agent from burning through your API budget? |
| **P3** | Lifecycle governance | Catalog, risk-tier and manage MCP servers/connections through lifecycle | Do you have a single source of truth for all MCP connections? |
| **P3** | Incident response | AI-specific playbooks for containment, evidence preservation and communication | Do you have an incident response playbook specifically for AI/agent incidents? |
| **P3** | Red teaming | Test for prompt injection, data exfiltration, tool abuse, jailbreaks | When did you last red team your MCP integrations? |

---

## Internal vs. External MCP Servers

The table below provides a detailed comparison of security controls for internal (self-hosted) versus external (vendor-hosted) MCP server deployments.

| Priority | Area | Internal MCP Server (Self-hosted) | External MCP Server (Vendor-hosted) |
|----------|------|-----------------------------------|-------------------------------------|
| **P1** | Scope and risk tier | Classify by data sensitivity, integration depth, agent capability (read-only -> supervised -> autonomous). Use 4-tier priority model. | Classify by what data you'll send and what actions the vendor can perform. Start restrictive, expand only with justification. |
| **P1** | Network | No public IPs. Private subnets only. Access via VPN/ExpressRoute. Inbound only from gateway. | All traffic via controlled egress (API management/gateway). Allowlist vendor FQDNs. Use private link if available. |
| **P1** | Identity and auth | Entra ID + OAuth 2.1. Validate JWT issuer/audience exactly. Short-lived tokens. Token binding for replay prevention. | Prefer your identity provider (Entra) over vendor accounts. Separate keys per environment. Verify vendor token handling. Confirm no sub-processor access. |
| **P1** | Authorization | Enforce at gateway AND backend. Object-level authz. Map Entra claims to MCP roles (admin/owner/user/read-only). | Map internal roles to vendor permissions. Avoid broad admin scopes. Require approval workflows for high-risk actions. |
| **P1** | Secrets and non-human identities | Unique service principal per MCP/agent. Managed identities preferred. Vault for secrets. Rotation + expiration alerts. | Keys only in vault, injected at gateway. Never expose keys to users/agents. Rotate keys on schedule and staff departure. |
| **P1** | Input validation | Direct and indirect prompt injection defense. Input sanitization. Output validation. Schema validation. Encoding checks. | DLP/redaction at gateway. Prompt shielding. Strip secrets before sending. Context length limits. |
| **P2** | Tool execution | Container/VM isolation. Resource limits (CPU/memory/time). Network segmentation. Filesystem isolation. Recursive call limits. | Vendor tools are black boxes. Broker via your APIs. No direct write to core systems. Monitor for capability drift. |
| **P2** | Data protection | Classify data. Segment sensitive indexes. Access checks at retrieval. Data lineage. Unlearning/deletion. Poisoning detection. | Define allowed data classes. Start with non-sensitive data. Synthetic data for testing. Minimal vendor logging. Differential privacy. |
| **P2** | Session/memory | Encrypt context at rest. Session timeouts. Cross-session isolation. Bound memory size. Automatic clearing. | Verify vendor session isolation. Confirm no cross-tenant leakage. Test for context persistence across sessions. |
| **P2** | Gateway layer | Centralized gateway for all MCP access. JWT validation at edge. Rate limiting. Schema validation. Request signing. | Force all traffic through your gateway. Inject vendor keys at gateway. Log everything. Rate limit per user/agent. |
| **P2** | Supply chain | Assess model dependencies, libraries, base images. Monitor for vulnerabilities in tool integrations. | Map vendor's model providers and subprocessors. Understand inference location. Require notification of upstream changes. |
| **P2** | Logging and monitoring | Full tracing with redaction. Log integrity (WORM). Oversight agents. Metrics. Correlation IDs. Alert on anomalies. | Log everything on your side. Ingest vendor signals. Correlation IDs. Behavioral baselines. Response integrity monitoring. |
| **P2** | Availability/SLAs | Define RTO/RPO. Backup configs and indices. Geographic redundancy. Disaster recovery testing. | Negotiate SLAs (uptime, latency). Degradation plans. Failover options. Monitor vendor status. |
| **P3** | Governance | Catalog MCP servers/agents. Risk tier -> capabilities. Change management. Promotion gates (shadow -> autonomous). | AI-flavored vendor risk assessment. SOC 2/ISO. Risk tier per vendor. Concentration risk. Financial health. |
| **P3** | Portability | Version control configs. Documented deployment. Reproducible infrastructure. | Assess lock-in API compatibility with alternatives. Abstraction layers. Migration runbooks. Data export. |
| **P3** | Integrity verification | Request signing. Replay prevention. TLS everywhere, including internal. | Response authenticity. Cert pinning. Detect response modifications. Monitor for vendor-side injection. |
| **P3** | Legal/compliance | Data security posture management (DSPM) for data residency. Regulatory mapping (including EU AI Act and emerging AI regulations). Multicontrols if applicable. | Data processing agreement (DPA)/business associates' agreement (BAA). AI clauses (no training, intellectual property ownership). Cyber insurance. Liability terms. Regulatory fit. |
| **P3** | Testing and validation | Threat model. Red team. Config review. Continuous security testing in CI/CD. Resource exhaustion tests. | Sandbox with synthetic data. Red-team integration. Continuous evaluation. Canary deployments. A/B testing. Cross-tenant tests. |
| **P4** | Multi-tenancy | Tenant isolation at MCP. Scoped indices. Per-tenant keys. Per-tenant rate limits and audit logs. | Verify vendor tenant isolation. Test for cross-tenant access. Contractual isolation guarantees. |

---

## Security Validation Questions

Use these questions to quickly assess MCP server security posture.

| Internal MCP Servers | External MCP Servers |
|----------------------|----------------------|
| Can requests reach MCP without going through the gateway? | Can any user/agent call the vendor directly (bypassing your gateway)? |
| What happens if a token is stolen? | Where does the vendor store your API keys and for how long? |
| Can one user access another user's data by manipulating IDs? | What data is the vendor allowed to log and retain? |
| What tools can an agent call without human approval? | Which of the vendor's tools can write to your systems? |
| How quickly can you disable a misbehaving agent? | How quickly can you cut all traffic to this vendor? |
| Where are secrets for this MCP server stored? | What happens to your data if the vendor is acquired? |
| Can you reconstruct what an agent did last week? | Does the vendor train models on your prompts/responses? |
| What's in your backup for this MCP and when was it tested? | What's your plan if this vendor has a major outage? |

---

## Decision Framework

Use the following criteria to determine which deployment model best fits your use case.

| Favor Internal MCP Servers When... | External MCP Servers May be Acceptable When... |
|------------------------------------|------------------------------------------------|
| Processing PII, PHI, PCI or highly confidential data | Working with public data or low-sensitivity internal data |
| Regulatory requirements mandate data residency control | Vendor meets all compliance requirements with attestation |
| Need full control over model behavior and guardrails | Vendor guardrails are sufficient for use case |
| Actions have high financial or operational impact | Actions are read-only or easily reversible |
| Require deep integration with internal systems | Integration is limited to specific, well-bounded use cases |
| Long-term strategic capability requiring investment | Rapid experimentation or time-to-value is critical |

---

## Related Resources

| Resource | Description |
|----------|-------------|
| [Enterprise AI Security Governance Framework](https://github.com/jasonmichaelrobbins-arch/-ai-security-governance-workbook) | Comprehensive AI tool governance with intake forms, guardrail checklists, and compliance mappings |
| [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/) | LLM-specific security vulnerabilities |
| [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework) | AI Risk Management Framework |
| [Model Context Protocol Spec](https://modelcontextprotocol.io/) | Official MCP specification |

---

## Contributing

This framework will evolve as MCP security practices mature. Contributions welcome:

* **Issues:** Found a gap? Have a question? Open an issue.
* **Pull Requests:** Additions, clarifications, real-world examples—all welcome.
* **Experience reports:** Implemented this? I'd love to hear what worked and what didn't.
* **Architecture patterns:** New deployment models or control patterns? Share them.

---

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

You are free to:

* **Share** — copy and redistribute in any medium or format
* **Adapt** — remix, transform, and build upon the material for any purpose

With attribution.

---

## Author

**Jason Robbins**

---

*Built for the security community. Not paywalled.*
