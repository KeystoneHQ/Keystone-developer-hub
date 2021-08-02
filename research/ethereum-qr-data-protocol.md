# Ethereum qr code data transmission protocol 

## Simple Summary

Currently more and more users would like to use complete offline singers like air gapped hardware wallets, mobile phones in offline mode to manage their private keys. In order to interact with blockchain. a "watch-only" wallet (like metamask etc.) which have network access are required to pair with these offline signers. Currently there is no standard for how the offline signer can work with "watch-only" wallets via QR Code. so This EIP aims to solve this issue.

## Abstract

This is the process and data transmission standard via QR Code for offline signer to work with watch-only wallet 

## Motivation

Currently more and more users would like to use complete offline signers like hardware wallets, mobile phones on offline mode to manage their private keys. In order to sign transaction or data, these offline signers have to work with an watch-only wallet. these watch-only will pepare the data to be sign. currently the data trasmission method between offline signers and watch-only wallet will include QR Code, usb,bluetooth and file transfer.Compare with other data trasmission method like usd, bluetooth, and file transfer, the QR Code data transmission have these advantages.

- Transparency and Security: Compared to usb or bluetooth, user can easily decode the data in QR Code (with the help of some tools), it can let user know what they are going to sign. this Transparency can provide more security.
- Better Compatibility: Compared to usb and Bluetooth, the QR Code data transmission have better compatibility, normally it will not be break by other software change like browser upgrade. system upgrade etc.
- Better User experience: The QR Code data transmission can provide more better user experience compare to usb, bluetooth and file transfer especially on the mobile environment.
- Smaller attack surface. USB and Bluetooth have a higher attack surface than QR-Codes.

Because of these advantages, the QR Code data transmission is a better choice. But currently there is no standard for how the offline signer work with watch-only wallet and how the data can be encoded.
This EIP presents a standard process and data transmission protocol for offline signers to work with the watch-only wallet.

## Specification

**Offline signer**: the offline signer is a device or application which holds user private keys and does not have network access.

**Watch-only wallet**: the watch only wallet is the wallet which has network access and will interact with blockchain.

## Process:
In order to work with offline signers, the watch-only wallet should follow the following process.
1. offline signers provide public key information to watch-only wallets to generate addresses and sync balance etc via QR Code.
2. a watch-only wallet generates the unsigned data and sends it to an offline signer to sign it including transaction, typed data etc via QR Code.
3. an offline signer signs the data and provides a signature back to the watch-only wallet via QR Code.
4. a watch-only wallet gets the signature and constructs the signed data (transaction) and performs the following action like broadcasting the transaction etc.

## Data transmission protocol

Since one QR Code can contain a limited size of data, the animated QR Codes should be included for data transmission.The BC-UR defined by BlockchainCommons have published a series data transmission protocol called br-ur. It provides a basic method for encoding data to animated QR Code. this EIP will use the bc-ur and extend its current definition. all the data are encoded by CBOR and the CDDL is the data definition language for CBOR. For more info about bc-ur, please check out (here)[https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md].


### Setup watch-only wallet by offline signer. 
In order to let a watch-only wallet collect information from blockchain, the offline signer should provide public keys to watch-only wallets which generate addresses from these keys and query info like balance from blockchain. Currently most wallet service providers are following BIP-44 to generate the address. 

In this case, offline signers will provide the extended public keys and derivation path. the UR Type called `crypto-hdkey` will be used to encode these data. and the derivation path will be encoded as `crypto-keypath` which defined in in bc-ur

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
Since the main purpose of the key is to transfer public key data. the defination of crypto-hdkey will be kept on ly public keys. 
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
```
{
    "extendPublicKey": "xpub6Cq5iFcEz9gTANMrxck2Ee3zp18ZzfYNFU9aXxBi7kFYNY42FYaCr8fWBLkHo5Vw3MwWYeUjcVvkSonynexiHWEge4yfPP13J9JCB7pakEm",
    "path": "M/44'/60'/0'",
    "MasterFignerPrint":"cfc23f3f",
}
```
CBOR Binary:

```
A303582103DA1A04CC1509CD716E215F1C8A3D1530A6B4EFDACB04D1641EAA117342EFA4B1045820FDCA62CADED4C32CD914A3CEF84529504BC534C721997E196BE421162F3EB81D06D90130A20186182CF5183CF500F5021ACFC23F3F
```

UR:

```
ur:crypto-public-key/otaxhdclaxtncyaasfbzassnjsjtclhecelefsbzdyolqzwstnsbaattieckpkbyjkfwwsoxpaaahdcxzcsgidsguetysrdwtabbottoyafedtgdgrskeestclnlkbcfjeveclcmdlfmrocaamtaaddyoeadlncsdwykcsfnykaeykaocytksafhfhgdlsvlpr
```

QR：
![](./hd-key.png)

### Sending the unsigned data from wallet-only wallet to offline signer.
For sending the unsigned data from a watch-only wallet to an offline signer. a new bc-ur type `eth-sign-request` will be introduced for encoding the signing request.

#### CDDL for Eth Sign Request.
The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.


```
; Metadata for the signing request for ethereum.
; request-id is the identifier for this signing request.
; sign-data is the data to be signed
; data-type is to indicate what is the data to be signed.
; chain-id to show it is the mainnet or testnet
; derivation-path is the path of the private key to sign the data
; address is the ethereum address of the signing type for verification purpose which is optional


; `sign-data-type` is to indicate what is the data to be signed. 
; current have three types transaction, EIP 712 typed data or raw types
; 
sign-data-type = {
    type: int .default 1 transaction data; type data defined as following value
}

eth-transaction-data = 1; for unsigned transaction if will be the rlp encoding bytes
eth-typed-data = 2;
eth-raw-bytes=3;
eth-typed-transaction=4;


eth-sign-request = (
    sign-data: sign-data-bytes, ; sign-data is the data to be signed by offline signer, currently it can be unsigned transaction or typed data
    data-type: #3.401(sign-data-type),
    chain-id: int .default 1,
    derivation-path: #5.304(crypto-keypath), ;the key path for signing this request
    ?request-id: uuid, ; the uuid for this signing request
    ? address: eth-address-bytes,            ;verification purpose for the address of the signing key
)

request-id = 1
sign-data = 2
data-type = 3

chain-id = 4 ;it will be the chain id of ethereum related blockchain
derivation-path = 5
address = 6

eth-address-bytes = bytes .size 20


sign-data-bytes = bytes ; for unsigned transactions it will be the rlp encoding for unsigned transaction data and ERC 712 typed data it will be the bytes of json string.

```
the sign-data: the unsigned transactions it will be the rlp encoding for unsigned transaction data and ERC 712 typed data it will be the bytes of json string. and raw bytes can also supported.

#### Example
TBD

### Offline signers provide the signature to watch-only wallets.
After signing the data offline signer should send the signature back to the watch-only wallet. and a new bc-ur type called `eth-signature` introduced here to encode the data.

#### CDDL for Eth Signature.
The following specification is written in Concise Data Definition Language [CDDL].

```
eth-signature  = (
    request-id: uuid,
    signature: eth-signature-bytes
)

eth-signature-bytes = bytes .size 65; the signature of the signing request (r,v,s)
```

## Rationale

## Backwards Compatibility

## Test Cases

## Implementation

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References
BC-UR https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md


