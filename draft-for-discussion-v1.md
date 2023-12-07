# Anoncreds W3C Compatibility

This design proposes to extend the Aries Cloud Agent Python (ACA-Py) to support Hyperledger AnonCreds credentials and presentations in the W3C Verifiable Credentials (VC) and Verifiable Presentations (VP) Format. The aim is to transition from the legacy AnonCreds format specified in Aries-Legacy-Method to the W3C VC format.
<br><br>

## Overview

We aim to wrap the enhancements made on the RUST Framework Anoncreds Rust first, the integration of AnonCreds with W3C VC Format in ACA-Py, which includes support for issuing, verifying, and managing W3C VC Format AnonCreds credentials.

Ideally the signatures will be in parallel with the Javascript Framework Document.
<br><br>

## Caveats

For now, we will only target compatibility with VCDM (Verifiable Credential Data Model) 1.1 because the Rust framework we are deriving from is also working with it and would support the features being implemented in the RUST frameworks, which include:

- Credentials: Verify validity of non-Creds Data Integrity proof signatures
- Presentations: Create presentations using non-AnonCreds Data Integrity proof signature
- Presentations: Verify validity of presentations, including non-AnonCreds Data Integrity proof signatures
- Presentations: Support different formats (for example, DIF) of Presentation Request

This is also because VCDM (Verifiable Credential Data Model) 2.0 implementations are not mature enough for interop yet.
<br><br>

## Issues to consider

- If and how the W3C VC Format attachments for the Issue Credential V2.0 and Present Proof V2 Aries DIDComm Protocols should be used when using AnonCreds W3C VC Format credentials.
- How AnonCreds W3C VC Format verifiable credentials are stored by the holder such that they will be discoverable when needed for creating verifiable presentations.
- How and when multiple signatures can/should be added to a W3C VC Format credential, enabling both AnonCreds and non-AnonCreds signatures on a single credential and their use in presentations.
  <br><br>

## Key Questions

### What are the functions we are going to wrap?

After thoroughly reviewing this [PR](https://github.com/hyperledger/anoncreds-rs/pull/273) from DSR Coporation, the classes or `AnoncredsObject` that we deemed necessary to be exported are:<br>

[W3CCredentialOffer](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R106)<br>
class methods (`create`, `load`)<br>
bindings functions (`create_w3c_credential_offer`)<br>

[W3CCredentialRequest](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R162)<br>
class methods (`create`, `load`)<br>
bindings functions (`create_w3c_credential_request`)<br>

[W3CCredential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R424)<br>
class methods (`create`, `load`)<br>
instance methods (`proceess`, `to_legacy`, `add_non_anoncreds_integrity_proof`, `set_id`, `set_subject_id`, `add_context`, `add_type`)<br>
class properties (`schema_id`, `cred_def_id`, `rev_reg_id`, `rev_reg_index`)<br>
bindings functions (`create_w3c_credential`, `process_w3c_credential`, `_object_from_json`, `_object_get_attribute`, `w3c_credential_add_non_anoncreds_integrity_proof`, `w3c_credential_set_id`, `w3c_credential_set_subject_id`, `w3c_credential_add_context`, `w3c_credential_add_type`)<br>

[W3CPresentation](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R791)<br>
class methods (`create`, `load`)<br>
instance methods (`verify`)<br>
bindings functions (`create_w3c_presentation`, `_object_from_json`, `verify_w3c_presentation`)<br>

They will be added to [\_\_init\_\_.py](https://github.com/hyperledger/anoncreds-rs/blob/main/wrappers/python/anoncreds/__init__.py) as additional exports of AnoncredsObject. <br><br>

We also have to consider which classes or anoncreds objects have been modified

The classes modified according to the same [PR](https://github.com/hyperledger/anoncreds-rs/pull/273) mentioned above are:<br>

[Credential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R402)<br>
added class methods (`from_w3c`)<br>
added instance methods (`to_w3c`)<br>
added bindings functions (`credential_from_w3c`, `credential_to_w3c`)<br>

[PresentCredential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-6f8cbd34bbd373240b6af81f159177023c05b074b63c7757fc6b3796a66ee240R603)<br>
modified instance methods (`_get_entry`, `add_attributes`, `add_predicates`)<br>
<br>

### How will they fit into aca-py?

There are two scenarios to consider when we want to add w3c format support.

- Creating a W3C VC credential from credential definition, and issuing and presenting it as is
- Converting an already issued legacy anoncreds to W3C format(vice versa) so the converted credential can be issued of presented.

#### Creating a W3C VC credential from credential definition, and issuing and presenting it as is

The issuance, presentation and verification of legacy anoncreds are implemented in this [./aries_cloudagent/anoncreds](https://github.com/hyperledger/aries-cloudagent-python/tree/main/aries_cloudagent/anoncreds) directory. Therefore, we will also start from there.<br>

Let us navigate these implementation examples through the respective processes of the concerning agents - **Issuer** and **Holder** as described in https://github.com/hyperledger/anoncreds-rs/blob/main/README.md.

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

- W3C Credential Offer

```
async def create_w3c_credential_offer(self, credential_definition_id: str) -> str:
...
...
  w3c_credential_offer = W3CCredentialOffer.create(...)
...
```

provided `W3CCredentialOffer` is already imported from `anoncreds` module.<br>

In a similar manner, we will proceed through the following processes in comparison with the legacy anoncreds implementations while watching out for signature differences between the two.<br>

- W3C Credential Create

**NOTE: There has been some changes to _encoding of attribute values_ for creating a credential, so we have to be adjust to the new changes.**

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
...
```

- W3C Credential Request

```
async def create_w3c_credential_request(
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
      W3CCredentialRequest.create,
      None,
      holder_did,
      credential_definition.to_native(),
      secret,
      AnonCredsHolder.MASTER_SECRET_ID,
      credential_offer,
  )
...
```

- W3C Credential Present

```
async def create_w3c_presentation(
        self,
        presentation_request: dict,
        requested_credentials: dict,
        schemas: Dict[str, AnonCredsSchema],
        credential_definitions: Dict[str, CredDef],
        rev_states: dict = None,
    ) -> str:
...
...
  try:
    secret = await self.get_master_secret()
    presentation = await asyncio.get_event_loop().run_in_executor(
        None,
        Presentation.create,
        presentation_request,
        present_creds,
        self_attest,
        secret,
        {
            schema_id: schema.to_native()
            for schema_id, schema in schemas.items()
        },
        {
            cred_def_id: cred_def.to_native()
            for cred_def_id, cred_def in credential_definitions.items()
        },
    )
...
```

#### Converting an already issued legacy anoncreds to W3C format(vice versa)

In this case, we can use `to_w3c` method of `Credential` class to convert from legacy to w3c and `to_legacy` method of `W3CCredential` class to convert from w3c to legacy.<br>

We could call `to_w3c` method like this:

```
w3c_cred = Credential.to_w3c(cred_def)
```

and for `to_legacy`:

```
legacy_cred = W3CCredential.to_legacy()
```

We don't need to input any parameters to it as it in turn calls `Credential.from_w3c()` method under the hood

### How to handle multiple signatures on a W3C VC Format credential?

### Do any new admin functions need to be built on the control channel?

### Compatibility with AFJ: how can we make sure that we are compatible?

### What is the roadmap for delivery? What will we build first, then second?

### Will we introduce new dependencies, and what is risky or easy?

**TBD**
