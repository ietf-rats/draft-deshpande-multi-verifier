---
title: Remote Attestation with Multiple Verifiers
abbrev: RATS Many-Verifiers
docname: draft-deshpande-rats-multi-verifier-latest
category: info
ipr: trust200902
area: Security
workgroup: RATS

stand_alone: yes
pi:

  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  text-list-symbols: -o*+
  docmapping: yes

author:

-
  name: Yogesh Deshpande
  org: Arm Ltd
  email: yogesh.deshpande@arm.com

-
  name: Jun Zhang
  org: Huawei Technologies France S.A.S.U.
  email:  junzhang1@huawei.com

- name: Houda Labiod
  org: Huawei Technologies France S.A.S.U.
  email: houda.labiod@huawei.com

-
  name: Henk Birkholtz
  org: Fraunhofer SIT
  email:  henk.birkholz@sit.fraunhofer.de

contributor:
 - name: Thomas Fossati
   organization: Linaro
   email: Thomas.Fossati@linaro.org

 - name: Thanassis Giannetsos
   organization: UBITECH Ltd.
   email: agiannetsos@ubitech.eu

 - name: Steven Bellock
   organization: NVIDIA
   email: sbellock@nvidia.com

 - name: Ghada Arfaoui
   organization: ORANGE
   email: ghada.arfaoui@orange.com

informative:
  RFC9334: rats-arch
  I-D.draft-ietf-rats-corim: corim
  RFC6024: trust-anchors

--- abstract

IETF RATS Architecture, defines the key role of a Verifier.  In a complex system, this role needs to be performed by multiple Verfiers coordinating together to assess the full trustworthiness of an Attester. This document focuses on various topological patterns for a multiple Verifier system. It only covers the architectural aspects introduced by the Multi Verifier concept, which is neutral with regard to specific wire formats, encoding, transport mechanisms, or processing details.

--- middle

# Introduction

A Verifier plays a central role in any Remote Attestation System. A Verifier appraises the Attester and produces Attestation Results, which are essentially a verdict of attestation. The results are consumed by the Relying Party to conclude the trustworthiness of the Attester, before making any critical decisions about the Attester, such as admitting it to the network or releasing confidential resources to it.
Attesters can come in wide varieties of shape and form. For example Attesters can be endpoints (edge or IoT devices) or complex machines in the cloud. Composite Attester {{sec-glossary}}, generate Evidence that consists of multiple parts. For example, in data center servers, it is not uncommon for separate attesting environments (AE) to serve a subsection of the entire machine. One AE might measure and attest to what was booted on the main CPU, while another AE might measure and attest to what was booted machine's GPU. Throughout this document we use the term Component Attester {{sec-glossary}} to address the sub-entity or an individual layer which produces its own Evidence in a Composite Attester system.

In a Composite Attester system, it may not be possible for a single Verifier to possess all the capabilities or information required to conduct a complete appraisal of the Attester. Please refer to {{sec-need-multiverifier}} for motivation of this document. Multiple Verifiers need to collaborate to reach a conclusion on the appraisal and produce the Attestation Results.


This document describes various topological patterns of multiple Verifiers that work in a coordinated manner to conduct appraisal of a Composite Attester to produce an Attestation Results.

# Need for Multiple Verifiers
{: #sec-need-multiverifier }
To conduct the task of Evidence appraisal, a Verifier requires:

1. Reference Values from trusted supply chain actors producing, aggregating, or administering Attesters (Reference Value Providers)

2. Endorsements from trusted supply chain actors producing, certifying, or compliance checking Attesters (Endorsers)

3. Appraisal Policy for Evidence, which is under the control of the Verifier Owner

The Verifier inputs listed above are linked to the shape of the Attesters.
Typically, Composite Attesters come with a varying degree of heterogeneity of Evidence formats, depending on the type of Attesting Environments that come with each Component Attester, for example, CPU variants or GPU/FPGA variants. When conducting Evidence appraisal for a Composite Attester, the following challenges remain:

1. An Attester's composition can change over time based on market requirements and availability (e.g., a set of racks in a data center gets thousands of new FPGAs).
It is highly unlikely that there is always one appropriate Verifier that satisfies all the requirements that a complex and changing Composite Attesters imposes.
It may not be economically viable to build and maintain such a degree of complexity in a single Verifier.
2. A Verifier Owner may have an Appraisal Policy for Evidence of a Component Attester that is internal to them and which they may choose not to reveal to a “monolithic" Verifier.
3. A Reference Values Provider may not wish to reveal its Reference Values or their lifecycle to a monolithic Verifier.
4. There may not be a single actor in the ecosystem that can stand up and take ownership of verifying every Component Attester due to a lack of knowledge, complexity, regulations or associated cost.
5. The mix today is a combination of Verifier services provided by component manufacturers, Verifiers provided by integrators, and Verifiers under local authority (i.e., close to the attester). Rarely is it just one of these.


# Reference Use Cases
This section covers generic use cases that demonstrate the applicability of Multi Verifier, regardless of specific solutions.
Its purpose is to motivate various aspects of the architecture presented in this document.
There are many other use cases; this document does not contain a complete list.

## Verification of Devices containing heterogenous components
A device may contain a central processing unit (CPU), as well as heterogeneous acceleration components (such as GPUs, NPUs and TPUs) from different suppliers.

These components can be used to speed up processing or assist with AI inference.
Trustworthiness assessment of the device requires trust in all of these components.
However, due to business concerns such as scalability, complexity and cost of infrastructure, the Verifier for each type of component may be deployed separately by each vendor.

When these Verifiers operate together, they must interact with each other, understand the topology and interoperate using standardised protocols.
For instance, they may need to exchange partial Evidence relating to the relevant component or partial Attestation Results for it.

Attester: A Device having multiple components

Relying Party: An entity which is making trust decisions for such an Attester

Verifier 1: Verifier to verify the status of the CPU

Verifier 2: Verifier to verify the status of the acceleration component

## Verification of Workloads operating in Confidential Computing environment

As organisations move more workloads into untrusted or shared environments, Confidential Computing is becoming increasingly important.
In such a system, an application or workload (which could be an AI model, database process or financial service, for example) is executed inside a TEE-protected virtual machine (VM).
When the workload starts, the TEE can generate a cryptographic attestation report providing:

1. The workload is running on a platform with a known state.
2. The workload is running the correct application.

The platform is often built by an independent TEE vendor, while the workloads are deployed by workload owners from different parts of the supply chain.

Verification of Attestation for such a system requires independent, yet mutually coordinated, verification of: Platform claims appraised by a Platform Verifier and Workload claims appraised by a Workload Verifier.

Attester: A layered Attester containing a platform and a workload running in a CC environment

Relying Party: An entity which is making trust decisions, such as a key release to a workload

Verifier 1: Verifier to verify the status of the workload

Verifier 2: Verifier to verify the status of the CC envrionment


# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and concepts defined by the RATS architecture. For a complete glossary, see {{Section 4 of -rats-arch}}.

Specifically this document heavily uses the terms Layered Attester {{Section 3.2 of -rats-arch}} and Composite Device {{Section 3.3 of -rats-arch}}

## Glossary
{: #sec-glossary }

This document uses the following terms:

Composite Attester:

: A Composite Attester is either a Composite Device or a Layered Attester or any composition involving a combination of one or more Composite Devices or Layered Attesters.

Component Attester:

: A Component Attester is a single Attester of a Composite Attester.
For this document, a Component Attester is an entity which produces a single Evidence which can be appraised by a Component Verifier.

Composite Evidence:

: Evidence produced by a Composite Attester.
Also referred to as CE in the document.

Lead Verifier:

: A Verifier which acts as a main Verifier to receive Composite Evidence from a Composite Attester in a Hierarchical pattern {{sec-lead-verifier}}.
Also referred to as LV in the document.

Component Verifier:
: A Verifier which is responsible for the Verification of one single component or a layer.
Also referred to as CV in the document.

Partial Attestation Results:
: Attestation Results produced by a Component Verifier, which contains partial results from atleast one or more Component Attesters.

Aggregated Attestation Results:
: An Aggregated Attestation Results (AAR) refers to a collection of Attestation Results produced upon completion of appraisal of a Composite Attester.

# Multi Verifier topological patterns
{: #sec-multi-verifier }

A Composite Attester has multiple Component Attesters. Each Attester requires a different set of Verifiers. Hence multiple Verifiers collaborate to appraise a Composite Attester.

## Hierarchical Pattern {#sec-lead-verifier}
Figure below shows the block diagram of a Hierarchical Pattern.

~~~ aasvg
                                  +----------+
                                  |          |             +-----------+
                                  |          |             |           |
                                  |          | Evidence 1  |           |
                                  |          +------------>+ Verifier 1|
                                  |          |             |           |
                                  |          +<------------+           |
                                  |          |   AR 1      +-----------+
                                  |          |
+-----------+  Composite Evidence |          |
|           +-------------------->|          | Evidence 2  +-----------+
|  Attester |                     | Lead     +------------>+           |
|   or      |  Aggregated         | Verifier |             |           |
|  RP       |<--------------------+          |             | Verifier 2|
+-----------+  Attestation Result |          +<------------+           |
                 (AAR)            |          |  AR 2       |           |
                                  |          |             +-----+-----+
                                  |          |                   |
                                  |          |                   |
                                  |          |                   .
                                  |          |                   |
                                  |          |                   |
                                  |          |                   |
                                  |          | Evidence N  +-----+-----+
                                  |          +------------>+           |
                                  |          |             |           |
                                  |          +<------------+ Verifier N|
                                  |          | AR N        |           |
                                  |          |             |           |
                                  |          |             +-----------+
                                  +----------+
~~~
{: #fig-h-pattern title="Hierarchical Pattern"}

The following sub-sections describe the various roles that exist in this pattern.

### Lead Verifier

In this topological pattern, there is an Entity known as Lead Verifier.

Lead Verifier is the central entity in communication with the Attester (directly in passport model or indirectly via the Relying Party in background-check model).
It receives Attestation Evidence from a Composite Attester.
If the Composite Attestation Evidence is signed, then it validates the integrity of the Evidence by validating the signature.
If signature verification fails, the Verification is terminated.
Otherwise it performs the following steps.

* Lead Verifier has the required knowledge to break down the Composite Evidence into individual Component Evidence. It decodes the Composite Evidence to extract the Component Attester Evidence. This may lead to "N" Evidence, one for each Component Attester.

* Lead Verifier delegates each Component Attester Evidence to its own Component Verifier (CV) and receives Component Attester Attestation Results also known as Partial Attestation Results after successful Appraisal of Evidence.
There are many protocols to determine how a Lead Verifier can select the Component Verifiers.
This document does not mandate any specific protocol for determining the Component Verifiers

* Once the Lead Verifier receives Partial Attestation Results from all the Verifiers, it combines the results from each Verifier to construct a Aggregated Attestation Results (AAR). The Lead verifier may apply its own policies and also add extra claims as part of its appraisal.

* Lead Verifier conveys the AAR to the Attester (in Passport model) or to the Relying Party (in background check model).

The overall verdict may be dependent on the Appraisal Policy of the Lead Verifier.

In certain topologies, it is possible that only the Composite Evidence is signed to provide the overall integrity, while the individual Component Attester Evidence (example Evidence 1) is not protected. In such cases, the Lead Verifer upon processing of Composite Evidence may wrap the Component Attester Evidence (example Evidence 1) in a signed Conceptual Message Wrapper (CMW), and send it to each Verifier (example Verifier 1).


### Component Verifier

The role of a Component Verifier is to receive Component Evidence from the Lead Verifier and produce Partial Attestation Results to the Lead Verifier.

### Trust Relationships

In this topology the Lead Verifier is fully trusted by Component Verifiers (example Verifier 1).
Each Component Verifiers are provisioned with the Trust Anchors (see {{-trust-anchors}}) for the Lead Verifier.

Also, each of the Component Verifier is fully trusted by the Lead Verifier.
Lead Verifier is provisioned with the Trust Anchors (see {{-trust-anchors}}) for Verifier 1..N.

## Cascaded Pattern {#sec-verifier-cascade}

Figure below shows the block diagram of a Cascaded Pattern.

~~~ aasvg
                               +-----------+          +-----------+             +-----------+
+--------+                     |           |          |           |             |           |
|        |  Composite Evidence |           |  (CE)    |           |   (CE)      |           |
|        +-------------------->+           +--------->+           +------------>+           |
|        |     (CE)            |           |Partial AR|           | Partial AR  |           |
|Attester|                     | Verifier 1|          | Verifier 2|             | Verifier N|
|  or    |  Aggregated         |           |          |           |             |           |
| RP     +<--------------------+           +<---------+           +<------------+           |
+--------+ Attestation Results |           |  (AAR)   |           |  (AAR)      |           |
              (AAR)            |           |          |           |             |           |
                               +-----------+          +-----------+             +-----------+
~~~
{: #fig-c-pattern title="Cascaded Pattern"}

In this topological pattern, the Attestation Verification happens in sequence. Verifiers are cascaded to perform the Attestation Appraisal.
Each Verifier in the chain has the knowledge to derive or extract the Component Evidence, which it can appraise, from the Composite Evidence.

Attester may send the Composite Evidence(CE) to any of the Verifier (directly in the passport model, or indirectly via the Relying Party in the background-check model). The Verifier which processes the Composite Evidence, Verifies the signature on the Evidence, if present. It extracts the
Component Evidence from the Composite Evidence, performs Appraisal of the Component Attester whose Reference Values and Endorsements are in its database. Once the appraisal is complete, it forwards the Composite Evidence and Partial Attestation Results to the subsequent Verifier.

The process is repeated, until the entire appraisal is complete. The last Verifier, i.e. Verifier-N, completes its Appraisal of the Component Evidence, that it can appraise. It has now all the Partial Attestation Results and creates the Aggregated Attestation Results(AAR). It returns
the AAR to the N-1 Verifier (from where it received the Composite Evidence and Partial AR). The process is repeated, i.e. AAR is returned in the chain until the Verifier, which recieved the initial Composite Evidence is reached. At this point in time the Aggregated Attestation Results are signed and the AAR is sent to the Attester (in Passport Model) or Relying Party (in background check model).

As shown in the picture, the Partial Attestation Results and Composite Evidence is transmitted to a chain of Verifier, till the Appraisal is complete.
Upon completion, the last Verifier in the chain combines the incoming Partial Attestation Results, combines the results from it own Evidence Appraisal and passes the Aggregated Attestation Results to the Verifier from which it receives Composite Evidence.

There are many protocols to determine how a Verifier can select the next Verifier to route the CE.
This document does not mandate any specific protocol for determining the Verifiers in cascade.

### Trust Relationships

### Verifiers
In the cascaded pattern, the communicating Verifiers fully trust each other. Each Verifier has the trust anchor for the Verifier it is communicating to (i.e. either sending information or receiving information). This prevents man in the middle attack for the partial Attestation Results received by a Verifier or a Aggregated Attestation Results (AAR) which it receives in the return path.

### Relying Party and Verifiers
In the cascaded pattern, the RP may communicate with any Verifier and thus receive its Attestation Results. Hence RP fully trusts all the Verifiers.

## Hybrid Pattern

In a particular deployment, there is a possibility that the two models presented above can be combined to produce a hybrid pattern. For example Verifier 2 in the Cascaded Pattern becomes the Lead Verifier for the remaining Verifers from 3, to N.

# Freshness
The Verifier needs to ensure that the claims included in the Evidence reflect the latest state of the Attester. As per RATS Architecture, the recommended freshness is ascertained using either Synchronised Clocks, Epoch IDs, or nonce, embedded in the Evidence.
In the case of Hierarchical Pattern, the Verification of Freshness should be checked by the Lead Verifier.

In the Cascaded Pattern, the freshness is always checked by the first Verifier in communication with either the Attester (Passport Model) or Relying Party (Background Check Model).

# Security Considerations

The Verifier is effectively part of the Attesters' and Relying Parties' trusted computing base (TCB).  When multiple Verifiers coordinate to conduct appraisal, it leads to larger TCB and hence more attack surface. Any mistake in the appraisal procedure conducted by one or more Verifiers could lead to severe security implications, such as incorrect Attestation Result of a component or a composition to the Relying party. This section details the security threats and mitigation strategies specific to the multi-verifier topologies described in this document. In addition to the considerations herein, Verifiers MUST follow the guidance detailed in the Security and Privacy considerations of a RATS Verifier as detailed in {{Section 11 of -corim}} and the RATS Architecture {{Section 11 and Section 12 of -rats-arch}}.

## Adversarial Model
The security analysis in this section assumes that attackers may:

1. Eavesdrop on any communication channel between Verifiers.

2. Inject, modify, replay, or delay messages traversing the network.

3. Compromise one or more Verifiers in the ecosystem, attempting to leak sensitive information (e.g., Evidence, Reference Values) or manipulate Attestation Results.

4. Perform Man-in-the-Middle (MitM) attacks between any two communicating entities.

The system is designed to be resilient under the assumption that the cryptographic keys used for signing Evidence and Attestation Results (by authentic entities) are not compromised.

## General Considerations

All communications between entities (Attester-Verifier, Verifier-Verifier, Verifier-RP) MUST be secured using mutually authenticated, confidential, and integrity-protected channels (e.g., TLS).

It is recommended that any two verifiers establishing a communication channel perform mutual attestation before exchanging  any attestation messages.

## Security for Topological Patterns

### Hierarchical Pattern

The hierarchical pattern introduces a central trust entity, the Lead Verifier (LV). The security of the entire system relies on the integrity and correct operation of the LV.

#### Threats and Mitigations

##### LV Compromise

**Threat:** A compromised LV can orchestrate attacks, such as approving malicious attestations, wrongly aggregating attestation results or leaking sensitive evidence. This is a single point of failure from a trust perspective.

**Mitigation:** The LV MUST be hardened and operate and store its Keys in a secure environment. Its operation SHOULD be auditable.
Component Verifiers should be made available suitable trust anchors so that they can establish required trust in the authority of the LV.

##### Communication Security (LV <-> CV)

**Threat:** Eavesdropping or manipulation of evidence/results in transit.

**Mitigation:** All communications between the LV and CVs MUST be mutually authenticated and confidential (e.g., using TLS with client authentication). This ensures integrity, confidentiality, and authenticity of the messages exchanged between the Verifiers.

##### Evidence Integrity and Origin Authentication (LV -> CV)

**Threat:** The LV could forward manipulated evidence to a CV, or an attacker could inject fake evidence.

**Mitigation:** The conceptual message containing the Component Evidence MUST be integrity-protected and authenticated. If the Component Evidence is natively signed by the Component Attester at origin, the CV can verify it directly. If the Component Evidence lacks inherent signatures (e.g., in UCCS), the LV MUST sign the Component Evidence using a key that the CV trusts. This prevents any on-path attacker from altering the Component Evidence.

##### Results Integrity and Origin Authentication (CV -> LV)

**Threat:** Partial Attestation Results could be manipulated in transit or forged by a malicious CV.

**Mitigation:** Each Partial Attestation Result MUST be digitally signed by the CV.
 LV should maintain a list of trust anchors for the CV's it communicates with.
The LV MUST validate the signature using the required trust anchor for the CV, before adding the Partial Attestation Results to the Aggregated Attestation Results.


##### Replay Attacks

**Threat:** An adversary Component Verifier replays old Evidence or Attestation Results.

**Mitigation:** The LV is responsible for enforcing freshness (via nonces, epochs, or timestamps). This freshness value MUST be propagated to CVs and back to the LV, to ensure final AR can be validated against the original challenge.

### Cascaded Pattern

The cascaded pattern distributes trust but requires each Verifier in the chain to be trusted to correctly handle and forward  Attestation messages. The chain's security is only as strong as its weakest link.


#### Threats and Mitigations

##### Verifier Compromise

**Threat:** Any compromised Verifier in the chain can block, delay, or manipulate the attestation process. It can inject false partial results, drop evidence, or leak sensitive information.

**Mitigation:** Relying Parties and Verifiers MUST be configured with strict trust policies defining the allowed paths and trusted Verifiers. Operations should be logged for auditability.

##### Communication Security

**Threat:** Eavesdropping or manipulation of evidence and results between Verifiers.

**Mitigation:** Each hop between Verifiers MUST be secured with mutually authenticated and confidential channels (e.g., TLS with client authentication).

##### Evidence and Results Protection

**Threat:** Lack of end-to-end security allows intermediate Verifiers to manipulate evidence or results that are not intended for them to appraise.

**Mitigation:** End-to-end integrity protection is RECOMMENDED. The Composite Evidence should be signed by the Attester. Partial and Aggregated Attestation Results SHOULD be signed by the Verifier that generated them. This allows subsequent Verifiers and the Relying Party to verify that results have not been tampered with by intermediate nodes.

##### Replay Attacks

**Threat:** An adversary replays old Evidence or Attestation Results.

**Mitigation:** The first Verifier in the chain (the one receiving evidence from the Attester/RP) is responsible for enforcing freshness (via nonces, epochs, or timestamps) for the entire cascade. This freshness value MUST be propagated with the Evidence and Results through the chain so the final AR can be validated against the original challenge.

### Security of the Hybrid Pattern

As the hybrid pattern is the composition of  hierarchical pattern and cascade pattern, all the threats and mitigations that are applicable for these two patterns are also applicable for the general hybrid pattern.



# Privacy Considerations

The appraisal of a Composite Attester requires exchange of attestation related messages, for example, partial Evidence and partial Attestation Results, among multiple Verifiers. This can potentially leak sensitive information about the Attester's configuration , identities and the nature of composition.

- Minimization: Attesters should only generate Evidence that is strictly necessary for the appraisal policy. Verifiers should only request necessary claims.
- Confidentiality: Encryption should be used to prevent unauthorized parties (including other Verifiers in the hierarchy or cascade) from accessing sensitive Evidence. This is crucial in multi-tenant environments.
- Policy Handling: Verifiers should be careful not to leak their internal appraisal policies (e.g., through error messages or timing side channels) when communicating with other Verifiers or Attesters, as this information could be exploited by an attacker to manipulate appraisal.

# IANA Considerations


# Acknowledgments
{:numbered="false"}

The authors would like to thank
Simon Frost
and
Usama Sardar
for their reviews and suggestions.
