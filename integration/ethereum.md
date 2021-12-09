# Keystone Web3 Mode (Ethereum) Integration Guide.
The goal of Keystone hardware wallet is to be the good signer to protect users' private keys, so Keystone would like to work with existing wallets and applications. Now, Keystone have provide with Web3 Mode which have worked with a lot of Ethereum wallets, like Metamask and a lot Dapps like Sushi Swap, Yearn, Gnosis Safe and other well known Dapps.

Now we have proposed an EIP for the QR Code data transmission. Here is the link for the [EIP](https://github.com/ethereum/EIPs/pull/4527)


## Wallet Integration
In order to work with Keystone, the wallet should follow the following process.
1. Keystone provides the public key information to the wallet to generate addresses, sync balances and etc via QR Codes.
2. The wallet generates the unsigned data and sends it to Keystone for signing via QR Code, data that can include transactions, typed data, and etc.
3. Keystone signs the data and provides a signature back to the wallet via QR Code.
4. The wallet receives the signature, constructs the signed data (transaction) and performs the following activities like broadcasting the transaction etc.

### Setting up the wallet with Keystone
In order to allow a wallet to collect information from the Ethereum blockchain, Keystone would need to provide the extended public key to the watch-only wallet in which the wallet will use them to generate addresses and query the necessary information from the Ethereum blockchain.

We use the `crypto-hdkey` to encode the extended public key and its derivation path, if you would like to know more about this,please check the `Specification` section of the [EIP](https://github.com/ethereum/EIPs/pull/4527)

Here is the sample QR Code image of extended public key.

#### library
we have published the library for help developer to extract the extended public key from the QRCode. 

Please check the library here：

ur-registry-eth: [source code](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-eth)

npm package: [`@keystonehq/bc-ur-registry-eth`](https://www.npmjs.com/package/@keystonehq/bc-ur-registry-eth)

#### Sample Code
Install the package `@keystonehq/bc-ur-registry-ethereum`

```js
yarn add @keystonehq/bc-ur-registry-ethereum
```

Usage:

```js

// this is the extended public key data from the QR Code
const UR = "UR:CRYPTO-HDKEY/PTAOWKAXHDCLAXAMLSDSDSFSYLLTFLGAAYLDFTBNMWRDPAHLPTJSFDOSDPONAAEENYHKMOYLREJLGOAAHDCXSEJORFDRBNSKKBESEHPMLDMTCYPMRSBNDRKBIDKIFLSSWLTIESDRHKTSZTHGHHINAHTAADEHOEADCSFNAOAEAMTAADDYOTADLNCSDWYKCSFNYKAEYKAOCYZCTEHDWKAXAXATTAADDYOEADLRAEWKLAWKAXAEAYCYPFKPEOASASISGRIHKKJKJYJLJTIHBKJOHSIAIAJLKPJTJYDMJKJYHSJTIEHSJPIEKBJZJTYN"
 
import { URRegistryDecoder } from '@keystonehq/bc-ur-registry-eth'

const decoder = new URRegistryDecoder();
decoder.receivePart(ur);
if(decoder.isSuccess()) {
    const cryptoHDkey = decoder.resultRegistryType();
    const path = cryptoHDkey.getOrigin().getPath()); // 44'/60'/0'
    const extenedPubKey = cryptoHDKey.getBip32Key(); //xpub6CwzeNGArLq2XuEwGtycrise31bP3dZe1cim6urwWvyJ4D4EStJtp7ppZfHbi8pQTiEyapmGQS7NMrEarbDwPdjNMkGKSQEtpXytCWcQvH4
} else {
    // logic for error handling
}
```

### Sending the unsigned data from the watch-only wallet to the offline signer

#### library

#### Sample Code

#### Sample Data


### The signature provided by offline signers to watch-only wallets


#### library

#### Sample Code

#### Sample Data



## Dapp Integration