# Keystone Aptos Mode Integration Guide

The goal of the Keystone hardware wallet is to provide users with a reliable and quality signer by protecting users'
private keys so Keystone seeks to integrate with existing wallets and applications.

We have drafted a QR Code data transmission, Here is the
link [QRCODE](https://github.com/KeystoneHQ/Keystone-developer-hub/issues/8), hopefully later we can propose an AIP.

## Firmware Requirement

You need Keystone Multi-coin Firmware support Aptos for the integration. Find it
from [here](https://keyst.one/firmware?locale=en).

## Wallet Integration

In order to work with Keystone, a wallet can follow the process:

1. Keystone provides the public key information to the wallet via QR code, to generate authentication key, which then
   can be used to search the Atops addresses on chain.
2. The wallet generates the unsigned transactions, sends it to Keystone to sign via QR Code.
3. Keystone would then sign the data and provide a signature as well as the related public key, back to the wallet via
   QR Code.
4. The wallet receives the signature and public key, constructs the signed data (transaction), and performs the
   following activities like broadcasting the transaction, etc.

## Library

[`@keystonehq/bc-ur-registry-aptos`](https://www.npmjs.com/package/@keystonehq/bc-ur-registry-aptos)

[`@keystonehq/animated-qr`](https://www.npmjs.com/package/@keystonehq/animated-qr)

[`@keystonehq/aptos-keyring`](https://www.npmjs.com/package/@keystonehq/aptos-keyring)

## Example

With the help of `@keystone/aptos-keyring`, wallets can easily integrate with Keystone without caring too much about UR.
`@keystone/aptos-keyring` encapsulates the details like encode/decode UR, construct UR from data etc. 

Wallets only need to pay some attention to the UI like display the qr code or render the camera to scan the QR
code.

`@keystone/aptos-keyring` uses `InteactionProvider` to handle the UI interaction, by default it uses `@keystone/sdk` as
the interaction provider to interact with wallet, it's functional but less UI designed, you can define your own
interaction provider by extending the `BaseKeyring`. With `DefaultKeyring` the integration is very simple:

### Connect Keystone

```ts
import {DefaultKeyring} from '@keystonehq/aptos-keyring';

const connectWallet = aysnc()=>{
...
    // Init a keyring instance 
    const keyring = DefaultKeyring.getEmptyKeyring();

    // This will call the video camera.
    // by scanning the QR code displayed in Keystone HardWare Wallet, The keyring will be initalized with public key, hdPath inforamtion.
    await keyring.readKeyring();
   console.log(keyring.getPubKeys()[0].pubKey);  // 8e53e7b10656816de70824e3016fc1a277e77825e12825dc4f239f418ab2e04e
...
}

```

![readKeyring waiting for scan QR code](../pics/aptos_read_keyring.png)

### Sign Transaction

```ts
// In your Transfer component, call the `keyring.signTransaction`.
// It will render the `aptos-sign-request` UR, 
// This promised will be resolved by scanning the `aptos-signature` UR generated from Keystone HardWare Wallet.
const {signature, authPubKey} = await keyring.signTransaction(authPubKey, signData);
console.log(signature.toString('hex')); // 47e7b510784406dfa14d9fd13c3834128b49c56ddfc28edb02c5047219779adeed12017e2f9f116e83762e86f805c7311ea88fb403ff21900e069142b1fb310e
console.log(authPubKey.toString('hex')); // 8e53e7b10656816de70824e3016fc1a277e77825e12825dc4f239f418ab2e04e
```

![transaction to be signed](../pics/aptos_sign_request.png)

## Customize the interaction provider

If you want to define your own interaction provider, you'll need to implement the `InteractionProvider` interface.

```ts
export interface InteractionProvider {
    readCryptoMultiAccounts: () => Promise<CryptoMultiAccounts>;
    requestSignature: (
        signRequest: AptosSignRequest,
        requestTitle?: string,
        requestDescription?: string
    ) => Promise<AptosSignature>;
}
```

Here is an example
of [MetaMask Interaction Provider](https://github.com/KeystoneHQ/keystone-airgaped-base/blob/master/packages/metamask-airgapped-keyring/src/MetaMaskInteractionProvider.ts)
.

By defining your own interaction provider, wallets can customize the `UI Components` by subscribing to the specific event
emitted by the interaction provider.

### Define your own interaction provider

```ts
export class WalletInteractionProvider extends EventEmitter implements InteractionProvider {
    static instance: WalletInteractionProvider;

    constructor() {
        super();
        if (WalletInteractionProvider.instance) {
            return WalletInteractionProvider.instance;
        }
       WalletInteractionProvider.instance = this;
    }

...
    requestSignature = (signRequest: AptosSignRequest) => {
        return new Promise((resolve, reject) => {
            const ur = signRequest.toUR();
            const requestIdBuffer = signRequest.getRequestId();
            const requestId = uuid.stringify(requestIdBuffer);
            const signPayload = {
                requestId,
                payload: {
                    type: ur.type,
                    cbor: ur.cbor.toString("hex"),
                }
            };
            this.emit('signRequest', requestPayload);
            this.once(`${requestId}-signed`, (cbor: string) => {
                const aptosSignature = AptosSignature.fromCBOR(Buffer.from(cbor, "hex"));
                resolve(aptosSignature);
            });
        });
    };
...
}
```

### Keyring with customized interaction provider

```ts
export class WalletKeyring extends BaseKeyring {
    getInteraction = (): WalletInteractionProvider => {
        return new WalletInteractionProvider();
    };
}
```

### UI Component

```tsx
// '@keystonehq/animated-qr' makes the handle of multiple QR codes much easier.
import {AnimatedQRCode} from '@keystonehq/animated-qr';

export default class ConfirmKeystoneTransaction extends Component {
    constructor(props) {
        super(props);
        this.state = {
            signRequest: null
        }
        this.props.wallet.keyring.getEventEmitter().on('signRequest', (signRequest) => {
            this.setState(() => ({
                signRequest
            }));
        });
    }
    return <AnimatedQRCode cbor={signRequest?.cbor} type={'sol-sign-request'} options={{ size: isSmallView ? 300 : 400 }} />
    
}

```
