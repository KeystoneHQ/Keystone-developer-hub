# Tron qr code data transmission protocol

## Abstract

This is the process and data transmission standard via QR Code for offline signer to work with watch-only wallet on Tron.

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

Because of these advantages, the QR Code data transmission is a better choice. However, there is no standard specification for QR usage in Tron.
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
For sending the unsigned data from a watch-only wallet to an offline signer. a new bc-ur type `tron-sign-request-kt` will be introduced for encoding the signing request.

#### CDDL for Tron Sign Request.
The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.

```
; Metadata for the signing request for Tron.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `derivation-path` is the path of the private key to sign the data
; `origin` is the origin of this sign request. like watch-only wallet name.
; `address` is the Tron address of the signing type for verification purpose which is optional 

tron-sign-request-kt = (
    ?request-id: uuid,
    sign-data: bytes,
    derivation-path: #5.304(crypto-keypath), ;the key path for signing this request
    ?origin: text,
    ?address: bytes
)

request-id = 1
sign-data = 2
derivation-path = 3
address = 4
origin = 5
```

`sign-data` currently supports transfer, includes Tron transfer, Trc10 and Trc20 token transfer.  
`sign-data` is JSON string in bytes, data structure

```
{
  "to": string, // to address
  "from": string, // from address
  "value": string, // send amount
  "fee": number, // fee
  "contractAddress": string, // optional, token contract address for TRC20 transfer only
  "latestBlock": {
    "hash": string // latest block hash in hex
    "number": number, // latest block number
    "timestamp": number, // latest block timestamp
  },
  "token": string, // "TRX" for coin transfer, token id for TRC10, field can be undefined for TRC20 transfer 
  "override": {  // optional
    "tokenShortName": string, // the short name of token
    "tokenFullName": string, // the full name of token
    "decimals": 0 // the decimals of token
  }
}
```

TRC10 transfer example
```json
{
  "to": "TKCsXtfKfH2d6aEaQCctybDC9uaA3MSj2h",
  "from": "TXhtYr8nmgiSp3dY3cSfiKBjed3zN8teHS",
  "value": "1",
  "fee": 100000,
  "latestBlock": {
    "hash": "6886a76fcae677e3543e546a43ad4e5fc6920653b56b713542e0bf64e0ff85ce",
    "number": 16068126,
    "timestamp": 1578459699000
  },
  "token": "1001090",
  "override": {
    "tokenShortName": "TONE",
    "tokenFullName": "TronOne",
    "decimals": 18
  }
}
```

#### Example
TBD

### Offline signers provide the signature to watch-only wallets.
After signing the data offline signer should send the signature back to the watch-only wallet. and a new bc-ur type called `tron-signature-kt` introduced here to encode the data.

#### CDDL for Sol Signature.
The following specification is written in Concise Data Definition Language [CDDL].

```
tron-signature  = (
    request-id: uuid,
    signature: bytes
)
```

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References
BC-UR https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md
