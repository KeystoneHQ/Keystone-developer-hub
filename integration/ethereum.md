# Keystone Web3 Mode (Ethereum) Integration Guide.
The goal of the Keystone hardware wallet is to be a good signer to protect users' private keys, so Keystone would like to work with existing wallets and applications. Now Keystone has provided the Web3 Mode which has worked with a lot of Ethereum wallets, like Metamask and a lot of Dapps like Sushi Swap, Yearn, Gnosis Safe, and other well known Dapps.

Now we have proposed an EIP for the QR Code data transmission. Here is the link for the [EIP](https://github.com/ethereum/EIPs/pull/4527)


## Wallet Integration
In order to work with Keystone, the wallet should follow the following process.
1. Keystone provides the public key information to the wallet to generate addresses, sync balances and etc via QR Codes.
2. The wallet generates the unsigned data and sends it to Keystone for signing via QR Code, data that can include transactions, typed data, and etc.
3. Keystone signs the data and provides a signature back to the wallet via QR Code.
4. The wallet receives the signature, constructs the signed data (transaction), and performs the following activities like broadcasting the transaction, etc.

### Setting up the wallet with Keystone
In order to allow a wallet to collect information from the Ethereum blockchain, Keystone would need to provide the extended public key to the watch-only wallet in which the wallet will use to generate addresses and query the necessary information from the Ethereum blockchain.

We use the `crypto-hdkey` to encode the extended public key and its derivation path, if you would like to know more about this, please check the `Specification` section of the [EIP](https://github.com/ethereum/EIPs/pull/4527)

Here is the sample QR Code image of the extended public key.

#### Library
we have published the library to help developers to extract the extended public key from the QRCode. 

Please check the library here：

ur-registry-eth: [source code](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-eth)

npm package: [`@keystonehq/bc-ur-registry-eth`](https://www.npmjs.com/package/@keystonehq/bc-ur-registry-eth)

#### Sample
Install the package `@keystonehq/bc-ur-registry-ethereum`

```js
yarn add @keystonehq/bc-ur-registry-ethereum
```

sample code:

```ts
// this is the extended public key data from the QR Code
const UR = "UR:CRYPTO-HDKEY/PTAOWKAXHDCLAXAMLSDSDSFSYLLTFLGAAYLDFTBNMWRDPAHLPTJSFDOSDPONAAEENYHKMOYLREJLGOAAHDCXSEJORFDRBNSKKBESEHPMLDMTCYPMRSBNDRKBIDKIFLSSWLTIESDRHKTSZTHGHHINAHTAADEHOEADCSFNAOAEAMTAADDYOTADLNCSDWYKCSFNYKAEYKAOCYZCTEHDWKAXAXATTAADDYOEADLRAEWKLAWKAXAEAYCYPFKPEOASASISGRIHKKJKJYJLJTIHBKJOHSIAIAJLKPJTJYDMJKJYHSJTIEHSJPIEKBJZJTYN"
 
import { URRegistryDecoder } from '@keystonehq/bc-ur-registry-eth'

const decoder = new URRegistryDecoder();

decoder.receivePart(UR);
if(decoder.isSuccess()) {
    const cryptoHDkey = decoder.resultRegistryType();
    const path = cryptoHDkey.getOrigin().getPath()); // 44'/60'/0'
    const extenedPubKey = cryptoHDKey.getBip32Key(); //xpub6CwzeNGArLq2XuEwGtycrise31bP3dZe1cim6urwWvyJ4D4EStJtp7ppZfHbi8pQTiEyapmGQS7NMrEarbDwPdjNMkGKSQEtpXytCWcQvH4
    return
} else if(decoder.isError()){
    // logic for error handling
    throw new Error() 
}
```

### Sending the unsigned data from the wallet to Keystone
For sending the unsigned data to Keystone, the data have to be encoded into the QR Codes. We will use the new UR Type `eth-sign-request` for this. For more info about the `eth-sign-request`, please check the `Specification` section of the [EIP](https://github.com/ethereum/EIPs/pull/4527)

When generating an eth-sign-request, we current defined four types of the unsigned data. 
```ts
enum DataType {
    transaction = 1, // For the legacy transaction, the rlp encoding of the unsigned data.
    typedData = 2, // For the EIP-712 typed data. Bytes for the json string.
    personalMessage = 3, // For the personal message signing. 
    typedTransaction = 4 // For the typed transaction, like the EIP-1559 transaction.
}
```

And for one sign request, these fields should be filled.

```ts
signData: Buffer, // the unsigend data bytes, for
signDataType: DataType, // supported data type
hdPath: string,  // derivation path for the signign key
xfp: string, // master fingerprint provided by Keystone when syncing with wallet
uuidString?: string,  // uuid for the request
chainId?: number, // chain id for this signing, optional
address?: string, // address for request this signing, optional
origin?: string // source of the request, wallet name etc, optional
```

#### Library
We have also provided the `eth-sign-request` in the `ur-registry-eth` library.

#### Sample

we will use the `@ethereumjs` to show how to generate the request

##### Legacy Transaction

```ts
import { Transaction } from '@ethereumjs/tx';
import Common, { Hardfork } from '@ethereumjs/common';
import { BN } from 'ethereumjs-util';
import {
    CryptoHDKey,
    generateAddressfromXpub,
    findHDpatfromAddress,
    EthSignRequest,
    DataType,
    ETHSignature,
} from '@keystonehq/bc-ur-registry-eth';

const _txParams = {
    to: "0x121212121212121212121212", 
    gasLimit: 200000,
    gasPrice: 120000000000,
    data: "0x",
    nonce: 1,
    value: txParams.value,
};

const tx = Transaction.fromTxData(_txParams, { common: this._common, freeze: false });
        
tx.v = new BN(tx.common.chainId());
        
tx.r = new BN(0);
        
tx.s = new BN(0);

const unsignedBuffer = tx.serialize(); // generate the unsinged transaction bytes
const requestId = uuid.v4();
const addressPath = "M/44'/60'/0'/0/0"

const ethSignRequest = EthSignRequest.constructETHRequest(
    unsignedBuffer,
    DataType.transaction,
    addressPath,
    "12345678", // master fingerprint
    requestId,
    1, // chainId
    "0xfromaddress",
);

const eachChunkNumberInEachQR = 400; // specify the each chunk Number in single QR Code

// get the ur decoder
const urDecoder = ethSignRequest.toUREncoder(eachChunkNumberInEachQR);

// render each chunk of data in sign QR Code
while (true) {
    renderQR(urDecoder.nextPart());
}
```

##### Typed Transaction
We will use the EIP-1559 transaction as an example.

``` ts
import { Transaction } from '@ethereumjs/tx';
import Common, { Hardfork } from '@ethereumjs/common';
import { BN } from 'ethereumjs-util';
import {
    CryptoHDKey,
    generateAddressfromXpub,
    findHDpatfromAddress,
    EthSignRequest,
    DataType,
    ETHSignature,
} from '@keystonehq/bc-ur-registry-eth';

const common = Common.forCustomChain('mainnet', { chainId: this._networkId }, Hardfork.London);
const eip1559Tx = FeeMarketEIP1559Transaction.fromTxData(txParams, { common });
const unsignedBuffer = Buffer.from(eip1559Tx.getMessageToSign(false)); // generate the unsinged transaction bytes
const requestId = uuid.v4();
const addressPath = "M/44'/60'/0'/0/0"

const ethSignRequest = EthSignRequest.constructETHRequest(
    unsignedBuffer,
    DataType.typedTransaction,
    addressPath,
    "12345678", // master fingerprint
    requestId,
    1, // chainId
    "0xfromaddress",
);

const ChunkNumberInEachQR    = 400; // specify the each chunk number in single QR Code

// get the ur decoder
const urDecoder = ethSignRequest.toUREncoder(eachChunkNumberInEachQR);

// render each chunk of data in sign QR Code
while (true) {
    renderQR(urDecoder.nextPart());
}
```

##### EIP-712 TypedData
```ts
import {
    CryptoHDKey,
    generateAddressfromXpub,
    findHDpatfromAddress,
    EthSignRequest,
    DataType,
    ETHSignature,
} from '@keystonehq/bc-ur-registry-eth';

const typeData = "{}" // json string the typed data
const dataHex = Buffer.from(typedData, 'utf-8');
const requestId = uuid.v4();
const addressPath = "M/44'/60'/0'/0/0"
const ethSignRequest = EthSignRequest.constructETHRequest(
    dataHex,
    DataType.typedData,
    addressPath,
    "12345678", // master fingerprint
    requestId,
    undefined,
    "0xfromaddress",
);

const ChunkNumberInEachQR    = 400; // specify the each chunk number in single QR Code

// get the ur decoder
const urDecoder = ethSignRequest.toUREncoder(eachChunkNumberInEachQR);

// render each chunk of data in sign QR Code
while (true) {
    renderQR(urDecoder.nextPart());
}
```

##### Message Signing Request
```ts
import {
    EthSignRequest,
    DataType,
    ETHSignature,
} from '@keystonehq/bc-ur-registry-eth';


const message = "68656c6c6f" // hex string of the message "hello"
const dataHex = Buffer.from(message, 'hex');
const requestId = uuid.v4();
const addressPath = "M/44'/60'/0'/0/0"
const ethSignRequest = EthSignRequest.constructETHRequest(
    dataHex,
    DataType.typedData,
    addressPath,
    "12345678", // master fingerprint
    requestId,
    undefined,
    "0xfromaddress",
);

const ChunkNumberInEachQR    = 400; // specify the each chunk number in single QR Code

// get the ur decoder
const urDecoder = ethSignRequest.toUREncoder(eachChunkNumberInEachQR);

// render each chunk of data in sign QR Code
while (true) {
    renderQR(urDecoder.nextPart());
}
```

### The signature provided by Keystone to the wallet
After Keystone signed the request, it will provide a signature to the wallet which can extract the signature either construct the transaction or verify the signature. we provide a new UR type `eth-signature` for this. For more info about the `eth-signature`, please check the `Specification` section of the [EIP](https://github.com/ethereum/EIPs/pull/4527)

##### Library
We have also provided the `eth-signature` in the `ur-registry-eth` library.

##### Sample
```ts
import {
    URDecoder,
    ETHSignature,
} from '@keystonehq/bc-ur-registry-eth';

const decoder = new URDecoder();

// sinature data from the QR Code
const ur = 'ur:eth-signature/oeadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfptywtosrftahprdctrkbegylogdghjkbafhflamfwlohghtpsseaozorsimnybbtnnbiynlckenbtfmeeamsabnaeoxasjkwswfkekiieckhpecckssptndzelnwfecylbwdlsgvazt';

decoder.receivePart(ur)

if(decoder.isSuccess()) {
    const ur = decoder.resultUR();
    const ethSignature = ETHSignature.fromCBOR(ur.cbor);
    const requestIdBuffer = ethSignature.getRequestId();
    const signature = ethSignature.getSignature(); // it will return the signature r,s,v
    const r = signature.slice(0, 32);
    const s = signature.slice(32, 64);
    const v = signature.slice(64, 65);
} else if(decoder.isError()){
    // logic for error handling
    throw new Error() 
}
```

## Dapp Integration
If you are a Dapp developer and would like to integrate with Keystone directly, we have published several packages to help you.

### [Onboard](https://www.npmjs.com/package/bnc-onboard)
If you have used onboard.js for wallet connection, Keystone has integrated with onboard.js since `1.32.0-0.2.0`. Install the latest version of onboard.js, you can enable Keystone as a wallet option.

### [Keystone-connector](https://www.npmjs.com/package/@keystonehq/keystone-connector)
If you are using web3-react or something like it for the wallet integration,  we have published the `keystone-connector` for help. You can follow the [Readme](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/keystone-connector) for integration
