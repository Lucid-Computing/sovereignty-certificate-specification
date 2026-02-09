# Sovereignty Certificate Specification

> **Status: Draft** — This specification is under active development and has not been finalized. Feedback and contributions are welcome.

## Overview

This specification defines a framework for generating and verifying **Sovereignty Certificates** — cryptographic credentials that provide verifiable, unforgeable proof of the physical location of a computing workload at a specific point in time. It introduces **Ping-Based Location Attestation**, a novel approach that leverages the physical constraint of the speed of light to provide hardware-backed proof of location, integrated with remote attestation of platform integrity.

The architecture conforms to the IETF Remote ATtestation procedureS (RATS) Architecture (RFC 9334) and uses the Entity Attestation Token (EAT) format (RFC 9711) for evidence and attestation results.

## Specification

The current draft specification is available at:

**[Sovereignty Certificates (Draft)](spec/sovereignty-certificates.md)**

## About the Working Group

The Sovereignty Certificates Working Group is a collaborative initiative bringing together experts from cloud service providers, artificial intelligence laboratories, hardware manufacturers, and enterprises in regulated industries. The working group was formed to address a critical gap in modern computing infrastructure: the absence of a standardized, cryptographically verifiable mechanism for proving the physical location of data processing workloads.

## Providing Feedback

We welcome feedback from implementers, security researchers, standards bodies, and the broader community.

- **Report an issue** — Use our [issue templates](https://github.com/Lucid-Computing/sovereignty-certificate-specification/issues/new/choose) to submit editorial corrections, technical concerns, or general questions
- **Join the discussion** — Use [GitHub Discussions](https://github.com/Lucid-Computing/sovereignty-certificate-specification/discussions) for open-ended conversations, implementation experience, and broader architectural questions
- **Propose a change** — See [CONTRIBUTING.md](CONTRIBUTING.md) for how to submit pull requests

## Key References

- [IETF RFC 9334 — Remote ATtestation procedureS (RATS) Architecture](https://datatracker.ietf.org/doc/rfc9334/)
- [IETF RFC 9711 — The Entity Attestation Token (EAT)](https://datatracker.ietf.org/doc/rfc9711/)
- [TCG TPM 2.0 Library Specification](https://trustedcomputinggroup.org/resource/tpm-library-specification/)
- [TCG DICE Attestation Architecture](https://trustedcomputinggroup.org/wp-content/uploads/4_G.Fedorkow_TCG-JRF02292024.pdf)

## License

This specification is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).
