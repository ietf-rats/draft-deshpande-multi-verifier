---
title: Remote Attestation with Multiple Verifiers
abbrev: RATS Many-Verifiers
docname: draft-deshpande-rats-multi-verifier-latest
date: {DATE}
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
 -  name: Thomas Fossati
    organization: Linaro
    email: Thomas.Fossati@linaro.org

 -  name: Thanassis Giannetsos
    organization: UBITECH Ltd.
    email: agiannetsos@ubitech.eu



informative:
  RFC9334: rats-arch
  I-D.draft-ietf-rats-corim: corim

--- abstract

IETF RATS Architecture, defines the key role of a Verifier.  In a complex system, this role needs to be performed by multiple Verfiers coordinating together to assess the full trustworthiness of an Attester. This document focuses on various topological patterns for a multiple Verifier system.

--- middle

# Introduction

A Verifier plays a central role in any Remote Attestation System. A Verifier appraises the evidence about the Attester and produces Attestation Results, which are essentially a verdict of attestation. The results are consumed by the Relying Party to conclude the trustworthiness of the Attester, before making any critical decisions about the Attester, such as admitting it to the network or releasing confidential resources to it.
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

: A Component Attester is a single Attester of a Composite Attester. For this document, a Component Attester is an entity which produces a single Evidence which can be appraised by a Verifier.

Composite Evidence:

: Evidence produced by a Composite Attester.

Lead Verifier:

: A Verifier which acts as a Main Verifier to receive Composite Evidence from a Composite Attester.

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
It receives Attestation Evidence from a Composite Attester. If the Composite Attestation Evidence is signed, then it validates the integrity of the Evidence by validating the signature. If signature verification fails, the Verification is terminated. Otherwise it performs the following steps.

* Lead Verifier has the knowledge about the overall structure of the Composite Evidence. It decodes the Composite Evidence to extract the Component Attester Evidence. This may lead to "N" Evidence, one for each Component Attester.

* Lead Verifier delegates each Component Attester Evidence to their own Verifier and receives Component Attester Attestation Results after successful Appraisal of Evidence.

* Once the Lead Verifier receives Attestation Results from all the Verifiers, it combines the results from each Verifier to construct a Aggregated Attestation Results (AAR). The Lead verifier may apply its own policies and also add extra claims as part of its appraisal.

* Lead Verifier conveys the AAR to the Attester (in Passport model) or to the Relying Party (in background check model).

The overall verdict may be dependent on the Appraisal Policy of the Lead Verifier.

In certain topologies, it is possible that only the Composite Evidence is signed to provide the overall integrity, while the individual Component Attester Evidence (example Evidence 1) is not protected. In such cases, the Lead Verifer upon processing of Composite Evidence may wrap the Component Attester Evidence (example Evidence 1) in a signed Conceptual Message Wrapper (CMW), and send it to each Verifier (example Verifier 1).

### Verifier for Component Attester

The role of a Component Attester Verifier is to receive Component Attester Evidence from the lead Verifier and produce Attestation Results to the Lead Verifier.

### Trust Relationships

In this topology the Lead Verifier is fully trusted by Component Attester Verifiers (example Verifier 1).

Also, each of the Component Attester Verifier is fully trusted by the Lead Verifier. Lead Verifier is provisioned with the Trust Anchors for Verifier 1..N.

## Cascaded Pattern {: #sec-verifier-cascade }

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

In this topological pattern, the Attestation Verification happens in sequence. Verifiers are cascaded to perform the Attestation Appraisal. Each Verifier in the chain possess the knowledge of the entire Composite Attester topology.

Attester may send the Composite Evidence(CE) to any of the Verifier (directly in the passport model, or indirectly via the Relying Party in the background-check model). The Verifier which processes the Composite Evidence, Verifies the signature on the Evidence, if present. It decodes the Composite Evidence performs Appraisal of the Component Attester whose Reference Values and Endorsements are in its database. Once the appraisal is complete, it forwards the Composite Evidence and partial Attestation Results to the subsequent Verifier.

The process is repeated, until the entire appraisal is complete. The last Verifier, i.e. Verifier-N, completes its Appraisal of the Component Attester Evidence and returns the complete Attestation Results to the N-1 Verifier, which passed Evidence to it. The N-1 Verifier then simply passes the Aggregated Attestation Results(AAR) from where it received its Combined Evidence. Alternatively, it may also modify the AAR based on the inspection of received AAR. For example, it may add its own Verifier Added Claims (policy claims) and produce a new AAR. The process is repeated, until the Verifier, which recieved the initial Evidence is reached. At this point in time the Aggregated Attestation Results are signed and the Aggregated Attestation Results are sent to the Attester (in Passport Model) or Relying Party (in background check model).

As shown in the picture, the partial results and Combined Evidence is transmitted to a chain of Verifier, till the Appraisal is complete.
The Verifier combines the incoming partial results, combines the results from it own Evidence Appraisal and passes the Aggregated Attestation Results to the Verifier from which it receives Combined Evidence.


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

# Security and Privacy Considerations

The Verifier is effectively part of the Attesters' and Relying Parties' trusted computing base (TCB). Any mistake in the appraisal procedure conducted by the Verifier could have security implications. For Security and Privacy considerations while conducting appraisal procedure the Verifiers described in this document MUST follow the guidance detailed in Security and Privacy considerations of a RATS Verifier as detailed in {{Section 11 of -corim}}.

## Conceptual Message Protection

### Hierarchical Pattern

In this topology the Lead Verifier communicates with the Attester/RP and with other Verifiers.

The Security and Privacy consideration for the messages between the Lead Verifier and the Attester/RP follows the guidance provided in RATS Architecture {{Section 11 and Section 12 of -rats-arch}}.

The Lead Verifier conveys Component Attester Evidence to each of the sub-Verifiers and receives partial
Attestation Results from them.

1. The communication among the Verifiers should use secure channels, such as TLS. This ensures confidentiality, integrity and authenticity of the message exchanged between the Verifiers.

2. For integrity protection at the application layer, each partial Attestation Result Message is signed by a key known to the Lead Verfier.

3. The Composite Attester Evidence contains Component Attester Evidence, each having signature
from the Attesting Environments that generated it. This ascertains the authenticity and integrity protection
of individual Evidence exchanged between the Verifier. However there may be cases (for example UCCS), where the
individual Evidence is not signed. In such scenarios, the Lead Verifier may add its own signature using a private key whose
public key is known to the sub Verifiers.

4. Evidence might contain sensitive or confidential information, there might be a need for
confidentiality protection of the individual Evidence from Lead Verifier to sub Verifiers.
The Lead Verifier may choose to Encrypt the individual Evidence using the public Key of the Verifier it communicates.

If there isn't confidentiality protection of conceptual messages themselves,
the underlying conveyance protocol should provide these protections.

### Cascaded Pattern

In this pattern, the Composite Evidence is received by each Verifier in the chain. As a result,
the Security and Privacy consideration of Evidence between the Attester/RP and each of the Verifier follows the guidance provided in RATS Architecture {{Section 11 and Section 12 of -rats-arch}}.

Partial and Aggregated Attestation Results are exchanged among the Verifiers.
It is TBD how the Security and Privacy of these messages can be ascertained.
Few possible options are listed below.

1. All the Verifiers in the Eco-System share a common Trust Anchor Store. The Sender Ensures the Confidentiality and Integrity of the Partial/Aggregated AAR. The receiver Verifies the Confidentiality of these messages using the Private Keys in its database. It Verifies the authenticity and integrity of these messages using the Trust Anchor Store Public Keys.

2. The Verfier always communicates with a known Verifier in the chain. Hence it only maintains the trust roots for its communicating Verifier.

If there isn't confidentiality protection of conceptual messages themselves,
the underlying conveyance protocol should provide these protections

These and new options will be discussed further in the RATS Working Group.

# IANA Considerations


# Acknowledgements


<cref>TODO</cref>
