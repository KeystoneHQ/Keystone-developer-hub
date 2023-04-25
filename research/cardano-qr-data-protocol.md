# Cardano qr code data transmission protocol

## Abstract

This is the process and data transmission standard via QR Code for offline signer to work with watch-only wallet on Cardano.

## Specification

**Offline signer**: the offline signer is a device or application which holds user private keys and does not have network access.

**Watch-only wallet**: the watch only wallet is the wallet which has network access and will interact with blockchain.

## Process:
In order to work with offline signers, the watch-only wallet should follow the following process.
1. offline signers provide public key information to watch-only wallets to generate addresses and sync balance etc via QR Code.
2. a watch-only wallet generates the unsigned data and sends it to an offline signer to sign it via QR Code.
3. an offline signer signs the data and provides a signature back to the watch-only wallet via QR Code.
4. a watch-only wallet gets the signature and constructs the signed data (transaction) and performs the following actions like broadcasting the transaction etc.

## Data transmission protocol

Since one QR Code can contain a limited size of data, the animated QR Codes should be included for data transmission.The BC-UR defined by BlockchainCommons have published a series data transmission protocol called br-ur. It provides a basic method for encoding data to animated QR Code. this article will use the bc-ur and extend its current definition. all the data are encoded by CBOR and the CDDL is the data definition language for CBOR. For more info about bc-ur, please check out (here)[https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md].


### Setup watch-only wallet by offline signer.
In order to let a watch-only wallet collect information from blockchain, the offline signer should provide public keys to watch-only wallets which generate addresses from these keys and query info like balance from blockchain. 

In this case, offline signers will provide the public keys and derivation path. the UR Type called `crypto-hdkey` will be used to encode these data. and the derivation path will be encoded as `crypto-keypath` which defined in bc-ur.Also, a new UR type `crypto-multi-accounts` will be used if we want to import multiple accounts in one animated QR. 

#### CDDL for Key Path
The following specification is written in Concise Data Definition Language [CDDL].

``` 
; Metadata for the derivation path of a key.
;
; `source-fingerprint`, if present, is the fingerprint of the
; ancestor key from which the associated key was derived.
;
; If `components` is empty, then `source-fingerprint` MUST be a fingerprint of
; a master key.
;
; `depth`, if present, represents the number of derivation steps in
; the path of the associated key, even if not present in the `components` element
; of this structure.
    crypto-keypath = {
        components: [path-component], ; If empty, source-fingerprint MUST be present
        ? source-fingerprint: uint32 .ne 0 ; fingerprint of ancestor key, or master key if components is empty
        ? depth: uint8 ; 0 if this is a public key derived directly from a master key
    }

    path-component = (
        child-index / child-index-range / child-index-wildcard-range,
        is-hardened
    )

    uint32 = uint .size 4
    uint31 = uint32 .lt 2147483648 ;0x80000000
    child-index = uint31
    child-index-range = [child-index, child-index] ; [low, high] where low < high
    child-index-wildcard = []

    is-hardened = bool

    components = 1
    source-fingerprint = 2
    depth = 3
```

#### CDDL for derivation key
Since the main purpose of the key is to transfer public key data. the defination of crypto-hdkey will be kept only public keys.
For HD-derivation-key, which will present the key data and its derivation path.
The following specification is written in Concise Data Definition Language [CDDL] and includes the crypto-keypath spec above.
```
; A derived key must be public, has an optional chain code, and
; may carry additional metadata about its use and derivation.
; To maintain isomorphism with [BIP32] and allow keys to be derived from
; this key `chain-code`, `origin`, and `parent-fingerprint` must be present.
; If `origin` contains only a single derivation step and also contains `source-fingerprint`,
; then `parent-fingerprint` MUST be identical to `source-fingerprint` or may be omitted.
derived-key = (
	key-data: key-data-bytes,
	? chain-code: chain-code-bytes       ; omit if no further keys may be derived from this key
	? origin: #6.304(crypto-keypath),    ; How the key was derived
	? name: text,                        ; A short name for this key.
)

; If the `use-info` field is omitted, defaults (mainnet BTC key) are assumed.
; If `cointype` and `origin` are both present, then per [BIP44], the second path
; component's `child-index` must match `cointype`.

key-data = 1
chain-code = 2
origin = 3
name = 4

uint8 = uint .size 1
key-data-bytes = bytes .size 32
chain-code-bytes = bytes .size 32
```
If the chain-code is provided is can be used to derive child keys and if the chain code is not provided it just an solo key and origin can be provided to indicate the derivation key path.

### Request key from off-line signer
In some cases, e.g. Eternl wallet, online watch-only wallet may want to request specific public keys from off-line signer.
Please refer to [key-derivation-call](key-derivation-call.md) for more info.

### Sending the unsigned data from wallet-only wallet to offline signer.
For sending the unsigned data from a watch-only wallet to an offline signer. a new bc-ur type `cardano-sign-request` will be introduced for encoding the signing request, multiple sign requests are supported.

#### CDDL for Cardano Sign Request.
The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.

```
; Metadata for the signing request for Cardano.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `utxos` is the transactions inputs.
; `origin` is the origin of this sign request. like watch-only wallet name.

cardano-sign-request = (
    ?request-id: uuid,
    sign-data: bytes, ; transaction cbor hex
    utxos: [+ cardano-utxo] ; will be checked with sign-data 
    ?origin: text,
)

request-id = 1
sign-data = 2
utxos = 3
origin = 4

; Metadata for the UTXO in cardano
; `transaction-hash` input's previous transaction hash
; `index` input's index in previous transaction
; `amount` input's amount , should be u64
; `key-path` the related private key path of this input
; `address` the display address of this input, might be a base address or an enterprise address
cardano-utxo = (
    transaction-hash: bytes .size 32
    index: uint,
    amount: uint,
    key-path: #6.304(crypto-keypath),
    address: text, 
)

transaction-hash = 1
index = 2
amount = 3
key-path = 4
address = 5
```

#### Example
TBD

### Offline signers provide the signature to watch-only wallets.
After signing the data offline signer should send the signature back to the watch-only wallet. and a new bc-ur type called `cardano-signature` introduced here to encode the data, multiple signatures are supported.

#### CDDL for Sol Signature.
The following specification is written in Concise Data Definition Language [CDDL].

```
cardano-signatures  = (
    request-id: uuid,
    signatures: [+ bytes]; might be multiple signatures
)

request-id = 1
signatures = 2

```

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References
BC-UR https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md


