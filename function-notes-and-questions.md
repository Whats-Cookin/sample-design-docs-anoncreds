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

## Notes (prev questions now answered)

how exactly is [create_credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/indy/issuer.py#L114) in aca-py tied to the wrapper function [create_credential](https://github.com/hyperledger/anoncreds-rs/blob/main/wrappers/python/anoncreds/bindings.py#L631C5-L631C22) - is this by some magic in the Dockerfile ?

[Comment from Daniel] -- ACA-PY's plugin model automatically loads certain functions. I have not studied this carefully, but it is possible that the plugin logic is the magic you are wondering about. TBD.

pipy import of anoncreds - in the .toml file

## Questions

---

Are we still implementing the same abstract method
[abstract method for create credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/indy/issuer.py#L113)

[Comment from Daniel] -- I think the creation of W3C credentials could fit into this abstract method. The Tails argument would always be None, and you would need some way of indicating which credential type you want, but it could be done. The bigger question, architecturally, is whether you want the format of the credential to be hidden behind the abstraction (in which case, using this abstract method makes sense), or whether you want callers to know which credential type they're dealing with (in which case, you need a second method for creating a different kind of artifact). The second strategy is clearer, and it makes some sense, because you will have different codepaths eventually, anyway. However, it makes codepaths split earlier in the lifecycle of the credential. This would be a good question to discuss with Stephen to see what his intuition is about the preferred tradeoff.

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

[Comment from Daniel] See my previous comment about tradeoffs. Given this change in signatures, it would seem that the Rust implementation is voting for the second option -- separate functions. I think that could make sense, and it would mean the answer to your question, "we need totally separate indy sdk functions" is "Yes".

at what level can we make them unified if at all? or we just provide different top level functions?  Can we have a FlexCreds creator that allows request for multiple signatures?

[Comment from Daniel] Needs confirmation, but it seems like Rust is voting for different top level functions. Yes a FlexCreds creator would be nice.

What is the highest level where these get called? where does the user of the SDK interact with them?

when is a credential actually issued? i couldn't find a function that explicitly does that in anondreds-rs, only the creation of credential. according to this [part](https://github.com/DSRCorporation/anoncreds-rs/blob/17bd63d8eb032232f418111dd9b0ae7751062aae/src/services/issuer.rs#L755), does it mean that if the `status` is `issuance_by_default`, will it issue a credential immediately after creating it?

[Comment from Daniel] In typical AnonCreds and Aries workflows, a credential typically isn't created unless it is issued, because AnonCreds credentials require a cryptographic commitment to a link (master) secret from the holder. That means an issuer can't create a credential until the holder is ready to participate, and I would expect virtually all codepaths that create a credental to be part of a larger codepath where issuance is underway. W3C VCs don't have this requirement, so it becomes possible for an issuer to create a credential without issuing it, or for a holder to create a credential via conversion, long after issuance has occurred.  

this is the [python handler function](https://github.com/hyperledger/aries-cloudagent-python/blob/c677185d3a3a36b498109236131684c322d75f5a/aries_cloudagent/protocols/issue_credential/v1_0/routes.py#L546) for issuing a credential manually _when the issuer and holder are not both configured for automatic responses_

[Comment from Daniel] This function is used by the DIDComm Credential Exchange protocol. You can think of it as being somewhat like the handler for credential issuance in VC API. The DIDComm protocol is the same whether each party is using a fully automated agent (e.g., on a server somewhere), or an agent that wants to interact with its user (e.g., on a mobile device). In the second case, the agent that will receive the credential may be stuck, waiting for user approval, at certain points. You can see in the function implementation that it awaits responses from the remote party before it continues. It looks to me like this function embodies the overall issuance workflow for the protocol: Preview, Propose, Send. 