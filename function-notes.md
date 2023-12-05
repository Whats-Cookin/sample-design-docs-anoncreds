# Functions and Pointers

existing functions:

[create_credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/anoncreds/issuer.py#L546)

`process_credential`,
`create_presentation`,
`verify_presentation`
`create_offer`,
`create_request`

new functions:

anoncreds_create_w3c_credential
anoncreds_process_w3c_credential

anoncreds_create_w3c_credential_offer
anoncreds_create_w3c_credential_request

anoncreds_w3c_credential_get_attribute
anoncreds_create_w3c_presentation
anoncreds_verify_w3c_presentation

see [Demo Scripts](https://github.com/hyperledger/anoncreds-rs/pull/266/files#diff-f0f0c92035decc44061ca415febaa763d3b4b86afc79572ba83f4d76d2a0f617R523)

Questions

Are we still implementing the same abstract method
[abstract method for create credential](https://github.com/hyperledger/aries-cloudagent-python/blob/cd4f1dc8fddc1194e0abc00ef4fb3d671745ad51/aries_cloudagent/indy/issuer.py#L113)

