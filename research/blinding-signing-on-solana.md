# Blind Signing on Solana Blockchain

## What is Blind signing
Transaction is the building block of blockchain. No matter whether is an asset transfer or minting a nft, everything happened on blockchain is based on transaction. Every vaild transaction should be signed by users' private key. In this case, user should know what is the transaction about and the result of signing the transaction. 

For blind signing, user have no aware about what they are signing about. Here is an example about the blind signing.

Here is the screenshot about how ledger show the data when doing an swap transction on saber exchange. It only show the transaction hash to make user to confirm which user actually doesn't know what it is about. This is an clear example of blind signing.

## Why blind signing is dangerous
Blind signing can be really dangerous for user. Let's think about these cases. One if the attacker has hacked user's computer, changed the transaction hash to an differnt one and trick user to sign it. Users' asset can be drained by this malicious transaction. 

Second an fishing exchange dapp(e.g. an fake saber exchange ) can trick user to sign the malicious transaction when user an performing an swap transaction. 


Actually these incident have happended. Hugh, the founder of Nexus, have been drained his assets beacause the attacker hacked his wallet and trick him to sign the malicious transaction.

So the blind signing can be really dangerous for the user. and This is an really important topic on the blockchain security.


## What is current status about blind signing on Solana blockchain.
Actually the blind signing issue have been discussed a lot on the ethereum commnuty and there are a lot inproment has been raised to fix this. Metamask has added an feature called Transaction Insight to decode the transaction data to make user to confirm. Keystone has created the `Smart Contract Metadata Registry` to collect smart contract abi data and use it as the data source to decode the transaction data. 

But what is the current status on the Solana blockchain. We have seen ledger is blind signing for solana blockchain. Unfortunately, most of the solana wallet are actualy blind signing. 

## How to fix the blind signing on solana blockchain
Since the blind signing can be really dangrouses, I think there are these fields can be really applied to fix the blind signing issue.

### Decode Transactions

The first thing to fix blind signing to decode the transaction. Currently in most wallet, transaction is showed as raw binary string to user. But actually we can decode it as json string which can provide more transparency to user. The solana transaction includes differnt fiels like `accounts`, `instructions`. After decoding, user can have a better idea about what is this transaction about.

### Use the verfication code to confirm the data transmission

On solana, there is really greate rpc called `simulateTransaction`. With it the wallet can simulate the result of the transaction and show the result to user to make them have more sense what the transaction is about. But there is one potenital issue here， specially if the user are using the hardware wallet as the signer. How to make sure the data hardware walle received is the same as the simulation ones? if the data is be replaced during transimation, user will still have the risk to be hacked.

One possible solution for this is showing an CRC32 verfication code and software side and on the hardware side, the hardware will also caculate CRC32 based on the received data and show it on the hardware screen. So if the data have been replaced during transmission. User can find it out when checking the verfication code. 

### Decode the instrucation data

