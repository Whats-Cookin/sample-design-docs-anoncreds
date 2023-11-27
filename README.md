 # DESIGN DOCS | Aries Cloud Agent Python (ACA-Py) Extension Project

**Project Overview**

This project involves extending the Aries Cloud Agent Python (ACA-Py) to support Hyperledger AnonCreds credentials and presentations in the W3C Verifiable Credentials (VC) and Verifiable Presentations (VP) Format. The aim is to transition from the legacy AnonCreds format to the W3C VC format.

**Key Features**

- Support for issuing and verifying W3C VC Format AnonCreds credentials.
  
- Enabling cryptographic agility in credentials through multiple signature types.

- Seamless storage and retrieval of W3C VC Format AnonCreds credentials by ACA-Py holders.
  
- Integration of AnonCreds with W3C VC Format in ACA-Py, considering DIDComm Protocol alignments and multiple signature handling.


**Interoperability**

- Coordination with Aries Framework JavaScript to ensure interoperability between the implementations.
- The ACA-Py will be used primarily for issuing, verifying, and potentially in an organizational wallet context.

### Important links to the key features, we will be implementing

- (issuing and verifying anoncreds)[./issuing-verifying.md]

- !(W3C VC Format)[./W3C-VC-Format.md]