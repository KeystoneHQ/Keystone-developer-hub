# UTXO qr code data transmission protocol

## Abstract

This is the process and data transmission standard via QR Code for offline signer to work with watch-only wallet on LTC, BCH, DASH, etc.

## Motivation

Currently, more and more users would like to use complete offline signers like hardware wallets,
mobile phones on offline mode to manage their private keys. In order to sign transaction or data,
these offline signers have to work with a watch-only wallet. these watch-only will prepare the data to be sign.

Currently, the data transmission method between offline signers and watch-only wallet will include QR Code, usb, bluetooth and file transfer.
Compare with other data transmission method like usd, bluetooth, and file transfer, the QR Code data transmission have these advantages.

- Transparency and Security: Compared to usb or bluetooth, user can easily decode the data in QR Code (with the help of some tools),it can let user know what they are going to sign. this Transparency can provide more security.
- Better Compatibility: Compared to usb and Bluetooth, the QR Code data transmission have better compatibility, normally it will not be break by other software change like browser upgrade. system upgrade etc.
- Better User experience: The QR Code data transmission can provide much better user experience compare to usb, bluetooth and file transfer especially on the mobile environment.
- Smaller attack surface. USB and Bluetooth have a higher attack surface than QR-Codes.

Because of these advantages, the QR Code data transmission is a better choice. However, there is no standard specification for QR usage in LTC, BCH, DASH, etc.
This article presents a standard process and data transmission protocol for offline signers to work with the watch-only wallet.

## Specification

**Offline signer**: the offline signer is a device or application which holds user private keys and does not have network access.

**Watch-only wallet**: the watch only wallet is the wallet which has network access and will interact with blockchain.

## Process
In order to work with offline signers, the watch-only wallet should follow the following process.
1. offline signers provide public key information to watch-only wallets to generate addresses and sync balance etc via QR Code.
2. a watch-only wallet generates the unsigned data and sends it to an offline signer to sign it via QR Code.
3. an offline signer signs the data and provides a signature back to the watch-only wallet via QR Code.
4. a watch-only wallet gets the signature and constructs the signed data (transaction) and performs the following action like broadcasting the transaction etc.

## Data transmission protocol

Since one QR Code can contain a limited size of data, the animated QR Codes should be included for data transmission.
The BC-UR defined by BlockchainCommons have published a series data transmission protocol called br-ur.
It provides a basic method for encoding data to animated QR Code. this article will use the bc-ur and extend its current definition.
all the data are encoded by CBOR and the CDDL is the data definition language for CBOR. For more info about bc-ur,
please check out [here](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md).

### Sending the unsigned data from wallet-only wallet to offline signer.
For sending the unsigned data from a watch-only wallet to an offline signer. a new bc-ur type `utxo-sign-request` will be introduced for encoding the signing request.

#### CDDL for UTXO Sign Request.
The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.

```
; Metadata for the signing request for UTXO.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `origin` is the origin of this sign request. like watch-only wallet name. 

utxo-sign-request = (
    ?request-id: uuid,
    sign-data: bytes,
    ?origin: text,
)

request-id = 1
sign-data = 2
origin = 3
```

`sign-data` currently supports transfer. `sign-data` is protobuf, check data structure [here](https://github.com/KeystoneHQ/keystone-sdk-rust/blob/master/libs/ur-registry/proto/base.proto).


#### Example
TBD

### Offline signers provide the signature to watch-only wallets.
After signing the data offline signer should send the signature back to the watch-only wallet. and a new bc-ur type called `utxo-sign-result` introduced here to encode the data.

#### CDDL for Sol Signature.
The following specification is written in Concise Data Definition Language [CDDL].

```
utxo-sign-result = (
    request-id: uuid,
    sign-result: bytes
)
```

Check `sign-result` data structure [here](https://github.com/KeystoneHQ/keystone-sdk-rust/blob/master/libs/ur-registry/proto/base.proto).

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References
BC-UR https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md
