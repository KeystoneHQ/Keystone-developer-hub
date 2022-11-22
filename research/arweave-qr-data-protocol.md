# Arweave qr code data transmission protocol

## Abstract

This is the process and data transmission standard via QR Code for offline signer to work with watch-only wallet on
Arweave.

## Motivation

Currently, more and more users would like to use complete offline signers like hardware wallets, mobile phones on
offline mode to manage their private keys. In order to sign transaction or data, these offline signers have to work with
a watch-only wallet. these watch-only will prepare the data to be signed.
Currently, the data transmission method between offline signers and watch-only wallet will include QR Code,
usb,bluetooth and file transfer. Compare with other data transmission method like usb, bluetooth, and file transfer, the
QR Code data transmission have these advantages.

- Transparency and Security: Compared to usb or bluetooth, user can easily decode the data in QR Code (with the help of
  some tools), it can let user know what they are going to sign. this transparency can provide more security.
- Better Compatibility: Compared to usb and Bluetooth, the QR Code data transmission have better compatibility, normally
  it will not be broken by other software changes like browser upgrade, system upgrade etc.
- Better User Experience: The QR Code data transmission can provide much better user experience compare to usb,
  bluetooth and file transfer especially on the mobile environment.
- Smaller Attack Surface. usb and bluetooth have a higher attack surface than QR-Codes.

Because of these advantages, the QR Code data transmission is a better choice. However, there is no standard
specification for QR usage in Arweave.
This article presents a standard process and data transmission protocol for offline signers to work with the watch-only
wallet.

## Specification

**Offline signer**: the offline signer is a device or application which holds user private keys and does not have
network access.

**Watch-only wallet**: the watch only wallet is the wallet which has network access and will interact with blockchain.

## Process:

In order to work with offline signers, the watch-only wallet should follow the following process.

1. offline signers provide public key information to watch-only wallets to generate addresses and sync balance etc via
   QR Code.
2. a watch-only wallet generates the unsigned data and sends it to an offline signer to sign it via QR Code.
3. an offline signer signs the data and provides a signature back to the watch-only wallet via QR Code.
4. a watch-only wallet gets the signature and constructs the signed data (transaction) and performs the following
   actions like broadcasting the transaction etc.

## Data transmission protocol

Since one QR Code can contain a limited size of data, the animated QR Codes should be included for data transmission.The
BC-UR defined by BlockchainCommons have published a series data transmission protocol called br-ur. It provides a basic
method for encoding data to animated QR Code. this article will use the bc-ur and extend its current definition. all the
data are encoded by CBOR and the CDDL is the data definition language for CBOR. For more info about bc-ur, please check
out (here)[https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md].

### Setup watch-only wallet by offline signer.

In order to let a watch-only wallet collect information from blockchain, the offline signer should provide public keys
to watch-only wallets which generate addresses from these keys and query info like balance from blockchain.

In this case, offline signers will provide the public keys. A new UR type `crypto-arweave-account` will be used import arweave account.

#### CDDL for arweave account

```
; `master-fingerprint` is the identifier for this arweave account.
; `key-data` is the RSA public key modulus for generating the arweave account address.
; `device` is the origin of the account. like watch-only wallet name.

account = {
    master-fingerprint: uint32, ; Master fingerprint (fingerprint for the master public key as per BIP32)
    key-data: key-data-bytes, ; RSA public key modulus.
    ? device: text ; Indicates the origin of the account, e.g. 'Keystone'.
}

master-fingerprint = 1
key-data = 2
device = 3
```

#### Example:

Test Data:
> TBD

QR：
> TBD

### Sending the unsigned data from wallet-only wallet to offline signer.

For sending the unsigned data from a watch-only wallet to an offline signer. a new bc-ur type `arweave-sign-request` will
be introduced for encoding the signing request, multiple sign requests are supported.

#### CDDL for Arweave Sign Request.

The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.

```
; Metadata for the signing request for Arweave.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `sign-type` is the data type to be signed, eg `sign-type-transaction` for signing transaction, `sign-type-data-item` for signing data item. 
; `salt-len` is the `rsa-pss` salt length, zero for deterministic signature.
; `origin` is the origin of this sign request. like watch-only wallet name.
; `account` is the Arweave account of the signing type for verification purpose which is optional.

arweave-sign-request = (
    ?request-id: uuid,
    sign-data: [+ bytes],
    sign-type: int .default sign-type-transaction, ;sign type identifier,
    salt_len: int, ;RSA-PSS salt length.
    ?origin: text,
    ?account: bytes,
)

request-id = 1
sign-data = 2
sign-type = 3
account = 4
origin = 5
```

#### Example

TBD

### Offline signers provide the signature to watch-only wallets.

After signing the data offline signer should send the signature back to the watch-only wallet. and a new bc-ur type
called `arweave-signature` introduced here to encode the data, multiple signatures are supported.

#### CDDL for Sol Signature.

The following specification is written in Concise Data Definition Language [CDDL].

```
arweave-signature  = (
    request-id: uuid,
    signature: [+ bytes]
)
```

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References

BC-UR https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md


