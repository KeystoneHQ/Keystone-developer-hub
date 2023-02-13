# EVM qr code data transmission protocol 

#### CDDL for EVM Sign Request.
The following specification is written in Concise Data Definition Language [CDDL].
UUIDs in this specification notated uuid are CBOR binary strings tagged with #6.37, per the IANA CBOR Tags Registry.


```
; Metadata for the signing request for ethereum.
; request-id is the identifier for this signing request.
; sign-data is the data to be signed.
; data-type is to indicate what is the data to be signed. Different chains have different type values.
; custom-chain-identifier is to show which chain it is. 
; derivation-path is the path of the private key to sign the data.
; address is the ethereum address of the signing type for verification purpose which is optional
; origin is to show where the data from.

sign-data-type = {
    type: int .default 1 arbitrary data; type data defined as following value
}
 
	evm-arbitrary-data = 1； 
	evm-cosmos-animo-data = 2;
	evm-cosmos-dierect-data = 3;

eth-sign-request = (
    sign-data: sign-data-bytes, ; sign-data is the data to be signed by offline signer, currently it can be unsigned transaction of evmos
    data-type: sign-data-type,
    ?custom-chain-identifier: int,  ; it will be the chain id of EVM related blockchain or empty
    derivation-path: #5.304(crypto-keypath), ;the key path for signing this request
    ?request-id: uuid, ; the uuid for this signing request
    ?address: eth-address-bytes,    ;verification purpose for the address of the signing key
    ?origin: text  ;the origin of this sign request
)

request-id = 1
sign-data = 2
data-type = 3
custom-chain-identifier = 4 
derivation-path = 5
address = 6
origin = 7
eth-address-bytes = bytes .size 20
sign-data-bytes = bytes 
```

#### CDDL for Evm Signature.
The following specification is written in Concise Data Definition Language [CDDL].

```
evm-signature  = (
    request-id: uuid,
    signature: evm-signature-bytes
)
evm-signature-bytes = bytes .size, variable length ;currently the signature of the signing request (r,s) is 64 for EVMOS.
```