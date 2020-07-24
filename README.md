# 0.9.1-invalid-utxo
The repo help to verify the invalid UTXO after the qitmeer 0.9.1-release-hotfix update.

## Introduction 

When we debug a [strange `qitmeer-walet` problem](https://github.com/Qitmeer/qitmeer-wallet/issues/54) reported by the community, a severe hidden bug since 0.9.0 network had found. The bug results in inconsistent tx fee calculation in a particular case. The transaction fee rule in `Qitmeer` has changed since version `0.9.0`. If two blocks contain the same tx, only the prior order block takes the transaction fee. The feature has introduced into the qitmeer network since `0.9.0` genesis. The bug hides with the new feature. 

The first incorrect trasaction fee in 0.9.0 network is block [`00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea`][bk_00005b] (block order `13567`)  
Transaction fee **`515000`** has been calculated incorrectly as [**`540600`**][wrong_bk_00005b]. 

The reason is the UTXO in the tx [`4067bb146ebc36a6cc244a267c59245c398132c0511a8ccaed90086d764ae6ab`][tx_4067bb] in the block [`f666bcfab2fe32400bd537ea05d5b9855ea13eb3c4fd11b3e786896e1eb3bcdd`][bK_f666bc] (Order 9739) is **12000025600** , but it has been caculated incorrectly as **12000051200** due to a bug in transaction fee caculation.

The wrong transaction fee : 
```
$ echo 67*12000000000+12000051200-815999510600|bc
540600
```
The correct transaction fee :
```
$ echo 67*12000000000+12000025600-815999510600|bc
515000
```

The inconsistent transaction fee bug has already fixed by the [Qitmeer-0.9.1-hotfix-release][v0.9.1-hotfix]. The Qitmeer [PR-#338][PR_338] contains the bug fix code. 

Since the Block [13567][bk_00005b], the STXO cache works in a incorrect status. That's the reason why you need to clean up your local database and redo the block synchronization. You need to upgrade your node to the latest version [0.9.1-hotfix-release][v0.9.1-hotfix] and clean up your local UTXO/STXO status database.

You have to clean up your local database and redo the block synchronization from scratch by using the following command before starting your node.
```
# first, make sure you are using the correct version.
./qitmeer --version
qitmeer version 0.9.1+hotfix-release-6e0a73a (Go version go1.14.4)

# then clean up your local database before you start your node.
./qitmeer --testnet --cleanup
```

You can verify if your node is working correctly by using the following RPC command :
```
curl -s -k -u test:test -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"2.0","method":"getBlockV2","params":["0x00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea",true,true,true],"id":1}' https://127.0.0.1:18131|jq .result.transactionfee
515000
```

But there is some side-effect of the fix. After applying the fix, some historical transaction has to change its valid/invalid status because since block [13567][bk_00005b], the UTXO/STXO database not in the correct state in the old version. It means if your transaction tries to spend an incorrect UTXO, the operation is invalid. It implies some address' **balance might change** accordingly because some transactions are invalid indeed, and on the other side, some transactions wrongly set to invalid need to change to valid.

The repo's purpose is to explain why some balances are changed and why some transactions marked invalid. Help user to verify their balances. Please feel free to open an [issue](https://github.com/Qitmeer/0.9.1-invalid-utxo/issues) to the repo if you experience a wrong balance or you have any doubt about an invalid transaction.

Sorry for your inconvenience, and we appreciated you are interested in the Qitmeer 0.9 Testnet.

-- Qitmeer Dev Team


[tx_4067bb]:https://explorer.qitmeer.io/tx/4067bb146ebc36a6cc244a267c59245c398132c0511a8ccaed90086d764ae6ab
[bk_f666bc]:https://explorer.qitmeer.io/block/f666bcfab2fe32400bd537ea05d5b9855ea13eb3c4fd11b3e786896e1eb3bcdd
[bk_00005b]:https://explorer.qitmeer.io/block/00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea
[wrong_bk_00005b]:http://091.meerscan.io/block/00005bb24703377e61113d414258d26af307f85dccf425f85a14ea3c14e58fea
[v0.9.1-hotfix]:https://github.com/Qitmeer/qitmeer/releases/tag/v0.9.1-release-hotfix
[PR_338]:https://github.com/Qitmeer/qitmeer/pull/338

## Note

All **balance** of address has listed/discussed on this repo based on a fixed order `76479`. (You know the **balance** always changed when block accumulated)  

```
Order : 76479  
Hash  : fdaa0d1bd1dd5fcc166de040780c140402bd5b90bbc04ffb9ffba44923446a8c
Date  : 2020-07-20 10:49:31
```
