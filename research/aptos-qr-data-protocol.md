# Aptos qr code data transmission protocol

## Abstract

This is the process and data transmission standard via QR Code for offline signer to work with watch-only wallet on Aptos.

## Motivation

Currently, more and more users would like to use complete offline signers like hardware wallets, mobile phones on offline mode to manage their private keys. In order to sign transaction or data, these offline signers have to work with a watch-only wallet. these watch-only will prepare the data to be sign. 
Currently, the data transmission method between offline signers and watch-only wallet will include QR Code, usb,bluetooth and file transfer. Compare with other data transmission method like usb, bluetooth, and file transfer, the QR Code data transmission have these advantages.

- Transparency and Security: Compared to usb or bluetooth, user can easily decode the data in QR Code (with the help of some tools), it can let user know what they are going to sign. this transparency can provide more security.
- Better Compatibility: Compared to usb and Bluetooth, the QR Code data transmission have better compatibility, normally it will not be broken by other software changes like browser upgrade, system upgrade etc.
- Better User Experience: The QR Code data transmission can provide much better user experience compare to usb, bluetooth and file transfer especially on the mobile environment.
- Smaller Attack Surface. usb and bluetooth have a higher attack surface than QR-Codes.

Because of these advantages, the QR Code data transmission is a better choice. However, there is no standard specification for QR usage in Aptos.
This article presents a standard process and data transmission protocol for offline signers to work with the watch-only wallet.

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

In this case, offline signers will provide the public keys and derivation path. the UR Type called `crypto-hdkey` will be used to encode these data. and the derivation path will be encoded as `crypto-keypath` which defined in bc-ur.Also, a new UR type `aptos-multi-accounts` will be used if we want to import multiple accounts in one animated QR. 

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

key-data = 3
chain-code = 4
origin = 6
name = 9

uint8 = uint .size 1
key-data-bytes = bytes .size 33
chain-code-bytes = bytes .size 32
```
If the chain-code is provided is can be used to derive child keys and if the chain code is not provided it just an solo key and origin can be provided to indicate the derivation key path.

#### Example:
Test Data:
> TBD

QR：
> TBD

#### CDDL for multiple accounts

The following specification is written in Concise Data Definition Language [CDDL].

```
key_exp = #6.303(crypto-hdkey)

accounts = {
    master-fingerprint: uint32, ; Master fingerprint (fingerprint for the master public key as per BIP32)
    keys: [+ key_exp] ; Different auth keys for a offline signer.
    ? device: text ; Indicates the origin of these accounts, e.g. 'Keystone'
}

master-fingerprint = 1
keys = 2
device = 3

```

#### Example:
Test Data:
> TBD

QR：
> TBD


### Sending the unsigned data from wallet-only wallet to offline signer.
For sending the unsigned data from a watch-only wallet to an offline signer. a new bc-ur type `aptos-sign-request` will be introduced for encoding the signing request, multiple sign requests are supported.

#### CDDL for Aptos Sign Request.
The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.


```
; Metadata for the signing request for aptos.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `authentication-key-derivation-paths` is the path of the private keys to sign the data. For sign-type-single type, there is only one value in the list. For sign-type-multi, there is N values in the list, N is the "K-of-N multisig authentication".
; `origin` is the origin of this sign request. like watch-only wallet name.
; `accounts ` is the Aptos account of the signing type for verification purpose which is optional  
; `type` is a field used to distinguish transaction types: single-sign or multi-sign or message.

path_exp = #6.304(crypto-keypath)

aptos-sign-request = (
    request-id: uuid,
    sign-data: bytes,
    authentication-key-derivation-paths: [+ path_exp], ;the key paths for signing this request
    ?accounts: [+ bytes],
    ?origin: text,
    type: int. default sign-type-single, ;sign type identifier
)

request-id = 1
sign-data = 2
authentication-key-derivation-paths = 3
accounts = 4
origin = 5
type = 6

sign-type-single = 1
sign-type-multi = 2
sign-type-message = 3


```

#### Example
TBD

### Offline signers provide the signature to watch-only wallets.
After signing the data offline signer should send the signature back to the watch-only wallet. and a new bc-ur type called `aptos-signature` introduced here to encode the data, multiple signatures are supported.

#### CDDL for Aptos Signature.
The following specification is written in Concise Data Definition Language [CDDL].

```
; Metadata for the signature for aptos.
; `request-id` is the identifier for this signature.
; `signature ` is the signature of the transaction.
; `authentication-public-key` is the public key corresponding to the private key that signed the transaction
aptos-signature  = (
    request-id: uuid,
    signature: bytes,
    ?authentication-public-key: bytes,;In case of multiple signatures, it must be passed to indicate which signer signed the transaction
)

request-id = 1
signature = 2
authentication-public-key = 3
```

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References
BC-UR https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md
