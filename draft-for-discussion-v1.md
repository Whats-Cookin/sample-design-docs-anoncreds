# Anoncreds W3C Compatibility

This design proposes to extend the Aries Cloud Agent Python (ACA-Py) to support Hyperledger AnonCreds credentials and presentations in the W3C Verifiable Credentials (VC) and Verifiable Presentations (VP) Format. The aim is to transition from the legacy AnonCreds format specified in Aries-Legacy-Method to the W3C VC format.
<br><br>

## Overview

We aim to wrap the enhancements made on the RUST Framework Anoncreds Rust first, the integration of AnonCreds with W3C VC Format in ACA-Py, which includes support for issuing, verifying, and managing W3C VC Format AnonCreds credentials.

Ideally the signatures will be in parallel with the Javascript Framework Document.
<br><br>

## Caveats

We will only target compatibility with VCDM (Verifiable Credential Data Model) 1.1 because, primarily, the Python framework is going to be a wrapper on the RUST implementation and would support the features being implemented in the RUST frameworks, which include:

- Credentials: Verify validity of non-Creds Data Integrity proof signatures
- Presentations: Create presentations using non-AnonCreds Data Integrity proof signature
- Presentations: Verify validity of presentations, including non-AnonCreds Data Integrity proof signatures
- Presentations: Support different formats (for example, DIF) of Presentation Request

This is also because VCDM (Verifiable Credential Data Model) 2.0 implementations are not mature enough for interop yet.
<br><br>

## Key Questions

### What are the functions we are going to wrap?

After thoroughly reviewing this [PR](https://github.com/hyperledger/anoncreds-rs/pull/273) from DSR Coporation, the objects that we deemed necessary to be exported are:<br>

[W3CCredentialOffer](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R106)<br>
class methods (`create`, `load`)<br>
rust functions (`create_w3c_credential_offer`)<br>

[W3CCredentialRequest](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R162)<br>
class methods (`create`, `load`)<br>
rust functions (`create_w3c_credential_request`)<br>

[W3CCredential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R424)<br>
class methods (`create`, `load`)<br>
instance methods (`proceess`, `to_legacy`, `add_non_anoncreds_integrity_proof`, `set_id`, `set_subject_id`, `add_context`, `add_type`)<br>
class properties (`schema_id`, `cred_def_id`, `rev_reg_id`, `rev_reg_index`)<br>
rust functions (`create_w3c_credential`, `process_w3c_credential`, `_object_from_json`, `_object_get_attribute`, `w3c_credential_add_non_anoncreds_integrity_proof`, `w3c_credential_set_id`, `w3c_credential_set_subject_id`, `w3c_credential_add_context`, `w3c_credential_add_type`)<br>

[W3CPresentation](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R791)<br>
class methods (`create`, `load`)<br>
instance methods (`verify`)<br>
rust functions (`create_w3c_presentation`, `_object_from_json`, `verify_w3c_presentation`)<br>

They will be added to [\_\_init\_\_.py](https://github.com/hyperledger/anoncreds-rs/blob/main/wrappers/python/anoncreds/__init__.py) as additional exports of AnoncredsObject. <br><br>

We also have to consider which classes or anoncreds objects have been modified

The classes modified according to the same [PR](https://github.com/hyperledger/anoncreds-rs/pull/273) mentioned above are:<br>

[Credential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R402)<br>
added class methods (`from_w3c`)<br>
added instance methods (`to_w3c`)<br>
added rust functions (`credential_from_w3c`, `credential_to_w3c`)<br>

[PresentCredential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R603)<br>
modified instance methods (`_get_entry`, `add_attributes`, `add_predicates`)<br>
<br>

### How will they fit into aca-py?

The issuance, presentation and verification of legacy anoncreds are implemented in this [./aries_cloudagent/anoncreds](https://github.com/hyperledger/aries-cloudagent-python/tree/main/aries_cloudagent/anoncreds) directory. Therefore, we will also start from there.<br>

Let us navigate this implementation path through the processes of each of the agents - Issuer, Holder and Verifier as described in https://github.com/hyperledger/anoncreds-rs/blob/main/README.md.

Looking at the [issuer.py](https://github.com/hyperledger/aries-cloudagent-python/blob/main/aries_cloudagent/anoncreds/issuer.py) file and this code block:

```
async def create_credential_offer(self, credential_definition_id: str) -> str:
...
...
  credential_offer = CredentialOffer.create(
                  schema_id or cred_def.schema_id,
                  credential_definition_id,
                  key_proof.raw_value,
              )
...
```

we can implement the same thing in w3c VC format to send a w3c credential offer like so:

#### W3C Credential Offer

```
async def create_w3c_credential_offer(self, credential_definition_id: str) -> str:
...
...
  w3c_credential_offer = W3CCredentialOffer.create(...)
...
```

provided `W3CCredentialOffer` is already imported from `anoncreds` module.<br>

We will proceed through the following processes in comparison with the legacy anoncreds implementations while watching out for differences between the two.<br>

#### W3C Credential Create

```
async def create_credential(
        self,
        credential_offer: dict,
        credential_request: dict,
        credential_values: dict,
    ) -> str:
...
...
  try:
    credential = await asyncio.get_event_loop().run_in_executor(
        None,
        lambda: W3CCredential.create(
            cred_def.raw_value,
            cred_def_private.raw_value,
            credential_offer,
            credential_request,
            raw_values,
            None,
            None,
            None,
            None,
        ),
    )
```

#### W3C Credential Request

```
async def create_credential_request(
        self, credential_offer: dict, credential_definition: CredDef, holder_did: str
    ) -> Tuple[str, str]:
...
...
try:
  secret = await self.get_master_secret()
  (
      cred_req,
      cred_req_metadata,
  ) = await asyncio.get_event_loop().run_in_executor(
      None,
      CredentialRequest.create,
      None,
      holder_did,
      credential_definition.to_native(),
      secret,
      AnonCredsHolder.MASTER_SECRET_ID,
      credential_offer,
  )
...
```

#### W3C Credential

### Do any new admin functions need to be built on the control channel?

### Compatibility with AFJ: how can we make sure that we are compatible?

### What is the roadmap for delivery? What will we build first, then second?

### Will we introduce new dependencies, and what is risky or easy?
