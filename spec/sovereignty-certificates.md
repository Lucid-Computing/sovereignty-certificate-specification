# **Sovereignty Certificates (Draft)**

0 Introduction	3

[0.1 General	3](#0.1-general)

[0.2 The Challenge of Verifiable Sovereignty	4](#0.2-the-challenge-of-verifiable-sovereignty)

[0.3 Principles of Location-Based Attestation	4](#0.3-principles-of-location-based-attestation)

[1\. Scope	5](#heading=)

[2\. Normative References	5](#heading=)

[3\. Terms and Definitions	6](#heading=)

[4\. Architectural Framework	8](#heading=)

[4.1 Roles and Responsibilities	8](#heading=)

[4.2 Conceptual Message Flow	10](#heading=)

[5\. Attester Requirements	11](#heading=)

[5.1 Hardware Root of Trust (HRoT) Foundation	11](#heading=)

[5.2 Trusted Execution Environment (TEE) Capabilities	11](#heading=)

[5.3 Ephemeral Key Management and Binding	11](#heading=)

[5.4 Platform Evidence Collection	12](#heading=)

[5.5 Location Measurement Agent (LVA)	12](#heading=)

[5.6 Evidence Envelope Assembly and Signing	13](#heading=)

[6\. Verifier Requirements	13](#heading=)

[6.1 Secure Endpoint and Transport	13](#heading=)

[6.2 Evidence Parsing and Validation	13](#heading=)

[6.3 Location Computation	14](#heading=)

[6.4 Policy Engine	14](#heading=)

[6.5 Verifiable Attestation Result (VAR) Generation and Signing	15](#heading=)

[7\. Anchor and Endorser Requirements	15](#heading=)

[7.1 Anchor Fleet Requirements	15](#heading=)

[7.2 Endorser Requirements	16](#heading=)

[8\. Security Considerations	16](#heading=)

[8.1 Threat Model	16](#heading=)

[8.2 Cryptographic Controls	17](#heading=)

[8.3 Mitigations for Specific Attacks	17](#heading=)

[8.4 Operational Security	18](#heading=)

[Annex A (Normative) Sovereignty Certificate EAT Profile	19](#annex-a-\(normative\)-sovereignty-certificate-eat-profile)

[A.1 Profile Definition	19](#heading=)

[A.2 Claims Set	20](#heading=)

[Annex B (Informative) Example Implementation Workflow	22](#annex-b-\(informative\)-example-implementation-workflow)

[B.1 Pod Startup and Gating	22](#heading=)

[B.2 Attester Initialization	22](#heading=)

[B.3 Location Measurement	22](#heading=)

[B.4 Evidence Submission	22](#heading=)

[B.5 Verification and VAR Issuance	23](#heading=)

[B.6 Application Ungating	23](#heading=)

[B.7 Continuous Rotation	23](#heading=)

[Annex C (Informative) Crosswalk to Existing Standards	23](#annex-c-\(informative\)-crosswalk-to-existing-standards)

## 

## **0 Introduction**

This document was prepared by the Sovereignty Certificates Working Group, a collaborative initiative bringing together experts from cloud service providers, artificial intelligence laboratories, hardware manufacturers, and enterprises in regulated industries.

The working group was formed to address a critical gap in modern computing infrastructure: the absence of a standardized, cryptographically verifiable mechanism for proving the physical location of data processing workloads. As data residency and sovereignty requirements become increasingly stringent worldwide, relying solely on contractual assurances is no longer sufficient to address regulatory or business concerns. This specification provides a technical foundation for verifiable trust of important computing properties, starting with location of computing workloads.

This document specifies the architecture, protocols, and data formats for issuing and consuming Sovereignty Certificates. Its purpose is to enable interoperable, vendor-neutral implementations that provide strong assurances about the physical locality of computing environments.

While this document is structured in a manner similar to international standards to promote clarity, precision, and adoption, it is an industry-led specification. It is not an official standard promulgated by a national or international standards body such as ISO or IEC. The specification augments already existing Remote attestation architecture, given in the Remote ATtestation procedureS (RATS) Architecture (RFC 9334).

The publication of this specification is intended to foster an ecosystem of trusted, sovereign computing. The working group anticipates that this document will evolve and may contribute to future formal standardization efforts as the technology and market mature.

### **0.1 General** {#0.1-general}

The increasing globalization of data processing and cloud computing, coupled with the proliferation of stringent data residency and sovereignty regulations such as the EU’s General Data Protection Regulation (GDPR), Cybersecurity Act and Cloud Certification Scheme (EUCS), as well as sectoral certification in healthcare and finance across jurisdictions, has created a critical need for organizations to possess verifiable proof of the physical location of their computing workloads.3 Existing methods for ensuring data residency when using cloud computing rely on contractual agreements, operational policies, and manual audits, which provide assurances but not cryptographic, machine-verifiable proof.[^1] This gap exposes organizations to significant compliance risks and security vulnerabilities.

This document specifies a framework for establishing, implementing, maintaining, and continually improving a system for generating and appraising "Sovereignty Certificates."[^2] These certificates are cryptographic credentials that provide verifiable, unforgeable proof of the physical location of a computing device, such as a Central Processing Unit (CPU) or Graphics Processing Unit (GPU), at a specific point in time.3 By defining a unified industry standard, this document aims to enable a future where any organization can cryptographically prove the jurisdiction in which its computing workloads are processed, fostering a new level of trust and security in the digital infrastructure.

### **0.2 The Challenge of Verifiable Sovereignty** {#0.2-the-challenge-of-verifiable-sovereignty}

The absence of a standardized method to cryptographically prove the physical location of data and applications presents two fundamental problems. Firstly, it creates significant compliance risk. Businesses in regulated industries like finance, healthcare, defense, and government may face heavy fines and reputational damage for failing to comply with data residency laws, as they cannot truly verify the sovereignty claims made by their service providers.3 Secondly, in specific scenarios, it may even introduce security vulnerabilities. Without a mechanism for continuous, automated verification of the physical location and integrity of hardware (as described by RATS), processing devices are unable to detect unauthorized physical movement or theft, thereby maintaining the device within a secure boundary and reducing the risk of undetected tampering or replacement.,3

The current market is characterized by a fragmented landscape of proprietary solutions, leaving customers without a simple, trustworthy, and interoperable method to verify that their data remains within a specific jurisdiction. This standard addresses this challenge by creating the technical foundation for a zero-trust computing environment where trust is not assumed based on network location or contractual promises, but is instead established through continuous cryptographic verification.3

### **0.3 Principles of Location-Based Attestation** {#0.3-principles-of-location-based-attestation}

This document introduces a novel approach to location verification that is founded on the principles of physics and extends the established practice of remote attestation. The core technical principle is "Ping-Based Location Attestation," which integrates physical geolocation measurements directly into the cryptographic proof of a device's identity and trustworthiness of the system state attested via RATS.

While other location methods exist—such as Global Navigation Satellite Systems (GNSS) or IP-based geolocation databases—they generally rely on the device honestly reporting its position or on mutable database entries. In a Zero Trust model where the infrastructure provider may be adversarial (see Threat Model 8.1), these signals can be easily spoofed or tunneled. This standard specifically defines 'Ping-Based Location Verification' because it provides a hardware-backed proof bounded by the immutable speed of light, rather than trusting external signals.

The process leverages a Hardware Root of Trust (HRoT) within a Trusted Execution Environment (TEE) to conduct a remote attestation challenge-response protocol with multiple, geographically distributed verifiers known as Anchors. The key innovation lies in precisely measuring the round-trip time (RTT) of this cryptographic exchange. Because the signals carrying the protocol messages cannot travel faster than the speed of light (c), the measured RTT provides a physical, upper-bound constraint on the device's distance from each Anchor. By performing this measurement with multiple Anchors at known locations, the system can use multilateration to calculate a bounded physical location for the device and confirm the legal jurisdiction in which it resides.5 This technique creates a hardware-backed proof of both a device's identity and its location, standing in sharp contrast to unreliable methods based on IP address databases or manual audits (which are described through Distance Error PDF/CDF), because the location proof is hardware-backed and bounded by the speed of light.

## **1\. Scope**

This document specifies the requirements for an information security system to establish, implement, maintain, and continually improve a framework for generating and verifying Sovereignty Certificates.

The requirements set out in this document are generic and are intended to be applicable to all organizations participating in the sovereign computing supply chain, specifically:

* Hardware Manufacturers (fulfilling the role of Endorsers for platform trust);  
* Cloud Service Providers (fulfilling the roles of Verifiers and Anchor Operators); and  
* Enterprise Consumers (fulfilling the roles of Attesters and Relying Parties).3

**Specifically, this document defines:**  
a) An architectural model for location-based attestation, including roles, responsibilities, and conceptual messages, which conforms to the principles of the IETF Remote ATtestation procedureS (RATS) Architecture.  
b) Normative requirements for each architectural component, including the Attester, Verifier, Anchor, and Endorser.  
c) The process of Hardware-Anchored Location Verification. This document specifically defines 'Ping-Based Location Verification' as the normative method for measuring location using physical latency constraints, including requirements for measurement protocols, receipt generation, and location computation algorithms.  
d) A normative data format profile, based on the IETF Entity Attestation Token (EAT), for conveying location evidence in an Evidence Envelope and attestation results in a Verifiable Attestation Result (VAR).  
e) Security requirements and mitigations for threats against the authenticity and confidentiality of the Sovereignty Certificate system and its components.  
This document does not specify the implementation details of the underlying Hardware Root of Trust, the Trusted Execution Environment, or the network transport protocols used for communication between architectural components, except where necessary to ensure the security and interoperability of the location verification process.

## **2\. Normative References**

The following documents are referred to in the text in such a way that some or all of their content constitutes requirements of this document. For dated references, only the edition cited applies. For undated references, the latest edition of the referenced document (including any amendments) applies.

* **IETF RFC 9334, *Remote ATtestation procedureS (RATS) Architecture***: The architectural model, roles (Attester, Verifier, Relying Party, Endorser), and conceptual messages defined in this standard are founded upon the framework specified in RFC 9334\. This ensures alignment with a consensus-driven, generalized architecture for remote attestation, promoting interoperability and a common vocabulary.7  
* **IETF RFC 9711, *The Entity Attestation Token (EAT)***: The data structures for the Evidence Envelope and the Verifiable Attestation Result (VAR) SHALL conform to the token format specified in RFC 9711\. Annex A of this document defines a specific, normative profile of EAT for use in Sovereignty Certificates, ensuring a standardized and extensible format for claims.9  
* **ISO/IEC 19790, *Information security, cybersecurity and privacy protection – Security requirements for cryptographic modules***: All cryptographic operations specified within this framework, including digital signatures and key generation, SHALL be performed by cryptographic modules that conform to the security requirements specified in this standard. This provides a baseline assurance of the quality and security of the underlying cryptographic functions.11  
* **TCG TPM 2.0 Library Specification, Family "2.0"**: In implementations where a Trusted Platform Module (TPM) is utilized as the Hardware Root of Trust, the TPM SHALL conform to this specification. This ensures a standardized, secure, and interoperable foundation for platform measurement, reporting, and key protection.13  
* **TCG DICE Attestation Architecture**: In implementations where a Device Identifier Composition Engine (DICE) is utilized as the Hardware Root of Trust, it SHALL conform to this specification. This provides a standardized alternative for lightweight, hardware-anchored identity and attestation suitable for constrained environments.14  
* **ISO 8601, *Data elements and interchange formats – Information interchange – Representation of dates and times***: All representations of dates and times used within data structures defined by this standard, including timestamps in tokens and receipts, SHALL conform to ISO 8601 to ensure unambiguous and interoperable time representation across all components.17

The credibility and utility of this standard are contingent upon its integration within the broader ecosystem of information security and trusted computing. By normatively referencing these foundational standards, this document does not create an isolated, proprietary solution. Instead, it specifies the novel application of location as a form of evidence within established, widely adopted frameworks for architecture (RATS), data formats (EAT), and hardware trust (TCG). This approach ensures that implementers can leverage existing hardware, software, and expertise, facilitating adoption and guaranteeing interoperability across a diverse and competitive market of compliant solutions.3

## **3\. Terms and Definitions**

For the purposes of this document, the following terms and definitions apply. ISO and IEC maintain terminological databases for use in standardization at the following addresses:

* ISO Online browsing platform: available at [https://www.iso.org/obp](https://www.iso.org/obp)  
* IEC Electropedia: available at [https://www.electropedia.org/](https://www.electropedia.org/)

**3.1**  
**anchor**  
physically and network-secured computing entity at a known, fixed geographic location that participates in location measurement protocols by returning signed, timestamped receipts in response to probes from an *attester* (3.2)

**3.2**  
**attester**  
role performed by an entity (typically a device) whose evidence must be appraised

Note 1 to entry: An *attester’s* evidence must be appraised in order to infer the extent to which the attester is considered trustworthy, such as when deciding whether it is authorized to perform some operation.

\[SOURCE: RFC 9334\]

**3.3**  
**endorser**  
architectural role that serves as a root of trust for specific reference data

Note 1 to entry: The endorser is not necessarily a single entity.

Note 2 to entry: In a production ecosystem, this role SHOULD be federated, with different organizations responsible for different data scopes (e.g., hardware manufacturers endorsing chips, regulatory bodies endorsing Anchor Directories).

\[SOURCE: RFC 9334\]

**3.4**  
**evidence envelope**  
data structure that contains claims about the *attester's* (3.2) platform integrity and location measurements, signed by the *attester's* (3.2) ephemeral key which is cryptographically bound to the *hardware root of trust* (3.5)

**3.5**  
**hardware root of trust**  
**HRoT**  
component within a computing system that is inherently trusted to behave in an expected manner

Note 1 to entry: The hardware root of trust serves as the immutable foundation for the chain of trust for measurement, storage, and reporting.

**3.6 ping-based location verification**  
process of determining an *attester's* (3.2) geographic location by measuring the round-trip time of a challenge-response protocol between the *attester* (3.2) and multiple *anchors* (3.1).

Note 1 to entry: Ping-based location verification uses the physical constraint of the speed of light in the transmission medium (or a conservative upper bound thereof) to calculate a bounded location estimate.

**3.7 sovereignty certificate**  
*verifiable attestation result* (3.9) that contains cryptographically verified claims about an *attester's* (3.2) identity, integrity and physical location, asserting its presence within a specific geographic or jurisdictional boundary

**3.8 trusted execution environment**  
**TEE**  
secure area inside a main processor that ensures the confidentiality and integrity of the code and data loaded within it, providing hardware-enforced isolation from the host operating system and other applications

**3.9 verifiable attestation result**  
**VAR**  
data structure,  that contains the *verifier's* (3.10) appraisal of the evidence and asserts claims about the *attester's* (3.2) trustworthiness, including its computed location, radius and confidence level

Note 1 to entry: The verifiable attestation result shall conform to the entity attestation token profile defined in Annex A, issued and signed by a verifier (3.10).

**3.10 verifier**  
architectural role embodied by a trusted service that receives an *evidence envelope* (3.4) from an *attester* (3.2), appraises it against policies and reference values, computes the *attester's* (3.2) location, and issues a *verifiable attestation result* (3.9).

\[SOURCE: RFC 9334\]

**3.11 reference value rrovider**  
**RVP**  
**a**rchitectural role embodied by a trusted entity (e.g. software vendor or supply chain authority) that produces reference values for platform integrity and trusted anchor directories

\[SOURCE: RFC 9334\]

## **4\. Architectural Framework**

### **4.1 Roles and Responsibilities**

The architecture for generating and verifying Sovereignty Certificates SHALL conform to the roles and conceptual model defined in IETF RFC 9334, *Remote ATtestation procedureS (RATS) Architecture*.7 This standard specifies the unique functions and normative requirements for these roles in the specific context of location verification. The primary roles are the Attester, Verifier, Anchor, Endorser, and Relying Party.

The deliberate decoupling of these roles is a foundational principle of this architecture. For instance, the separation of the Endorser (the source of ground truth for platform integrity and Anchor identity) from the Verifier (the decision engine) enhances the overall security and resilience of the system. A compromise of a Verifier does not automatically compromise the system's trust anchors, as the Verifier can only appraise evidence against the signed endorsements it receives. This modularity also fosters an open and competitive ecosystem, as envisioned in the initiative's goals, allowing different organizations to specialize in providing distinct services (e.g., hardware vendors as Endorsers, cloud providers as Verifiers) without creating a monolithic, single-vendor solution.3

#### **4.1.1 Attester** 

The Attester is the entity whose location and integrity are being verified. This role is embodied by a 'Sovereign Attestation Agent' executing within the same trust boundary as the application workload. In containerized environments, this is typically deployed as a sidecar container. In Virtual Machine (VM) environments, this is typically deployed as a system service, daemon, or background process running within the Guest Operating System. 

Regardless of the deployment model, the Attester is responsible for:

a) Generating an ephemeral key pair for each attestation cycle.  
b) Interacting with the platform's Hardware Root of Trust (HRoT) to obtain a signed measurement (quote) derived from the HRoT's protected measurements of the platform's state, cryptographically binding the ephemeral public key to a  non-falsifiable measurement.  
c) Obtaining signed location measurement receipts, either by directly probing the Anchor Fleet or by querying a local trusted proxy.  
d) Assembling the platform quote and location measurement receipts into a signed Evidence Envelope.  
e) Submitting, or queueing for submission, the Evidence Envelope to the Verifier for appraisal.

#### **4.1.2 Verifier**

The Verifier is the trusted service that appraises the Evidence provided by the Attester. The Verifier is responsible for:  
a) Receiving and parsing the Evidence Envelope.  
b) Validating the authenticity and integrity of the platform quote by verifying its signature chain against trust anchors provided by the Endorser.  
c) Validating the authenticity of the location measurement receipts by verifying their signatures against the Anchor public keys provided by the Endorser.  
d) Computing the Attester's geographic location: radia and an intersecting polygon based on the timing data in the receipts.  
e) Evaluating the appraised evidence and computed location against a set of configurable policies.  
f) Issuing a signed Verifiable Attestation Result (VAR) upon successful policy evaluation.

#### **4.1.3 Anchor Fleet**

The Anchor Fleet is a globally distributed set of servers that act as fixed reference points for the location measurement protocol. Each Anchor is responsible for:  
a) Maintaining a secure, known, and stable physical and network location.  
b) Responding to measurement probes from Attesters.  
c) Generating and signing a receipt containing a high-precision timestamp and other protocol data for each valid probe.

#### **4.1.4 Endorser and Reference Value Provider (RVP)**

The Endorser is the authoritative source of trust anchors and reference data. It is responsible for:  
a) Publishing a signed directory of all trusted Anchors, including their public keys and precise geographic coordinates.  
b) Publishing signed Reference Values for platform integrity, such as known-good measurements for firmware, bootloaders, and other components of the Trusted Computing Base (TCB).  
c) Securely managing the lifecycle of its own signing keys and the endorsements it produces.

#### **4.1.5 Relying Party**

The Relying Party is the ultimate consumer of the Sovereignty Certificate (VAR). It can be the application workload running alongside the Attester or a downstream service. The Relying Party is responsible for:  
a) Trusting the public key of one or more Verifiers.  
b) Obtaining the VAR, either by receiving it from the Attester (Passport Model) or by retrieving it directly from the Verifier (Background Check Model).  
c) Validating the signature on the VAR.  
d) Parsing the claims within the VAR (e.g., location, integrity status) and using them to make authorization decisions, such as granting access to sensitive data or enabling specific functionality. Note: While the Verifier validates the correctness of the claims (Appraisal), the Relying Party determines if those claims satisfy the specific access requirements of the workload (Authorization).

### **4.2 Conceptual Message Flow**

The interaction between the roles defined in 4.1 follows a defined sequence of conceptual message exchanges. This flow aligns with the challenge-response and uni-directional interaction models described in the IETF RATS reference interaction models.20

#### **4.2.1 Evidence Generation and Submission**

The process begins with the Attester, which initiates an attestation cycle.

1. **Initialization & Challenge:** The Attester contacts the Verifier to initiate the session. The Verifier returns a fresh nonce (for liveness) and the current signed Anchor Directory (obtained from the Endorser) to the Attester**.**  
2. **Measurement**: The Attester collects platform integrity evidence from the HRoT and performs location measurements by probing multiple Anchors from the directory.  
3. **Assembly**: The Attester assembles the platform evidence and the signed location receipts from the Anchors into a canonical Evidence Envelope data structure.  
4. **Submission**: The Attester signs the complete Evidence Envelope with its ephemeral private key and submits it to the Verifier's secure endpoint.5

#### **4.2.2 Verification and Policy Evaluation**

Upon receipt of the Evidence Envelope, the Verifier performs a comprehensive appraisal.

1. **Evidence Validation**: The Verifier first validates the signature on the envelope. It then fetches the necessary endorsements and reference values from the Endorser.  
2. **Platform Appraisal**: The Verifier validates the platform quote chain and compares the TCB measurements against the reference values. It critically verifies the binding of the Attester's ephemeral public key to the quote.  
3. **Location Appraisal**: The Verifier validates the signatures on all Anchor receipts and computes the Attester's location, radius, and confidence score.  
4. **Policy Evaluation**: The Verifier's policy engine evaluates all appraised data against its configured policies (e.g., allowed jurisdictions, maximum location radius, minimum TCB version).5

#### **4.2.3 Issuance and Consumption of Verifiable Attestation Results**

The final stage involves the creation and use of the Sovereignty Certificate.

1. **Issuance**: If the policy evaluation is successful, the Verifier generates a VAR containing the results of the appraisal, including the verified location claim. The Verifier signs the VAR with its private key and returns it to the requesting entity (either the Attester in the Passport model, or the Relying Party in the Background Check model)..  
2. **Consumption**: The Relying Party validates the VAR and uses the claims to gate functionality or authorize access. In the Passport model, the Attester presents the VAR to the Relying Party; in the Background Check model, the Relying Party already possesses the VAR from the previous step.5

## **5\. Attester Requirements**

### **5.1 Hardware Root of Trust (HRoT) Foundation**

The entire security guarantee of a Sovereignty Certificate originates from the physical hardware of the Attester. Therefore, a secure and non-spoofable foundation is paramount.

5.1.1 The Attester platform SHALL possess a Hardware Root of Trust (HRoT) that provides, at a minimum, a Root of Trust for Measurement (RTM) and a Root of Trust for Identity (RTI), as defined by the Trusted Computing Group (TCG).14 The RTM provides the capability to securely measure the first mutable code executed during the boot process, establishing the start of the chain of trust. The RTI provides a unique, cryptographically verifiable identity for the hardware.

5.1.2 The HRoT SHALL provide a mechanism to produce a cryptographically signed attestation quote that includes measurements of the platform's boot and runtime state.This mechanism SHALL conform to a published, industry-recognized specification for hardware-anchored attestation.

Valid implementations include, but are not limited to: a) TCG Standards: The TCG TPM 2.0 Library Specification or the TCG DICE Attestation Architecture. b) Confidential Computing Architectures: The hardware-enforced attestation reporting structures defined for AMD SEV-SNP, Intel TDX/SGX, or Arm CCA.

Regardless of the specific architecture, the mechanism MUST utilize a signing key that is protected by the hardware and inaccessible to the host operating system.

### Note: Future revisions of this standard may specify requirements for physical tamper-evidence mechanisms (e.g., epoxy potting, active mesh sensors, or tamper-evident seals) to further harden the HRoT against local physical attacks. Implementers are encouraged to consider physical hardening measures where the device is deployed in uncontrolled environments.

### **5.2 Trusted Execution Environment (TEE) Capabilities**

To protect the attestation process from a potentially compromised host operating system or other malicious software, the core logic of the Attester must execute in an isolated environment.

5.2.1 The Attester's evidence collection, location measurement, and evidence signing processes SHALL execute within a Trusted Execution Environment (TEE).3 The TEE SHALL provide hardware-enforced memory isolation and protection for its executing code and data, preventing inspection or modification by processes running outside the TEE, including those with higher privilege levels in the host OS.19

5.2.2 The TEE SHALL ensure the integrity of the Attester code executing within it and the confidentiality of all cryptographic materials, particularly the ephemeral private key, used during the attestation process.

### **5.3 Ephemeral Key Management and Binding**

To prevent replay attacks, where an attacker captures valid evidence and resubmits it later, each attestation must be unique and fresh. This is achieved by using a single-use key that is provably linked to a specific platform instance at a specific time verified by an external source (e.g., a Verifier-supplied nonce).

5.3.1 Upon initialization of an attestation cycle, the Attester SHALL generate a new, ephemeral asymmetric key pair.

5.3.2 The Attester SHALL create a cryptographic binding between the ephemeral public key and the platform's integrity measurement. This SHALL be achieved by creating a data structure containing the ephemeral public key and a freshness value (e.g., a nonce provided by the Verifier). A cryptographic hash of this data structure SHALL be placed into a specific register or data field (e.g., REPORTDATA in Intel TDX, UserData in AMD SEV-SNP) that is included in the signed attestation quote produced by the HRoT.5 This binding provides non-repudiable proof that the ephemeral key was generated by a genuine, measured platform instance and was intended for the current attestation session.

### **5.4 Platform Evidence Collection**

The Attester SHALL collect evidence of the platform's Trusted Computing Base (TCB) to prove its integrity. This evidence SHALL include, but is not limited to, firmware measurements, secure boot state, bootloader and kernel measurements, and digests of container images the software components comprising the Sovereign Workload (e.g., the application container image, the Attester agent image, and any security-critical dependencies).

### **5.5 Location Measurement Agent (LVA)**

The LVA is the component of the Attester responsible for executing the Ping-Based Location Verification protocol.

#### **5.5.1 Anchor Probing Protocols**

5.5.1.1 The Attester's LVA SHALL probe a set of trusted Anchors using one or more Networkand Transport layer protocols that are capable of precise round-trip time (RTT) measurement. Suitable protocols include, but are not limited to, ICMP, UDP and QUIC

5.5.1.2 The set of Anchors probed SHALL be selected to maximize the geometric quality of the resulting location calculation (i.e., minimizing Geometric Dilution of Precision, or GDOP).

Implementation Note: The Attester SHOULD prioritize:

- Proximity: Anchors with the lowest expected latency.  
- Azimuth Distribution: Anchors that are geographically distributed around the Attester (e.g., selecting Anchors in different cardinal directions) rather than clustered in a single sector.

#### **5.5.2 Receipt Collection and Validation**

5.5.2.1 For each probe, the LVA SHALL collect the corresponding signed, timestamped receipt returned by the Anchor.

5.5.2.2 The LVA SHOULD validate the signature on each receipt against the public keys provided in the Endorser's Anchor Directory before including it in the Evidence Envelope. This preliminary check helps to filter out invalid receipts early, reducing the payload sent to the Verifier.

### **5.6 Evidence Envelope Assembly and Signing**

5.6.1 The Attester SHALL assemble the collected platform evidence (5.4) and the validated location measurement receipts (5.5) into a canonical Evidence Envelope. This envelope SHALL conform to the normative EAT profile defined in Annex A.

5.6.2 The entire canonical representation of the Evidence Envelope SHALL be digitally signed using the ephemeral private key generated in 5.3.1. This signature ensures the integrity and authenticity of the complete evidence package.

## **6\. Verifier Requirements**

### **6.1 Secure Endpoint and Transport**

The Verifier acts as the central trust arbiter in the system and must protect the confidentiality and integrity of its communications. The Verifier SHALL expose a secure network endpoint for receiving Evidence Envelopes from Attesters. This endpoint SHALL use a strong, authenticated transport-layer security protocol, such as TLS 1.3, to protect data in transit.

### **6.2 Evidence Parsing and Validation**

The Verifier's primary function is the appraisal of evidence. This requires a rigorous, multi-step validation process.

#### **6.2.1 Platform Evidence Appraisal**

6.2.1.1 The Verifier SHALL parse the platform attestation quote from the Evidence Envelope and validate its cryptographic integrity. This includes verifying the signature on the quote and the entire certificate chain back to a trusted hardware vendor root certificate. These root certificates SHALL be obtained from a trusted Endorser.

6.2.1.2 The Verifier SHALL compare the TCB measurements contained within the quote against known-good reference values (e.g., golden measurements) obtained from the Endorser for the specified platform.

6.2.1.3 The Verifier SHALL perform a replay-prevention check by re-computing the cryptographic hash of the ephemeral public key and nonce (both extracted from the Evidence Envelope) and comparing the result with the value contained in the quote's REPORTDATA field (or equivalent). A mismatch SHALL result in the immediate rejection of the evidence.5 This step is critical as it confirms the binding established in 5.3.2 and ensures the evidence is fresh and not replayed from another session or machine.

#### **6.2.2 Location Measurement Appraisal**

The Verifier SHALL parse the array of location measurement receipts from the Evidence Envelope. For each receipt, it SHALL validate the digital signature against the corresponding Anchor's public key as published in the signed Anchor Directory obtained from the Endorser.5 Any receipt with an invalid signature SHALL be discarded.

### **6.3 Location Computation**

After validating the authenticity of the location receipts, the Verifier computes the Attester's physical location.

#### **6.3.1 Multilateration Algorithms**

6.3.1.1 The Verifier SHALL use a deterministic multilateration or multilocation algorithm to compute a feasible geographic region for the Attester. This computation SHALL be based on the known geographic coordinates of the Anchors and the RTT values derived from the timestamps in the validated receipts.

6.3.1.2 Physical Consistency Checks: The algorithm SHALL verify that the intersection of the computed location radii is non-empty. Violations of geometric constraints (e.g., the triangle inequality), where the measured distances from two Anchors precludes the existence of a single intersection point, SHALL be treated as evidence of network path manipulation (e.g., tunneling) and result in the rejection of the attestation.

6.3.2 Confidence and Radius Calculation

The Verifier SHALL calculate a geographic radius (in meters) and a confidence score (a value between 0 and 1\) for the computed location. The confidence score MUST be a function of the quality and consistency of the measurement data, including factors such as the number of successful probes, the geometric distribution of the anchors, and the statistical variance of the RTT measurements.

### **6.4 Policy Engine**

The Verifier translates the raw appraisal results into a business-relevant compliance decision using a policy engine.

Note: By evaluating compliance policies at the Verifier, the system produces a simplified artifact (VAR) that abstracts the complexity of geospatial computation from downstream Relying Parties.

#### **6.4.1 Policy Definition and Management**

The Verifier SHALL provide a secure interface for administrators to define, manage, and audit security and compliance policies. Policies SHALL be expressed in a structured, machine-readable format.

#### **6.4.2 Policy Evaluation Logic**

The Verifier's policy engine SHALL evaluate the appraised evidence and computed location against the defined policies. The policy language MUST support rules based on thresholds and allowlists, including, but not limited to: minimum\_anchors, maximum\_radius, minimum\_confidence, TCB\_floor (minimum acceptable TCB version), allowed container image digests, and allowed geographic regions or legal jurisdictions.5 The evaluation of these policies SHALL be deterministic.

### **6.5 Verifiable Attestation Result (VAR) Generation and Signing**

Upon successful policy evaluation, the Verifier mints the Sovereignty Certificate.

6.5.1 The Verifier SHALL generate a Verifiable Attestation Result (VAR) that conforms to the normative EAT profile defined in Annex A.

6.5.2 The VAR SHALL contain claims that assert the key outcomes of the appraisal, including the computed location, radius, confidence score, and a list of the policy identifiers that were successfully passed. It SHALL also include an expiration time (exp claim) to limit its validity period.

6.5.3 The entire canonical representation of the VAR SHALL be digitally signed by the Verifier's private key. The corresponding public key SHALL be made available to Relying Parties to enable them to validate the VAR's authenticity.

## **7\. Anchor and Endorser Requirements**

### **7.1 Anchor Fleet Requirements**

The trustworthiness of the location measurement depends directly on the integrity and reliability of the Anchor fleet.

#### **7.1.1 Physical and Network Security**

Anchors SHALL be deployed in physically secure facilities, such as Tier III or higher data centers, to protect against unauthorized access and tampering. Their network infrastructure SHALL be hardened to prevent IP address spoofing and to mitigate denial-of-service attacks.

#### **7.1.2 Time Synchronization**

All Anchors SHALL be synchronized to a reliable and precise time source. They SHOULD be synchronized to Coordinated Universal Time (UTC) to ensure consistent timestamping of Evidence receipts and audit logs.

Note: While the Ping-Based RTT measurement relies on the Anchor's local clock stability (differential timing) rather than absolute UTC, synchronization is required to correlate events across the Anchor Fleet and validate certificate freshness.

#### **7.1.3 Signed Receipt Generation**

Upon receiving a valid measurement probe from an Attester, an Anchor SHALL generate a receipt. This receipt SHALL contain, at a minimum:  
a) A high-precision timestamp indicating the time of receipt of the probe.  
b) The nonce or challenge value from the Attester's probe.  
c) A unique identifier for the Anchor.  
This entire receipt data structure SHALL be digitally signed with the Anchor's private key before being returned to the Attester.

#### **7.1.4 Anchor Integrity Monitoring**

The Anchor Fleet SHOULD implement a continuous peer-to-peer monitoring protocol. Anchors shall periodically measure the RTT to other trusted Anchors in the directory. If the measured RTT between two Anchors significantly deviates from the physical baseline expected from their registered coordinates (accounting for standard network jitter), the Anchors SHALL report a health warning to the Endorser, and the anomalous Anchor SHOULD be temporarily removed from the active directory.

### **7.2 Endorser Requirements**

The Endorser is the root of trust for reference data in the ecosystem. Its integrity is critical.

#### **7.2.1 Management of Reference Values**

The Endorser role is responsible for provisioning Reference Values, which serve as the 'Golden Measurements' for appraisal. These include: \* Platform Integrity Models: Allow-lists of valid TCB measurements (e.g., firmware/kernel hashes). \* Anchor Manifests: The authoritative list of trusted Anchors, their public keys, and certified locations.

#### **7.2.2 Publication of Anchor Directory and Keys**

The Endorser SHALL maintain and publish a canonical, signed directory of all trusted Anchors. For each Anchor, this directory SHALL include its unique identifier, its public key for signature verification, and its precise geographic coordinates (latitude and longitude).5

#### Implementation Note: To ensure interoperability with RATS-compliant Verifiers, this directory SHOULD be formatted as a Concise Reference Integrity Manifest (CoRIM). Because standard CoRIM definitions do not include geospatial claims, implementations SHALL utilize the \[Sovereignty Certificate CoRIM Profile\] (defined in \[Future/External Ref\]), which specifies the extension points for Anchor coordinates.

#### **7.2.3 Endorsement Signing and Lifecycle**

All data artifacts published by the Endorser, including the Anchor Directory and Reference Value bundles, SHALL be digitally signed with the Endorser's private key. The Endorser SHALL implement and follow a documented policy for the secure management and rotation of its signing keys.

## **8\. Security Considerations**

This clause specifies the security requirements necessary to ensure the integrity, authenticity, and confidentiality of the Sovereignty Certificate system. The requirements are derived from the threat model implied by the Quality Attribute Scenarios 5 and are aligned with information security management best practices.4

### **8.1 Threat Model**

The threat model for this system considers a sophisticated attacker whose goal is to obtain a valid Sovereignty Certificate for a workload that is either malicious (i.e., running on a compromised platform) or located outside of an authorized jurisdiction. The attacker is assumed to have control over the network between the Attester, Anchors, and Verifier, and may have administrative (root) privileges on the host operating system or hypervisor where the Attester's TEE resides. This explicitly includes the scenario of an untrusted cloud service provider (hyperscaler).. The HRoT and the TEE hardware and microcode are considered part of the trusted computing base and are assumed to be secure against software-based attacks.

The threat model currently assumes the attacker may have physical access to the device. While the TEE protects memory confidentiality, sophisticated physical attacks (e.g., bus probing, side-channel analysis) remain a residual risk.

### **8.2 Cryptographic Controls**

The security of this system is fundamentally dependent on the correct application of strong cryptography.

#### **8.2.1 Approved Algorithms and Key Lengths**

All cryptographic operations, including digital signatures, key generation, and hashing, SHALL use algorithms and key lengths that are recognized as secure by international standards bodies and are not considered deprecated. Implementations SHOULD conform to the guidance provided by bodies such as the US National Institute of Standards and Technology (NIST) or the European Union Agency for Cybersecurity (ENISA).24 For example, digital signatures SHOULD use ECDSA with NIST P-256 or stronger curves.

#### **8.2.2 Key Management Lifecycle**

The management of all cryptographic keys used within the system—including the Attester's ephemeral keys, Anchor signing keys, the Verifier signing key, and the Endorser signing key—SHALL be governed by a comprehensive key management policy. This policy SHALL address all phases of the key lifecycle: generation, distribution, storage, usage, and destruction. The principles outlined in ISO/IEC 27001:2022 Annex A Control 8.24 provide a suitable framework for such a policy.26 All long-lived private keys (Anchor, Verifier, Endorser) SHALL be protected by a hardware security module (HSM) conforming to ISO/IEC 19790\.11

### **8.3 Mitigations for Specific Attacks**

The architecture includes several specific controls to mitigate known attack vectors.

#### **8.3.1 Replay Attack Prevention**

The cryptographic binding of an ephemeral public key and a nonce into the HRoT attestation quote, as required in clause 5.3, is the primary mitigation against the replay of platform evidence. The Verifier's mandatory check of this binding, as specified in 6.2.1.3, ensures that captured evidence from a previous session cannot be successfully re-used.5

#### **8.3.2 Evidence Tampering**

The requirement in 5.6 that the entire Evidence Envelope be digitally signed by the Attester's ephemeral private key ensures the integrity of the evidence bundle. The Verifier's mandatory signature validation, as specified in 6.2, ensures that any in-transit modification of the evidence is detected and results in rejection.5

#### **8.3.3 Anchor Spoofing and Collusion**

The requirement in 6.2.2 that the Verifier only accept receipts from Anchors listed in the signed Anchor Directory from the Endorser prevents an attacker from impersonating a trusted Anchor. To mitigate the risk of collusion among a small set of malicious or compromised Anchors, Verifier policies SHOULD require measurements from a geographically diverse set of Anchors operated by multiple independent entities.5

#### **8.3.4 Network Topology Concealment  (e.g., VPNs)**

Attackers may utilize network tunnels or proxies to mask their true IP address or network topology.

* Impact: Since tunneling inherently adds latency (distance), it generally prevents an attacker from successfully claiming a location closer to an Anchor than they truly are. However, it typically results in Geometric Inconsistency (e.g., violating the Triangle Inequality), where the RTT-derived distances to different Anchors contradict each other.  
* Mitigation: The Verifier's location algorithm SHALL strictly enforce geometric consistency. If the intersection of the measured radii is empty or mathematically impossible, the attestation SHALL be rejected (Fail-Closed).

### **8.4 Operational Security**

#### **8.4.1 Secure Boot and Runtime Integrity**

The Attester platform SHALL implement a secure boot process. This process ensures that the initial code measured by the RTM is authentic and unmodified, establishing the integrity of the entire chain of trust for subsequent measurements.

#### **8.4.2 Token Rotation and Fail-Closed Mechanisms**

The Verifiable Attestation Result (VAR) is a short-lived credential. The Attester SHOULD proactively refresh the VAR at a frequency of no more than half its specified time-to-live (TTL). If a valid VAR expires, or if the Attester is unable to obtain a new one (due to policy failure, network outage, etc.), the Attester and its associated application workload SHALL enter a fail-closed state. In this state, the application MUST cease all sensitive operations and SHOULD signal a non-ready status to its orchestration platform or load balancer, or enforce a local firewall rule, preventing it from receiving new traffic or accessing protected resources.5

### **Table 8.1 — Threat Scenarios and Normative Mitigations**

The following table provides a mapping from identified threat scenarios to the specific, mandatory controls within this standard that are designed to mitigate them. This serves as a reference for implementers and auditors to verify that critical security risks are addressed by normative requirements.

| Threat Scenario (from 5) | Description | Normative Mitigation (Clause Reference) |
| :---- | :---- | :---- |
| Bound Identity Replay | An attacker replays a valid platform attestation quote from a different machine or a previous session to obtain a valid VAR for an unmeasured, malicious workload. | The Verifier SHALL validate the binding of the ephemeral public key and nonce to the platform quote's REPORTDATA field. **(5.3, 6.2.1.3)** |
| Tampered Envelope | An attacker on the network intercepts and modifies the contents of the Evidence Envelope in transit between the Attester and Verifier. | The Attester SHALL sign the entire canonical Evidence Envelope. The Verifier SHALL validate this signature before processing. **(5.6, 6.2)** |
| Anchor Spoofing | An attacker sets up a malicious server that impersonates a legitimate Anchor to provide false timing information to the Attester. | The Verifier SHALL only accept location receipts signed by Anchors whose public keys are present in the signed Anchor Directory from the Endorser. **(5.5.2, 6.2.2, 7.2.2)** |
| Policy Evasion via VPN | An Attester outside an allowed jurisdiction routes its traffic through a VPN or proxy located within the jurisdiction to falsify its location. | The Verifier's location computation algorithm SHALL incorporate checks for network path anomalies that are characteristic of tunneling. **(6.3.1.3, 8.3.4)** |
| Stale VAR Usage | An application continues to operate using a previously valid VAR after its expiration, potentially violating a newly enforced security policy. | The Attester and Relying Party SHALL enforce the VAR's expiration time. The system SHALL operate in a fail-closed mode if a valid VAR cannot be maintained. **(8.4.2)** |

## 

## **Annex A (Normative) Sovereignty Certificate EAT Profile** {#annex-a-(normative)-sovereignty-certificate-eat-profile}

### **A.1 Profile Definition**

This annex defines the "Sovereignty Certificate Profile" for the Entity Attestation Token (EAT), as specified in IETF RFC 9711\.9 All Evidence Envelopes and Verifiable Attestation Results (VARs) that claim conformance to this standard SHALL implement and adhere to this profile.

The profile identifier for this version of the standard SHALL be the URI: urn:scwg:profile:sovereignty-cert:v1. This identifier SHALL be included in the eat\_profile (key 39\) claim of every token.9

The claims set defined in this profile MAY be serialized as either:

1. A CBOR Web Token (CWT) protected by a COSE\_Sign1 structure; OR  
2. A JSON Web Token (JWT) protected by a JWS structure.

#### **A.1.1 CWT Serialization Requirements**

If CWT serialization is used, the implementation of CBOR encoding and COSE protection SHALL conform to the requirements of the "Constrained Device Standard Profile" defined in Section 6.10 of IETF RFC 9711\. This includes the use of definite-length encoding for all CBOR structures and the use of ECDSA with NIST P-256, P-384, or P-521 for signatures (COSE Algorithms ES256, ES384, or ES512).

#### **A.1.2 JWT Serialization Requirements** 

If JWT serialization is used, the implementation SHALL conform to RFC 7519\. Signatures SHALL be generated using ECDSA with NIST P-256, P-384, or P-521 curves (JWA Algorithms ES256, ES384, or ES512), in accordance with RFC 7518\.

*Implementation Note: While JWT is permitted for Evidence Envelopes to support broad interoperability, CWT is RECOMMENDED for the Attester role (Evidence generation). The use of CWT minimizes the complexity of parsing and serialization libraries required within the Trusted Execution Environment (TEE), thereby reducing the Trusted Computing Base (TCB) size and attack surface (see 5.2). Conversely, the Verifier is RECOMMENDED to support issuing VARs in JWT format to facilitate easier consumption by Relying Parties.*

### **A.2 Claims Set**

The following table defines the normative claims set for the Sovereignty Certificate Profile. The "Present In" column indicates whether a claim is intended for use in the Evidence Envelope (generated by the Attester), the Verifiable Attestation Result (VAR, generated by the Verifier), or both. Claims marked as "Mandatory" for a given token type SHALL be present in that token.

| Claim Name | Claim Key | Claim Value Type | Present In | Claim Description |
| :---- | :---- | :---- | :---- | :---- |
| eat\_profile | 265 | tstr | Evidence, VAR | The URI identifying this profile. Mandatory in Evidence and VAR. |
| ueid | 256 | bytes | Evidence, VAR | Universal Entity ID of the Attester's HRoT. Mandatory in Evidence and VAR. |
| bootseed | 268 | bytes | Evidence, VAR | A random value generated at boot to identify the current boot session and provide an additional source of freshness. Mandatory in Evidence and VAR. |
| platform\_quote | \-70000 | bytes | Evidence | The complete, signed platform integrity quote from the HRoT (e.g., Intel TDX Quote, AMD SEV-SNP Report). The format is specific to the HRoT. Mandatory in Evidence. |
| ephemeral\_pub\_key | \-70001 | COSE\_Key | Evidence | The ephemeral public key (COSE format) that is bound into the platform\_quote. Mandatory in Evidence. |
| location\_receipts | \-70002 | array of bytes | Evidence | An array where each element is a complete, signed receipt from an Anchor. Mandatory in Evidence. |
| location\_claim | \-70003 | map | VAR | A map containing the Verifier's computed location. Keys are lat (float), lon (float), radius\_m (unsigned int), and confidence (float, 0.0-1.0). Mandatory in VAR. |
| policy\_ids | \-70004 | array of tstr | VAR | An array of strings, where each string is a unique identifier for a policy that was successfully evaluated by the Verifier. Mandatory in VAR. |
| iss | 1 | tstr | VAR | The issuer of the token. For a VAR, this is the unique identifier of the Verifier. Mandatory in VAR. |
| iat | 6 | int | Evidence, VAR | The issuance time of the token, as a Unix timestamp (seconds since 1970-01-01T00:00:00Z). Mandatory in Evidence and VAR. |
| exp | 4 | int | VAR | The expiration time of the VAR, as a Unix timestamp. Mandatory in VAR. |

## 

## **Annex B (Informative) Example Implementation Workflow** {#annex-b-(informative)-example-implementation-workflow}

This annex provides a non-normative, step-by-step example of the end-to-end process for obtaining and using a Sovereignty Certificate. This example is intended to aid implementers in understanding the practical application of the normative requirements defined in this standard.

*Note: This example utilizes a Kubernetes architecture. Equivalent workflows apply to non-containerized Virtual Machine (VM) deployments, where:*

* *The Sidecar Container role is fulfilled by a system daemon or background service (e.g., a systemd unit) running with sufficient privileges to access the HRoT driver.*  
* *The Shared Volume is replaced by a protected local filesystem path (e.g., /run/sovereignty/ or /var/run/secrets/) accessible only to the daemon and the authorized application workload.*

### **B.1 Pod Startup and Gating**

A new containerized workload (Pod) is scheduled to run on a node in a data center. The Pod contains two containers: the primary application and a Sovereign Sidecar, which implements the Attester role. The application container is configured with a startup probe that checks for the existence and validity of a VAR file on a shared memory volume. The application will not be marked as "Ready" by the orchestrator until this probe succeeds, preventing it from serving traffic.5

### **B.2 Attester Initialization**

The Sovereign Sidecar container starts. Its first action is to initialize an attestation cycle.

1. It generates a new, ephemeral P-256 ECDSA key pair.  
2. It constructs a data structure containing the public key and a 256-bit nonce.  
3. It computes the SHA-256 hash of this structure.  
4. It makes a call to the underlying platform's TEE/HRoT driver (e.g., the Intel TDX driver) to request an attestation quote, passing the computed hash as the REPORTDATA parameter. The HRoT generates a quote containing platform measurements and signs it with its private Attestation Key (AK).5

### **B.3 Location Measurement**

The sidecar's Location Verification Agent (LVA) begins the location measurement process.

1. It sends a request to the Endorser service to fetch the latest signed Anchor Directory. It validates the directory's signature using the Endorser's public key.  
2. From the directory, it selects 8 geographically diverse Anchors.  
3. It sends a UDP probe packet to each selected Anchor. Each packet contains a unique nonce for that probe.  
4. It listens for responses. For each response received, it records the local time of receipt and validates the signature on the enclosed Anchor receipt. It successfully collects 7 valid, signed receipts.5

### **B.4 Evidence Submission**

The sidecar assembles the Evidence Envelope as a CWT, conforming to the profile in Annex A.

1. It populates the claims: eat\_profile, ueid (from the platform), bootseed, platform\_quote (the binary blob from step B.2), ephemeral\_pub\_key, and the location\_receipts array.  
2. It creates a COSE\_Sign1 structure, placing the CWT Claims Set in the payload.  
3. It signs the structure using the ephemeral private key generated in step B.2.  
4. It establishes an mTLS connection to the Verifier's API endpoint and submits the signed CWT.5

### **B.5 Verification and VAR Issuance**

The Verifier receives the CWT and performs the appraisal.

1. It validates the COSE\_Sign1 signature using the ephemeral\_pub\_key from the claims.  
2. It validates the platform\_quote, including the signature chain and the REPORTDATA binding.  
3. It validates the 7 Anchor receipts.  
4. It computes the location using the RTTs, resulting in coordinates (47.6062, \-122.3321), a radius of 45 meters, and a confidence score of 0.92.  
5. It evaluates this result against its policies. The location falls within the "us-west-2" jurisdiction, the radius of 45m is less than the maximum of 50m, and the confidence of 0.92 is greater than the minimum of 0.85. The TCB measurements also match the expected values. All policies pass.  
6. The Verifier generates a new CWT for the VAR, populating the location\_claim, policy\_ids, iss, iat, and exp claims. The exp is set to 10 minutes in the future.  
7. It signs the VAR with its own private Verifier key and returns it to the sidecar.5

### **B.6 Application Ungating**

The sidecar receives the signed VAR.

1. It validates the VAR's signature using the Verifier's public key.  
2. It writes the VAR token atomically to a file (e.g., result.jwt) on the shared memory volume.  
3. On the next execution, the application's startup probe finds the valid VAR, succeeds, and the orchestrator marks the Pod as "Ready." The application begins serving traffic, now possessing a valid Sovereignty Certificate that it can present to downstream services to prove its location and integrity.5

### **B.7 Continuous Rotation**

Approximately five minutes later, before the current VAR expires, the sidecar's internal timer triggers a new attestation cycle, and it repeats steps B.2 through B.6 to proactively refresh the VAR, ensuring continuous, uninterrupted proof of sovereignty for the workload.5

### **B.8 Alternative Architectures (Informative)**

#### **B.8.1 Virtual Machine (VM) Implementation**

In non-containerized VM environments, the architectural roles map as follows:

* **Attester:** Implemented as a background system daemon (e.g., systemd unit) with root privileges.  
* **Gating:** Downstream services on the VM check for the presence of the VAR file in a restricted directory (e.g., `/run/sovereignty/var.jwt`) before starting.

#### **B.8.2 Background Check Model**

In scenarios where the Relying Party relies on the "Background Check" model (refer to 4.2.3 ), the Attester does not present the VAR directly. Instead, the Attester presents a Reference Ticket (or the ephemeral public key), and the Relying Party submits this identifier to the Verifier to retrieve the VAR directly. This shifts the "Ungating" logic (B.6) from the Application to the Relying Party.

## **Annex C (Informative) Crosswalk to Existing Standards** {#annex-c-(informative)-crosswalk-to-existing-standards}

#### **C.1 Introduction**

This appendix provides a crosswalk from the Sovereignty Certificate specification to other relevant industry standards and regulatory frameworks. The purpose of this section is to show how this specification aligns with, builds upon, and can be used to support existing best practices in identity, security, location, and compliance.

By explicitly mapping the concepts in this document to established standards, we aim to:

* **Promote Interoperability:** Ensure that Sovereignty Certificates can be seamlessly integrated into the broader digital trust ecosystem.  
* **Accelerate Adoption:** Allow implementers to leverage existing tools, libraries, and expertise.  
* **Streamline Auditing and Compliance:** Provide a clear path for auditors and regulators to verify that the implementation of a Sovereignty Certificate meets specific control objectives from recognized security and privacy frameworks.

This crosswalk is intended for implementers, solution architects, security auditors, and compliance professionals.

#### **C.2 Standards Mapping**

The following table maps the core components and concepts of the Sovereignty Certificate to their corresponding external standards.

| Sovereignty Certificate Concept | Related Standard/ Framework | Mapping, Rationale, and Implementation Guidance |
| :---- | :---- | :---- |
| **C.2.1 Certificate Structure & Identity** | **W3C Verifiable Credentials (VC) Data Model 1.1** & **VC-JWT v1.1** | **Mapping:** The Sovereignty Certificate **IS** a specific profile of a W3C Verifiable Credential. The issuer, holder, credentialSubject, and proof fields defined in this specification directly map to the core VC data model.  **Rationale:** Adopting the W3C VC model ensures cryptographic verifiability, promotes interoperability with digital identity wallets and verifier systems, and provides a mature, standardized format for making attestable claims. **Implementation:** Certificates **MUST** be structured as a Verifiable Credential. For use in web and API contexts, they **SHOULD** be encoded as a JSON Web Token (JWT) according to the VC-JWT 1.1 specification. |
| **C.2.2 Attestation & Workload Integrity** | **IETF Remote Attestation Procedures (RATS) Architecture (RFC 9334\)** | **Mapping:** The entire process of generating and verifying the integrity of the computing environment follows the RATS architectural model. The "Attester" is the TEE, the "Relying Party" is the entity consuming the Sovereignty Certificate, and the "Verifier" is the service that appraises the attestation evidence. **Rationale:** Aligning with the RATS architecture provides a vendor-neutral, standardized framework for the remote attestation lifecycle. It ensures that the roles, terminology, and interaction flows are well-understood and interoperable across different hardware and software implementations. **Implementation:** The attestationReport claim within the certificate **MUST** be structured as "Evidence" in the RATS model. The verification process **SHOULD** follow the appraisal procedures for "Attestation Results" as defined in RATS. |
| **C.2.3 Attestation & Workload Integrity** | **Confidential Computing Consortium (CCC) Attestation Standards** | **Mapping:** This provides the specific technical underpinnings for the evidence generated within the RATS framework. The CCC defines standards for attestation in various Trusted Execution Environments (TEEs) like Intel SGX, AMD SEV, and ARM TrustZone.  **Rationale:** While RATS provides the architecture, CCC standards offer the concrete definitions and requirements for what constitutes valid attestation evidence from a TEE. This ensures a high level of assurance that the workload is running in a cryptographically isolated and protected environment. **Implementation:** The payload of the attestationReport claim **MUST** contain evidence verifiable against the specifications defined by the CCC for the specific TEE in use. |
| **C.2.4 Location & Geospatial Claims** | **Open Geospatial Consortium (OGC) Standards** | **Mapping:** The geolocation claim, which specifies the physical location of the computing workload, maps to OGC standards for encoding geographic information.  **Rationale:** Using OGC standards ensures that location data is represented in a precise, unambiguous, and universally understood format. This is critical for the primary function of the certificate. It also allows for the essential expression of uncertainty.  **Implementation:** Geographic coordinates **SHOULD** be encoded using OGC standards. When expressing location uncertainty, the claim **SHOULD** leverage concepts from the OGC Uncertainty Markup Language (UncertML) to represent confidence levels and margins of error. |
| **C.2.5 Timestamps & Proof of Timeliness** | **IETF Network Time Protocol v4 (NTPv4, RFC 5905\)** & **Precision Time Protocol (PTP, IEEE 1588\)** | **Mapping:** All timestamps included in the certificate, particularly the issuanceDate and any timestamps within the proof or evidence sections. **Rationale:** To prevent fraudulent backdating and ensure the temporal integrity of the evidence, all time-sensitive data must be sourced from a high-accuracy, synchronized clock. NTPv4 and PTP are the industry standards for achieving this.  **Implementation:** All systems involved in the creation and issuance of the certificate (the Attester, Verifier, and Issuer) **MUST** be synchronized to a trusted time source via NTPv4 or a protocol of equivalent or higher accuracy, such as PTP. |
| **C.2.6 Security Controls & Physical Security** | **Cloud Security Alliance (CSA) Cloud Controls Matrix (CCM) v4.0** | **Mapping:** The physicalSecurity claim within the certificate can attest to compliance with specific controls from the CCM's "Datacenter Security" (DCS) and "Physical & Environmental Security" (PE) domains. **Rationale:** This maps the technical attestations of the certificate to a widely recognized cloud security framework used for auditing and vendor risk management. It allows an organization to use the certificate as direct evidence of a Cloud Service Provider's compliance. **Implementation:** A Sovereignty Certificate **MAY** include claims asserting compliance with specific CCM controls (e.g., ccm\_control\_id: "DCS-01"). Verifiers can use this to automate checks against their own compliance requirements. |
| **C.2.7 Security Controls & Data Residency** | **NIST Special Publication 800-53 Rev. 5** | **Mapping:** The certificate serves as a technical mechanism to help satisfy control objectives within the "Physical and Environmental Protection" (PE) and "System and Communications Protection" (SC) control families.  **Rationale:** For organizations aligned with the NIST Risk Management Framework (RMF), the certificate provides verifiable evidence for controls related to physical location, information flow enforcement, and boundary protection.  **Implementation:** An organization's system security plan (SSP) **MAY** reference the use of Sovereignty Certificates as a technical implementation for satisfying specific NIST SP 800-53 controls (e.g., PE-3: Physical Access Control, SC-7: Boundary Protection). |
| **C.2.8 Regulatory & Data Sovereignty Alignment** | **EU General Data Protection Regulation (GDPR)** & **EU Cybersecurity Certification Scheme for Cloud Services (EUCS)** | **Mapping:** The certificate provides a technical means to demonstrate compliance with data residency and data localization requirements, which are central tenants of GDPR (e.g., Chapter V on data transfers) and the "High" assurance level of EUCS.  **Rationale:** These regulations require data controllers to be accountable for where data is processed. The certificate moves beyond contractual assurances to provide tangible, cryptographic proof, significantly strengthening an organization's compliance posture.  **Implementation:** Organizations **SHOULD** use Sovereignty Certificates as part of their technical and organizational measures (as required by GDPR Article 32\) to demonstrate that data processing occurs within a permitted jurisdiction. |

#### 

## **References**

1. ISO/IEC Directives — Principles and rules for the structure and ..., accessed August 20, 2025, [https://metanorma.github.io/mn-samples-iso/documents/directives/part2/document.html](https://metanorma.github.io/mn-samples-iso/documents/directives/part2/document.html)  
2. ISO/IEC Directives Part 2: Quality Standards for Documentation \- 6clicks, accessed August 20, 2025, [https://www.6clicks.com/resources/glossary/iso-iec-directives-part-2](https://www.6clicks.com/resources/glossary/iso-iec-directives-part-2)  
3. Sovereignty Certificates Working Group  
4. ISO/IEC 27001:2022 – Information Security Management \- IT Governance, accessed August 20, 2025, [https://www.itgovernance.co.uk/iso27001](https://www.itgovernance.co.uk/iso27001)  
5. Brainstorm: End-To-End Sovereign Container  
6. ISO/IEC 27001 – Information Security Management Certification \- NSF, accessed August 20, 2025, [https://www.nsf.org/management-systems/iso-iec-27001](https://www.nsf.org/management-systems/iso-iec-27001)  
7. RFC 9334 \- Remote ATtestation procedureS (RATS) Architecture, accessed August 20, 2025, [https://datatracker.ietf.org/doc/rfc9334/](https://datatracker.ietf.org/doc/rfc9334/)  
8. Information on RFC 9334 \- » RFC Editor, accessed August 20, 2025, [https://www.rfc-editor.org/info/rfc9334](https://www.rfc-editor.org/info/rfc9334)  
9. RFC 9711 \- The Entity Attestation Token (EAT) \- IETF Datatracker, accessed August 20, 2025, [https://datatracker.ietf.org/doc/rfc9711/](https://datatracker.ietf.org/doc/rfc9711/)  
10. Information on RFC 9711 \- » RFC Editor, accessed August 20, 2025, [https://www.rfc-editor.org/info/rfc9711](https://www.rfc-editor.org/info/rfc9711)  
11. ISO/IEC 19790:2025—Security for Cryptographic Modules \- The ANSI Blog, accessed August 20, 2025, [https://blog.ansi.org/ansi/iso-iec-19790-2025-cryptographic-modules/](https://blog.ansi.org/ansi/iso-iec-19790-2025-cryptographic-modules/)  
12. Security Requirements for Cryptographic Modules \- NIST Technical Series Publications, accessed August 20, 2025, [https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-3.pdf](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-3.pdf)  
13. Remote Platform Integrity Attestation \- Trusted Computing Group, accessed August 20, 2025, [https://trustedcomputinggroup.org/remote-platform-integrity-attestation/](https://trustedcomputinggroup.org/remote-platform-integrity-attestation/)  
14. Overview of TCG Technologies for Device Identification and Attestation \- Trusted Computing Group, accessed August 20, 2025, [https://trustedcomputinggroup.org/wp-content/uploads/4\_G.Fedorkow\_TCG-JRF02292024.pdf](https://trustedcomputinggroup.org/wp-content/uploads/4_G.Fedorkow_TCG-JRF02292024.pdf)  
15. TPM 2.0 Keys for Device Identity and Attestation \- Trusted Computing Group, accessed August 20, 2025, [https://trustedcomputinggroup.org/resource/tpm-2-0-keys-for-device-identity-and-attestation/](https://trustedcomputinggroup.org/resource/tpm-2-0-keys-for-device-identity-and-attestation/)  
16. Overview of TCG Technologies for Device Identification and Attestation \- Trusted Computing Group, accessed August 20, 2025, [https://trustedcomputinggroup.org/wp-content/uploads/Overview-of-TCG-Technologies-for-Device-Identification-and-Attestation-Version-1.0-Revision-1.37\_5Feb24-2.pdf](https://trustedcomputinggroup.org/wp-content/uploads/Overview-of-TCG-Technologies-for-Device-Identification-and-Attestation-Version-1.0-Revision-1.37_5Feb24-2.pdf)  
17. ISO 8601 \- Wikipedia, accessed August 20, 2025, [https://en.wikipedia.org/wiki/ISO\_8601](https://en.wikipedia.org/wiki/ISO_8601)  
18. A Trusted Execution Environment RISC-V System-on-Chip Compatible with Transport Layer Security 1.3 \- MDPI, accessed August 20, 2025, [https://www.mdpi.com/2079-9292/13/13/2508](https://www.mdpi.com/2079-9292/13/13/2508)  
19. Trusted execution environment \- Wikipedia, accessed August 20, 2025, [https://en.wikipedia.org/wiki/Trusted\_execution\_environment](https://en.wikipedia.org/wiki/Trusted_execution_environment)  
20. draft-ietf-rats-reference-interaction-models-14 \- Reference Interaction Models for Remote Attestation Procedures \- IETF Datatracker, accessed August 20, 2025, [https://datatracker.ietf.org/doc/draft-ietf-rats-reference-interaction-models/](https://datatracker.ietf.org/doc/draft-ietf-rats-reference-interaction-models/)  
21. TEE System Architecture v1.2 \- GlobalPlatform, accessed August 20, 2025, [https://globalplatform.org/wp-content/uploads/2017/01/GPD\_TEE\_SystemArch\_v1.2\_PublicRelease.pdf](https://globalplatform.org/wp-content/uploads/2017/01/GPD_TEE_SystemArch_v1.2_PublicRelease.pdf)  
22. Trusted Execution Environments in Digital Advertising: A Pathway to Enhanced Data Privacy, Security, and Regulatory Compliance \- IAB, accessed August 20, 2025, [https://www.iab.com/wp-content/uploads/2025/04/IAB\_Trusted\_Execution\_Environments\_Whitepaper\_April\_2025.pdf](https://www.iab.com/wp-content/uploads/2025/04/IAB_Trusted_Execution_Environments_Whitepaper_April_2025.pdf)  
23. ISO 27002: Security Controls \- IT Governance USA, accessed August 20, 2025, [https://www.itgovernanceusa.com/iso27002](https://www.itgovernanceusa.com/iso27002)  
24. NIST Cryptographic Standards & Their Adoptions in International / Industry Standards, accessed August 20, 2025, [https://csrc.nist.gov/CSRC/media/Presentations/NIST-Cryptographic-Standards-Their-Adoptions-in/images-media/ispab\_june-11\_crypto-standards\_lchen.pdf](https://csrc.nist.gov/CSRC/media/Presentations/NIST-Cryptographic-Standards-Their-Adoptions-in/images-media/ispab_june-11_crypto-standards_lchen.pdf)  
25. ISO/IEC JTC 1/SC 27/WG 2 \- Cryptography and security mechanisms \- iTeh Standards, accessed August 20, 2025, [https://standards.iteh.ai/catalog/tc/iso/5883b159-c1b5-4ef4-8e4b-077ace20a48e/iso-iec-jtc-1-sc-27-wg-2](https://standards.iteh.ai/catalog/tc/iso/5883b159-c1b5-4ef4-8e4b-077ace20a48e/iso-iec-jtc-1-sc-27-wg-2)  
26. ISO 27001:2022 Annex A 8.24 – Use of Cryptography \- ISMS.online, accessed August 20, 2025, [https://www.isms.online/iso-27001/annex-a/8-24-use-of-cryptography-2022/](https://www.isms.online/iso-27001/annex-a/8-24-use-of-cryptography-2022/)  
27. ISO 27002:2022 – Control 8.24 – Use of Cryptography \- ISMS.online, accessed August 20, 2025, [https://www.isms.online/iso-27002/control-8-24-use-of-cryptography/](https://www.isms.online/iso-27002/control-8-24-use-of-cryptography/)  
28. RFC 9782: Entity Attestation Token (EAT) Media Types, accessed August 20, 2025, [https://www.rfc-editor.org/rfc/rfc9782.html](https://www.rfc-editor.org/rfc/rfc9782.html)

[^1]:   Other initiatives exist that focus on on proving that "workloads run on legitimate cloud hardware in secure facilities” such as [Proof of Cloud](https://proofofcloud.org/%20), however, they use physical verification of facilities. This working group will focus on TEE attestation, which is an independent security layer from physical verification. Outside public cloud, data residency is commonly sought through on-premise computing or private clouds.

[^2]:  Sovereignty is a contested term. This working group is primarily focused on technical controls for sovereignty, in line with the European Commission’s [Cloud Sovereignty Framework](https://www.google.com/url?q=https://commission.europa.eu/document/download/09579818-64a6-4dd5-9577-446ab6219113_en?filename%3DCloud-Sovereignty-Framework.pdf&sa=D&source=docs&ust=1764089166222959&usg=AOvVaw0VqMa2TKYWaOAhlUMSHRW5), which defines Data & AI Sovereignty as “the protection, control, and independence of data assets and AI services within the EU. It addresses how data is secured, where it is processed, and the degree of autonomy customers retain over AI capabilities." 