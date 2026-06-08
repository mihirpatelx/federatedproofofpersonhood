# Federated Human Verification: Authentication Infrastructure for an Autonomous World

**Mihir**  
*March 2026*

---

## Abstract

Every autonomous system operating today—military drones, unmanned vehicles, robotic platforms, AI agents—lacks a cryptographic standard for proving that a human authorized its actions. As these systems move to the edge and operate in communications-denied environments, the ability to verify human-in-the-loop locally, offline, and across sovereign boundaries becomes mission-critical infrastructure. We propose a federated human verification protocol built on the W3C Verifiable Credentials standard. Sovereign governments and authorized institutions issue human verification credentials according to their own standards. A confidence scoring layer aggregates and weights these credentials. Autonomous systems verify them offline against a cached trust store. The protocol is designed for a world where no single entity is trusted by all parties—the only architecture that works across coalition operations, allied defense systems, and multinational civilian deployments.

---

## Contents

1. [Introduction](#1-introduction)
2. [The Protocol](#2-the-protocol)
   - 2.1 [Credential Layer: W3C Verifiable Credentials](#21-credential-layer-w3c-verifiable-credentials)
   - 2.2 [Confidence Score](#22-confidence-score)
   - 2.3 [Application-Defined Thresholds](#23-application-defined-thresholds)
3. [Issuers](#3-issuers)
4. [Edge Verification](#4-edge-verification)
   - 4.1 [The Problem](#41-the-problem)
   - 4.2 [Human-at-the-Top](#42-human-at-the-top)
   - 4.3 [Offline Verification](#43-offline-verification)
   - 4.4 [Why Federated Issuance Is the Only Viable Model](#44-why-federated-issuance-is-the-only-viable-model)
5. [Data Flow and Aggregation](#5-data-flow-and-aggregation)
   - 5.1 [Client-Side Aggregation](#51-client-side-aggregation)
   - 5.2 [Decentralized Aggregator Network](#52-decentralized-aggregator-network)
   - 5.3 [Deduplication](#53-deduplication)
6. [Convergence](#6-convergence)
7. [The Issuer Registry](#7-the-issuer-registry)
8. [Open Problems](#8-open-problems)
9. [Conclusion](#9-conclusion)

---

## 1 Introduction

Autonomous systems are proliferating across defense, industrial, and civilian domains. Drones execute missions in communications-degraded environments. Robotic platforms operate in contested networks. Autonomous vehicles navigate without persistent connectivity. AI agents take consequential actions on behalf of human operators.

Every one of these systems faces the same fundamental question before executing a consequential action: has a verified human authorized this?

Today, there is no universal standard for answering that question. Chain of command verification depends on proprietary systems, network connectivity, or nothing at all. In communications-denied environments, autonomous systems either halt—failing their mission—or act without verification, which is unaccountable autonomy. As international law increasingly requires demonstrable human-in-the-loop for lethal autonomous systems, the absence of a verification standard becomes an operational, legal, and strategic vulnerability.

The problem is compounded by sovereignty. A US defense system cannot depend on credentials issued by a foreign commercial entity. A NATO coalition system needs credentials from allied government issuers with mutual recognition. A Japanese industrial robot needs credentials from issuers that Japanese regulators trust. No single provider will be trusted across all of these contexts.

We observe that the world coordinates on infrastructure challenges like this through shared standards with distributed issuance. ICAO defines the passport standard; each nation issues its own. X.509 defines the certificate standard for TLS; certificate authorities are distributed. Visa and Mastercard define interchange standards; issuing banks are local. Human verification should follow the same model.

The protocol consists of three components:

1. A human verification credential schema built on the W3C Verifiable Credentials Data Model 2.0, allowing any sovereign government or authorized institution to issue human verification credentials within an existing, widely adopted standard.
2. A confidence score computed from weighted credentials with temporal decay.
3. Application-defined thresholds that let each system—from a social platform to a weapons platform—set its own verification requirements.

---

## 2 The Protocol

### 2.1 Credential Layer: W3C Verifiable Credentials

The protocol builds on the W3C Verifiable Credentials Data Model 2.0, a formal W3C Recommendation that defines how credentials are formatted, signed, presented, and verified. The VC spec supports the three-party model this protocol requires—issuers produce credentials, holders carry them, and verifiers consume them—and it supports zero-knowledge proofs as a securing mechanism.

What the VC spec does not define is what human verification looks like as a credential, or how to evaluate the trustworthiness of credentials from different sovereign issuers. This protocol fills that gap by defining a human verification credential schema and a scoring layer on top of the existing VC infrastructure.

Any sovereign government or authorized institution can become an issuer by producing human verification VCs that conform to this schema. The US Department of Defense issues credentials to military personnel. Estonia issues credentials through its e-Residency infrastructure. India issues credentials anchored to Aadhaar. A private entity issues credentials based on biometric verification or device binding. All produce Verifiable Credentials in the same format, verifiable by the same systems, interoperable across the entire VC ecosystem.

Building on VCs provides a critical adoption advantage: governments and institutions are already implementing VC infrastructure for digital identity programs. Adding a human verification credential type to their existing systems is incremental.

Human verification credential types fall into three categories of decreasing signal strength:

- **Biometric** (iris, fingerprint, facial recognition) — strongest signal. Proves a specific physical human.
- **Government identity** (national ID, passport, military ID, security clearance) — strong, backed by civil or defense registries.
- **Behavioral** (device identity, location patterns, biometric health signals, social accounts) — weaker individually, strong in combination. Useful as supplementary signals and as an on-ramp for civilian applications.

### 2.2 Confidence Score

The protocol aggregates a user's credentials into a single confidence score. Each credential type carries a weight reflecting its strength as a verification signal, and each credential decays over time since issuance. Given a user $u$ with credentials $\{a_1, \ldots, a_n\}$ at current time $t$:

$$C(u, t) = 1 - \exp\!\left(-\lambda \sum_{k=1}^{n} w(\tau_k) \cdot \exp\!\left(-\frac{\ln 2}{h_{\tau_k}} \cdot (t - t_k)\right)\right)$$

where $w(\tau_k)$ is the weight of credential type $\tau_k$, $h_{\tau_k}$ is the half-life governing how quickly that credential type decays, $t_k$ is the issuance time, and $\lambda$ is a rate parameter. The score lives in $[0, 1]$ and saturates toward 1 as credential strength increases.

The decay component is critical. A credential degrades without re-verification. This ensures the score reflects current status—personnel change roles, clearances expire, assignments rotate. The decay rate varies by type: a military ID credential decays slowly, while a device-binding credential decays faster.

### 2.3 Application-Defined Thresholds

Each system that consumes the protocol sets its own verification requirements. A threshold policy specifies a minimum confidence score, optionally required credential types, and optionally required issuers:

- **Surveillance drone:** Moderate threshold, government identity credential from the operating nation's defense issuer.
- **Weapons platform:** Highest threshold, biometric credential plus government defense credential from the specific issuer authorized for that system, freshness within the last hour.
- **Autonomous vehicle:** High threshold, government identity plus biometric credential from the relevant jurisdiction's regulatory issuer.
- **Industrial robot:** Moderate threshold, employer-issued credential plus safety certification credential from the relevant regulatory body.
- **Civilian platform (bot prevention):** Low threshold, behavioral credentials sufficient.

The protocol provides the score; the system defines the policy. The same credential infrastructure serves a spectrum from civilian platform authentication to lethal autonomy authorization. The difference is the threshold, the required credential types, and the required issuers.

---

## 3 Issuers

The protocol has issuers, not centralized registries. There is no single global authority. There are sovereign issuers—governments, defense departments, regulatory bodies, authorized institutions—each issuing human verification credentials according to their own standards and legal frameworks.

The US Department of Defense is an issuer. NATO member states are issuers. The EU member states are issuers whose credentials interoperate through existing political agreements, the same way their passports do. Japan's safety regulators are issuers for industrial contexts. Each issuer controls its own enrollment, its own verification methods, and its own credential lifecycle.

Coordination between issuers happens at the political level—through treaties, alliances, and mutual recognition agreements—the same way it already works for passports, security clearances, and military interoperability standards. The protocol defines the credential format and the scoring model. Everything beyond that is politics, not protocol.

Because government issuers have already solved internal deduplication through civil and defense registries, the protocol inherits this property. The protocol's job is cross-issuer interoperability.

---

## 4 Edge Verification

### 4.1 The Problem

Autonomous systems increasingly operate at the edge, in environments where persistent connectivity to a central authority is unavailable or actively denied. A drone in a communications-degraded theater cannot query a remote server. A robotic system in a contested network cannot depend on a round trip to a cloud-hosted identity provider. A surgical robot mid-operation cannot pause for a connectivity timeout.

These systems make decisions with irreversible consequences: a defense drone executes a strike, a surgical robot makes an incision, an autonomous vehicle navigates a crowd, an industrial system initiates a process that cannot be safely interrupted. Verification failure in these contexts is a mission failure, a safety hazard, or a loss of life.

### 4.2 Human-at-the-Top

In any authorization chain for autonomous systems, the human verification credential is the root. Everything downstream—what actions are permitted, what autonomy level is granted, what rules of engagement apply, what liability attaches—flows from the initial proof that a human is in the loop.

In defense contexts, this is chain of command. A credential issued by a government's defense apparatus, carried in an operator's secure device, verifiable offline against a cached trust store, answers the question: is this a verified member of this military, authorized to command this system? The full authorization chain is cryptographically auditable—from the human operator, through the credential, to the action taken. This matters for rules of engagement compliance, post-action accountability, and international law.

In civilian contexts, the stakes are equally concrete. An autonomous vehicle handoff failure at highway speed is fatal. A surgical robot credential failure mid-procedure endangers the patient. An industrial authorization failure could mean an uncontrolled chemical process.

As AI systems become more capable and are deployed in more consequential settings, the question of "was a human in the loop" becomes the single most important verification in the entire system.

### 4.3 Offline Verification

The protocol is designed for offline verification from the ground up. The confidence score is precomputed by the network and delivered to the user's device as a signed, timestamped score certificate. At verification time, the autonomous system checks the certificate's signature against a cached issuer registry and checks the timestamp against its freshness policy. No network round trip. No central server. Signature verification takes milliseconds.

The issuer registry is cached locally on edge devices, the same way devices pre-load encryption keys or certificate trust stores. Updates are synced when connectivity is available. Between syncs, the system operates fully autonomously against its cached trust store.

Freshness tolerance is mission-defined. A long-duration autonomous patrol might accept certificates that are days old. A strike authorization might require certificates from the last hour. The framework supports the full spectrum.

### 4.4 Why Federated Issuance Is the Only Viable Model

No defense department will outsource chain of command verification to a commercial provider. No sovereign nation will accept credentials issued by a foreign private entity for mission-critical authorization. No coalition operation will depend on a single commercial root of trust.

The federated model is the only architecture that works: each sovereign entity issues credentials according to its own standards, each system pre-loads the trust store relevant to its operating context, and verification happens locally. Allied interoperability is achieved through mutual recognition of issuers within the shared standard—the same mechanism that enables NATO interoperability on communications, encryption, and identification standards.

---

## 5 Data Flow and Aggregation

The VC spec defines how credentials are stored, presented, and secured with zero-knowledge proofs. This protocol inherits all of that. The question the VC spec does not answer is: who aggregates credentials from multiple issuers into a single score?

### 5.1 Client-Side Aggregation

At launch, the operator's VC wallet handles aggregation. The wallet holds credentials from multiple issuers, computes the confidence score locally, and generates a zero-knowledge proof that the score meets the system's policy—without revealing which credentials contribute to it.

This approach is simple and requires no additional infrastructure. It has one limitation: the operator controls which credentials to present. A ZK proof guarantees correct computation from the presented inputs, but cannot prove all credentials were presented. For most applications this is acceptable. For high-stakes defense applications, the system can require specific credentials from specific issuers, eliminating selective disclosure as a concern.

### 5.2 Decentralized Aggregator Network

The long-term architecture introduces a network of independent nodes that collectively aggregate credentials and run anomaly detection via threshold computation. No single node sees the full picture. The network precomputes scores asynchronously and delivers signed score certificates to operators' devices. This closes the selective disclosure gap for applications that require it.

Key design constraints:

- **No single point of trust.** Compromise requires collusion across a threshold number of nodes.
- **No token dependency.** Node operators are compensated through protocol fees or institutional funding.
- **Governance alignment.** Node operation is open to governments, defense institutions, universities, and nonprofits—entities with reputational stakes that align with honest operation.

### 5.3 Deduplication

The VC spec's ZK proof support handles credential presentation privacy. Cross-issuer deduplication—ensuring the same person has not registered with multiple issuers—requires an additional commitment: when issuing a credential, the issuer derives a cryptographic commitment from the operator's identity signal and publishes it to the protocol. The commitment is deterministic (same person always produces the same commitment) and hiding (reveals nothing about who the person is). For government issuers with civil or defense registries, the same identifier always maps to the same commitment, enabling deduplication without exposing identity data.

---

## 6 Convergence

The protocol is designed to become simpler over time.

At launch, the system accepts many credential types of varying strength—behavioral signals, device identity, social accounts—because they are the only signals available for civilian applications. As government and biometric issuers come online, a single government credential carries so much weight that it dominates the score. The weaker credential types do not get removed. They simply stop mattering for anyone who has access to stronger ones.

The result is that the protocol organically converges toward a state where most operators need only two or three strong credentials—a government identity and a biometric. The system starts broad and narrows naturally, driven by the mathematics of the scoring function.

For defense applications, convergence is immediate: the required credentials are government-issued from day one. The convergence property primarily benefits the civilian adoption path, where the protocol is useful before governments participate, and strengthens continuously as stronger issuers join. There is no threshold of government adoption that must be reached before the system works.

---

## 7 The Issuer Registry

The protocol is open. The scoring formula is public. Anyone can issue a human verification VC. The question every system faces is: which issuers should I trust, and how much?

This is what the issuer registry solves, and it is where value is captured.

The model mirrors the TLS certificate ecosystem. The X.509 standard is open. Anyone can issue a certificate. But browsers ship with a trusted root store—a curated list of certificate authorities whose certificates are accepted. The entities that manage root stores hold enormous power over the ecosystem without owning the standard itself.

The issuer registry is the root store for human verification credentials. It is a curated list of which issuers are admitted, what credential types they are authorized to issue, and what weight their credentials carry in the scoring function. Systems that want a meaningful confidence score integrate the registry. Issuers that want their credentials to count apply for admission. The registry operator vets issuers, assigns weights, monitors quality through verification-layer auditing, and removes or downgrades issuers whose credentials show anomalous patterns.

If an issuer is not in the registry, their credentials carry zero weight. The more systems rely on the registry, the more issuers need to be in it. The more issuers are in it, the more useful the registry becomes. This is a network effect.

In defense contexts, the registry takes on additional significance. The question of which issuers are trusted for which systems is a sovereign security decision. A defense-specific registry—or a defense tier within the broader registry—curated in partnership with defense institutions, becomes critical infrastructure for autonomous operations.

The protocol can outlive any single registry operator. Governments are adopting an open standard that happens to have a registry available. If the operator disappears, the protocol works and another entity can step in.

Competition is possible. Multiple registries on the same protocol is analogous to multiple browser root stores on the same TLS standard. Defense registries, civilian registries, and sector-specific registries can coexist.

Revenue aligns with growth. The registry operator charges for access and score verification—per check, by subscription, or by contract. As adoption grows, revenue scales with verification volume.

---

## 8 Open Problems

1. **Cross-issuer sybil resistance.** An individual with valid credentials from multiple jurisdictions may obtain independent credentials from each. The anomaly detection mechanism raises the cost but does not eliminate the possibility.

2. **Incentive alignment among issuers.** Sovereign issuers may have incentives to over-issue credentials. Can the protocol detect or penalize lax issuers without a central authority?

3. **Cross-jurisdiction deduplication.** Government identity credentials use jurisdiction-specific identifiers (national ID numbers) that do not match across borders. Cross-jurisdiction deduplication is only possible when issuers use a person-intrinsic signal—such as a biometric—as the commitment input. As the protocol converges toward biometric credentials, this gap closes naturally.

4. **Commitment scheme governance.** How is the commitment scheme updated without breaking backward compatibility or requiring all issuers to re-enroll their populations?

5. **Decay calibration.** The choice of half-lives for each credential type requires empirical data from deployment that does not yet exist.

6. **Decentralized aggregator bootstrapping.** Who operates the initial nodes? How is honest operation incentivized? What is the minimum viable node count for meaningful threshold security?

---

## 9 Conclusion

Every autonomous system operating today lacks a cryptographic standard for proving a human authorized its actions. As these systems become more capable, more numerous, and more consequential, this gap becomes an operational, legal, and strategic vulnerability.

We have presented a federated human verification protocol built on the W3C Verifiable Credentials standard. Sovereign governments and authorized institutions issue credentials according to their own standards. A confidence scoring layer with temporal decay aggregates and weights these credentials. Systems set their own verification thresholds—from civilian bot prevention to lethal autonomy authorization. Verification happens offline against a cached trust store, with no network dependency.

A trusted issuer registry provides the curation that gives scores meaning, with a network effect that strengthens as adoption grows. The protocol is designed to converge: it starts permissive, accepting many weak signals, and naturally simplifies as stronger sovereign issuers come online.

The federated model is the only architecture that works for human verification at this scale—because no single entity will be trusted across all the sovereign, military, and regulatory contexts where autonomous systems operate. The protocol is designed for that world.
