# Anoncreds W3C Compatibility

This design proposes to extend the Aries Cloud Agent Python (ACA-Py) to support Hyperledger AnonCreds credentials and presentations in the W3C Verifiable Credentials (VC) and Verifiable Presentations (VP) Format.

The aim is to transition from the legacy AnonCreds format specified in [Aries-Legacy-Method] (https://hyperledger.github.io/anoncreds-methods-registry/#hyperledger-indy-legacy-anoncreds-method) to the W3C VC format.


## Overview

We aim to wrap the enhancements made on the RUST Framework [Anoncreds Rust] (https://github.com/hyperledger/anoncreds-rs) first, the integration of AnonCreds with W3C VC Format in ACA-Py, which includes support for issuing, verifying, and managing W3C VC Format AnonCreds credentials.

Also, we're emphasizing cryptographic agility and advanced storage capabilities, enabling multiple signature types, ensuring smooth integration with DIDComm Protocol alignments within ACA-Py, and making the framework interoperable with the Javascript Framework [Document](https://github.com/hyperledger/aries-framework-javascript).


## Caveats (or What's out of scope)

We will only target compatibility with VCDM (Verifiable Credential Data Model) 1.1 because, primarily, the Python framework is going to be a wrapper on the RUST implementation and would support the features being implemented in the RUST frameworks, which include:

* Credentials: Verify validity of non-Creds Data Integrity proof signatures
* Presentations: Create presentations using non-AnonCreds Data Integrity proof signature
* Presentations: Verify validity of presentations, including non-AnonCreds Data Integrity proof signatures
* Presentations: Support different formats (for example, DIF) of Presentation Request

This is also because VCDM (Verifiable Credential Data Model) 2.0 implementations are not mature enough for interop yet.

## Key Questions

### How will these functions be exposed in Acapy?


### Will you write helper methods that are even higher-level than these six functions?


### Do any new admin functions need to be built on the control channel?


### Compatibility with AFJ: How will you make sure that you are compatible?




## Diagram representation 

![w3c diagram](./w3c-diagram.png)
