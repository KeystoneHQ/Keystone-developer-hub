# Key Derivation Call

## Summary

Currently integrations between Keystone and other wallets meets more and more complex situations. For example, we are facing the situation that Keystone needs to receive a serials of arguments to generate Wallet QR codes.
So we propose to create an this protocol for Keystone to provide an excutable API.

## Work Flow

- Watch-Only wallet construct a Keystone Request as defination below, and generate a QR code with BC-UR; 
- Keystone scan the QR code(s) and excute the Keystone Call inside. 
    - Keystone then generate a QR code which presents the call execution result if the Keystone Call has an defination of the result. if so, watch-only wallet should receive this QR code(s) and do the rest stuff.

## CDDL for key-derivation-call
```
qr-hardware-call = (
    name: call-name, ; the purpose of this call
    params: call-params ; the params of this call
)

call-name = "key-derivation"
call-params = key-derivation-params ; may add others in the future

name = 1
params = 2

key-derivation-params = (
    keypath: [#6.304(crypto-keypath)],
    ? curve: curve_exp, ; default to "secp256k1"
    ? algo: derivation_algorithm_exp, ;default to "slip10"
    ? origin: string, ; indicates where is this call from
)

curve_exp = "secp256k1" / "ed25519"

derivation_algorithm_exp = "slip10" / "bip32ed25519"

keypath = 1
curve = 2
algo = 3
origin = 4
```

### Acceptable Combination

| curve     | derivation_algorithm |             |
| --------- | -------------------- | ----------- |
| secp256k1 | slip10               | OK          |
| secp256k1 | bip32ed25519         | Not Working |
| ed25519   | slip10               | OK          |
| ed25519   | bip32ed25519         | OK          |

## Call Execution Result

Keystone will conctruct a `crypto-multi-accounts` to presents the key specified by [keypath, curve, derivation_algorithm].

> crypto-multi-accounts: https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#cddl-for-multiple-accounts


## Test Vectors
TBD