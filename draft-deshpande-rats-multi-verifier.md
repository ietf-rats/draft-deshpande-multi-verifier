---
title: Remote Attestation MultiVerifier
abbrev: RATS MultiVerifier
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


--- abstract

IETF RATS Architecture, defines the key role of a Verifier.  In a complex system, this role needs to be performed by multiple Verfiers coordinating together to assess the full trustworthiness of an Attester. This document focuses on various topological patterns for a multiple Verifier system.

--- middle

# Introduction

A Verifier plays a Central Role in any Remote Attestation System. A Verifier appraises the Attester and produces Attestation Results, which is essentially a verdict of attestation. The results are consumed by the Relying Party to conclude the trustworthiness of the Attester, prior to making any critical decisions about the Attester, such as admitting to network or releasing confidential resources to it.
Attesters can come in wide varieties of shape and form. For example Attesters can be endpoints (edge or IoT devices) or complex machines in the cloud. Composite Attester {{sec-glossary}}, generate Evidence that consists of multiple parts. For example, in data center servers, it is not uncommon for separate attesting environments (AE) to serve a subsection of the entire machine. One AE might measure and attest to what was booted on the main CPU, while another AE might measure and attest to what was booted machine's GPU. Throughout this document we use the term Component Attester {{sec-glossary}} to address the sub-entity or an individual layer which produces its own Evidence in a Composite Attester system.

A Verifier needs Reference Values and Endorsements from the supply chain actors (for example OEM/ODM) to conduct the appraisal of an Attester. Given the range of Component Attesters in a Composite Attester, it is possible that a single Verifier may not have all the capabilities or the information required to conduct the complete appraisal of the Composite Attester. In this case, multiple Verifiers need to collaborate to conclude the appraisal and produce the Attestation Results.


This document describes various topological patterns of multiple Verifiers that work in a coordinated manner to conduct appraisal of a Composite Attester to produce an Attestation Results.

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

# Security Considerations

<cref>TODO</cref>

# IANA Considerations

## CBOR Tag Registrations


# Acknowledgements


<cref>TODO</cref>
