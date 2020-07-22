# 0.9.1-invalid-utxo
The repo help to verify the invalid UTXO after the qitmeer 0.9.1-release-hotfix update.

## Introduction 

When we debug a [strange `qitmeer-walet` problem](https://github.com/Qitmeer/qitmeer-wallet/issues/54) reported by the community, a severe hidden bug since 0.9.0 network had found. The bug results in inconsistent tx fee calculation in a particular case. The transaction fee rule in `Qitmeer` has changed since version `0.9.0`. If two blocks contain the same tx, only the prior order block takes the transaction fee. The feature has introduced into the qitmeer network since `0.9.0` genesis. The bug hides with the new feature. 

The first incorrect trasaction fee in 0.9.0 network is block `00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea` (block order `13567`)  

Transaction fee **`515000`** has been calculated incorrectly as **`540600`**. 

The wrong transaction fee has already fixed by the [Qitmeer-0.9.1-hotfix-release](https://github.com/Qitmeer/qitmeer/releases/tag/v0.9.1-release-hotfix). The Qitmeer [PR-#338](https://github.com/Qitmeer/qitmeer/pull/338) contains the bug fix code. 

You can verify it by using the following RPC command :
```
curl -s -k -u test:test -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"2.0","method":"getBlockV2","params":["0x00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea",true,true,true],"id":1}' https://127.0.0.1:18131|jq .result.transactionfee
515000
```
Or from the qitmeer block explorer https://explorer.qitmeer.io/block/00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea 

But there is some side-effect of the fix. After applying the fix, some historical transaction has to change its valid/invalid status. It means some address' **balance might change** accordingly because some transactions are invalid indeed, and on the other side, some transactions wrongly set to invalid need to change to valid.

The repo's purpose is to explain why some balances are changed and why some transactions marked invalid. Help user to verify their balances. Please feel free to open an [issue](https://github.com/Qitmeer/0.9.1-invalid-utxo/issues) to the repo if you experience a wrong balance or you have any doubt about an invalid transaction.

Sorry for your inconvenience, and we appreciated you are interested in the Qitmeer 0.9 Testnet.

-- Qitmeer Dev Team

## Note

All balance of address has listed on this repo based on a fixed order `76479`. (You know the balance always changed when block accumulated)  

```
Order : 76479  
Hash  : fdaa0d1bd1dd5fcc166de040780c140402bd5b90bbc04ffb9ffba44923446a8c
Date  : 2020-07-20 10:49:31
```
