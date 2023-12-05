# Functions and Pointers

### python wrapper

https://github.com/hyperledger/anoncreds-rs/blob/main/wrappers/python/demo/test.py

see new functions being put into the python wrapper:
[create_w3c_credential_offer](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-f700874ced6aa62466f7a24eb730cd568dcdfa6f6f98322ea1e802dbc8fe91a8R712)

### Indy SDK level

[create_credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/indy/sdk/issuer.py#L153)
...

### existing functions (anoncreds level):

[create_credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/anoncreds/issuer.py#L546)

`process_credential`,
`create_presentation`,
`verify_presentation`
`create_offer`,
`create_request`

### new functions:

anoncreds_create_w3c_credential - actually the new binding is [create_w3c_credential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-f700874ced6aa62466f7a24eb730cd568dcdfa6f6f98322ea1e802dbc8fe91a8R1043)
anoncreds_process_w3c_credential

anoncreds_create_w3c_credential_offer
anoncreds_create_w3c_credential_request

anoncreds_w3c_credential_get_attribute
anoncreds_create_w3c_presentation
anoncreds_verify_w3c_presentation

see [Demo Scripts](https://github.com/hyperledger/anoncreds-rs/pull/266/files#diff-f0f0c92035decc44061ca415febaa763d3b4b86afc79572ba83f4d76d2a0f617R523)

## Questions

---

Are we still implementing the same abstract method
[abstract method for create credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/indy/issuer.py#L113)

---

how exactly is [create_credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/indy/issuer.py#L114) in aca-py tied to the wrapper function [create_credential](https://github.com/hyperledger/anoncreds-rs/blob/main/wrappers/python/anoncreds/bindings.py#L631C5-L631C22) - is this by some magic in the Dockerfile ?

---

The function signatures have changed -

original [create_credential](https://github.com/hyperledger/anoncreds-rs/blob/3004b69871f80a34514a68c24a822b3c46bc55fd/wrappers/python/anoncreds/bindings.py#L631)

```
def create_credential(
    cred_def: ObjectHandle,
    cred_def_private: ObjectHandle,
    cred_offer: ObjectHandle,
    cred_request: ObjectHandle,
    attr_raw_values: Mapping[str, str],
    attr_enc_values: Optional[Mapping[str, str]],
    revocation_config: Optional[RevocationConfig],
) -> ObjectHandle:
```

to
[create_w3c_credential](https://github.com/hyperledger/anoncreds-rs/pull/273/files#diff-f700874ced6aa62466f7a24eb730cd568dcdfa6f6f98322ea1e802dbc8fe91a8R1043)

```
def create_w3c_credential(
    cred_def: ObjectHandle,
    cred_def_private: ObjectHandle,
    cred_offer: ObjectHandle,
    cred_request: ObjectHandle,
    attr_raw_values: Mapping[str, str],
    revocation_config: Optional[RevocationConfig],
    encoding: Optional[str],
```

how does that impact our implementation, does this mean we need totally separate indy sdk functions to wrap each of these, it should not be a single one?

at what level can we make them unified if at all? or we just provide different top level functions?

What is the highest level where these get called? where does the user of the SDK interact with them?
