# Blind Signing on Solana Blockchain

## What is Blind signing

Transaction is the building block of blockchain. No matter whether it is an asset transfer or minting a nft, everything happening on blockchain is based on transaction. Every valid transaction should be signed by users' private keys. In this case, the user should know what the transaction is about and the result of signing the transaction.

For blind signing, user are not aware about what they are signing about. Here is an example about the blind signing.

![](./ledger-solana-tx.png)

Here is the screenshot about how ledger shows the data when doing a swap transaction on [Saber exchange](https://saber.so/). It only shows the transaction hash to make the user confirm which user actually doesn't know what it is about. This is a clear example of blind signing.

## Why blind signing is dangerous

Blind signing can be really dangerous for users. Let's think about these cases. One if the attacker has hacked the user's computer, changed the transaction hash to a different one and tricked the user to sign it. Users' assets can be drained by this malicious transaction.

Second and fishing exchange dapp(e.g. a fake [Saber exchange](https://saber.so/) ) can trick user to sign the malicious transaction when the user performs a swap transaction.

Actually these incidents have happened. Hugh, the founder of Nexus, has been drained of his assets because the attacker hacked his wallet and tricked him to sign the malicious transaction.

So the blind signing can be really dangerous for the user. and This is a really important topic on blockchain security.

## What is the current status of blind signing on Solana blockchain?

Actually the blind signing issue has been discussed a lot in the ethereum community and a lot of improvement has been raised to fix this. Metamask has added an feature called Transaction Insight to decode the transaction data to make user to confirm. Keystone has created the `Smart Contract Metadata Registry` to collect smart contract abi data and use it as the data source to decode the transaction data.

But what is the current status on the Solana blockchain? We have seen that ledger is blind signing for solana blockchain. Unfortunately, most of the solana wallets are actually blind signing.

## How to fix the blind signing on solana blockchain

Since the blind signing can be really dangrouses, I think these fields can be applied to fix the blind signing issue on solana blockchain.

### Decode Transactions

The first thing to fix blind signing to decode the transaction. Currently in most wallets, transactions are shown as raw binary string to the user. But actually we can decode it as a json string which can provide more transparency to users. The solana transaction includes different fields like `accounts`, `instructions`. After decoding, user can have a better idea about what is this transaction about.

Current Transaction data shown to users on Solana.

```
010001032d3c742b98f9ea7738f25a07e7019a6ec8ef0b08f1616b3a3d293e2010aef5bd3a5f18dc95951ad2b1d23f3c5d5aa57b299a9ef9a433be789229a19398e946bc0000000000000000000000000000000000000000000000000000000000000000a756488a76e24b97285c0f334e669b1b7165004f2f520bdad5ecbdbb3641057901020200010c0200000000ca9a3b00000000
```

Instead of showing the raw hex, we propose to show the json string.

```json
{
  "accounts": [
    "43ap6hKfRVzFWzrhmHnsUnY2TWPS81HorucbWte6mW5r",
    "4vrkYqBkdq2joekbn9YNNgosraNv1xnvLvWUt51AHN6P",
    "11111111111111111111111111111111"
  ],
  "block_hash": "CGDPugJoBqVzvmTfjnofbRjKYLhajAfYxb3dCP664ikG",
  "header": {
    "num_readonly_signed_accounts": 0,
    "num_readonly_unsigned_accounts": 1,
    "num_required_signatures": 1
  },
  "instructions": [
    {
      "account_indexes": [0, 1],
      "accounts": "43ap6hKfRVzFWzrhmHnsUnY2TWPS81HorucbWte6mW5r4vrkYqBkdq2joekbn9YNNgosraNv1xnvLvWUt51AHN6P",
      "data": "3Bxs3zzLZLuLQEYX",
      "program_account": "11111111111111111111111111111111",
      "program_index": 2
    }
  ]
}
```

### Use the verification code to confirm the data transmission

On Solana, there is a really great rpc called `simulateTransaction`. With it the wallet can simulate the result of the transaction and show the result to the user to make them have more sense about the transaction's result. But there is one potenital issue here， especially if the user is using the hardware wallet as the signer. How to make sure the data hardware walle received is the same as the simulation ones? If the data is replaced during transmission, users will still have the risk of being hacked.

One possible solution for this is showing an CRC32 verification code and software side and on the hardware side, the hardware will also calculate CRC32 based on the received data and show it on the hardware screen. So if the data have been replaced during transmission. Users can find it out when checking the verification code.

```
CRC32(unsigned transaction data) == CRC32(received data)
```

### Decode the instruction data

On solana, one transaction can contain multiple instructions. Take this unsigned transaction as an example.

```json
{
  "accounts": [
    "ETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVc",
    "7fDG38NmBGVgM4dnZJcv7ZJ1ZsX5Z8QX8ET79A7qfpqr",
    "3Jzkz1f3Hy4uZNAG71HDa9pwivrCX7WLR5PoZJZVyAvs",
    "6aFutFMWR7PbWdBQhdfrcKrAor9WYa2twtSinTMb9tXv",
    "6MKBqwbRQSLisCGgtRjhBmAv7o6KtCWpWk7jTmob26Wn",
    "HXbhpnLTxSDDkTg6deDpsXzJRBf8j7T6Dc3GidwrLWeo",
    "J53kBEY831Mr3RdZtUjEkJeuweKNxTeYNbV9vLYLhp5W",
    "11111111111111111111111111111111",
    "72E8LfHqoxQCxnxmBbDG6WSHnDx1rWPUHNKwYvoL5qDm",
    "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL",
    "Crt7UoUR6QgrFrN7j8rmSQpUTNWNSitSwWvsWGf1qZ5t",
    "EJwZgeZrdC8TXTQbQBoL6bfuAnFUUy1PVCMB4DYPzVaS",
    "SSwpkEEcbUqx4vtoEByFjSkhKdCT862DNVb52nZg1UZ",
    "SysvarC1ock11111111111111111111111111111111",
    "SysvarRent111111111111111111111111111111111",
    "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
    "VeNkoB1HvSP6bSeGybQDnx9wTWFsQb2NBCemeCDSuKL"
  ],
  "block_hash": "MCLBNJ9fS9f6CUzB2JRZG5oquvrfzLkgKMSxT8K9Kn9",
  "header": {
    "num_readonly_signed_accounts": 0,
    "num_readonly_unsigned_accounts": 10,
    "num_required_signatures": 2
  },
  "instructions": [
    {
      "account_indexes": [0, 4, 0, 11, 7, 15, 14],
      "accounts": "ETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVc6MKBqwbRQSLisCGgtRjhBmAv7o6KtCWpWk7jTmob26WnETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVcEJwZgeZrdC8TXTQbQBoL6bfuAnFUUy1PVCMB4DYPzVaS11111111111111111111111111111111TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DASysvarRent111111111111111111111111111111111",
      "data": "",
      "program_account": "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL",
      "program_index": 9
    },
    {
      "account_indexes": [0, 1],
      "accounts": "ETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVc7fDG38NmBGVgM4dnZJcv7ZJ1ZsX5Z8QX8ET79A7qfpqr",
      "data": "1111d4cKUwWT791nL5fDAsbFpB8Z3U45oQ33rHwofEZqfVv9X9FAApjuHP73AKbnJks8N",
      "program_account": "11111111111111111111111111111111",
      "program_index": 7
    },
    {
      "account_indexes": [1, 2, 4, 0],
      "accounts": "7fDG38NmBGVgM4dnZJcv7ZJ1ZsX5Z8QX8ET79A7qfpqr3Jzkz1f3Hy4uZNAG71HDa9pwivrCX7WLR5PoZJZVyAvs6MKBqwbRQSLisCGgtRjhBmAv7o6KtCWpWk7jTmob26WnETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVc",
      "data": "8QL9xsAz2gjrVBFt8QhiPPMarbiNTJic1tfh",
      "program_account": "Crt7UoUR6QgrFrN7j8rmSQpUTNWNSitSwWvsWGf1qZ5t",
      "program_index": 10
    },
    {
      "account_indexes": [1, 15, 12, 0, 16, 8, 13, 2, 3, 4, 5, 6],
      "accounts": "7fDG38NmBGVgM4dnZJcv7ZJ1ZsX5Z8QX8ET79A7qfpqrTokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DASSwpkEEcbUqx4vtoEByFjSkhKdCT862DNVb52nZg1UZETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVcVeNkoB1HvSP6bSeGybQDnx9wTWFsQb2NBCemeCDSuKL72E8LfHqoxQCxnxmBbDG6WSHnDx1rWPUHNKwYvoL5qDmSysvarC1ock111111111111111111111111111111113Jzkz1f3Hy4uZNAG71HDa9pwivrCX7WLR5PoZJZVyAvs6aFutFMWR7PbWdBQhdfrcKrAor9WYa2twtSinTMb9tXv6MKBqwbRQSLisCGgtRjhBmAv7o6KtCWpWk7jTmob26WnHXbhpnLTxSDDkTg6deDpsXzJRBf8j7T6Dc3GidwrLWeoJ53kBEY831Mr3RdZtUjEkJeuweKNxTeYNbV9vLYLhp5W",
      "data": "DzUHxT3nqgc",
      "program_account": "Crt7UoUR6QgrFrN7j8rmSQpUTNWNSitSwWvsWGf1qZ5t",
      "program_index": 10
    },
    {
      "account_indexes": [1, 4, 0, 0],
      "accounts": "7fDG38NmBGVgM4dnZJcv7ZJ1ZsX5Z8QX8ET79A7qfpqr6MKBqwbRQSLisCGgtRjhBmAv7o6KtCWpWk7jTmob26WnETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVcETd398KHD12CV9htTeADMBwNQfXVuYRLmomGUKpgcdVc",
      "data": "XDKtopMv1Pm",
      "program_account": "Crt7UoUR6QgrFrN7j8rmSQpUTNWNSitSwWvsWGf1qZ5t",
      "program_index": 10
    }
  ]
}
```

Each instruction contains one field called data, you can see it in the above example. This is the field indicating which function and params will be executed on this transaction. If the blind signing would be completely resolved, this kind of data should also be decoded. On ethereum, the contract abi can be used to decode the data. But on solana, since all the smart contracts are written by Rust. And this seems no existing solution to decode the instruction data for now.

But fortunately currently there is a project like [`Anchor`](https://project-serum.github.io/anchor/) to help developers to build the program on solana. Anchor has introduced `IDL` to describe the program. For any programs built on `Anchor`, the IDLs have also been submitted to the [`Anchor` project](https://anchor.projectserum.com/). So these IDLs can be used just like ABI to decode the instruction data. But to achieve this, it would need some changes on the Anchor side to support this.

So for the latest step, the wallet provider can work with the Anchor team to make the instruction data be fully decoded by using the IDL.

These are three ways we propose to completely fix the blind signing issue on the Solana blockchain. And we are currently adding the Solana support on our Keystone wallet, we will continually introduce these ways on our product.
