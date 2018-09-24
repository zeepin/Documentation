<h1 align="center">JAVA_SDK_EN</h1>
<h4 align="center">Version 1.0 </h4>

[English](/English/JAVA_SDK_EN.md) | [中文](/Chinese/JAVA_SDK_CN.md)

## Outline:
  
  
* [Managing Account](#managing-account)
    * [Convert old address to new address](#convert-old-address-to-new-address)
    * [Managing using a Private Key (without a wallet)](#managing-using-a-private-key-without-a-wallet)
        * [Creating Accounts Randomly](#creating-accounts-randomly)
        * [Creating Accounts based on Private Keys](#creating-accounts-based-on-private-keys)
    * [Managing with a Wallet](#managing-with-a-wallet)
* [Generating Addresses](#generating-addresses)
* [Transferring ZPT and Gala](#transferring-zpt-and-gala)
    * [1. Initialization](#1-initialization)
    * [2. Information Query](#2-information-query)
        * [Query Zeepin\Gala Balance](#query-zeepingala-balance)
        * [Query whether the transaction is still in the transaction pool](#query-whether-the-transaction-is-still-in-the-transaction-pool)
        * [Query whether the transaction has been successful](#query-whether-the-transaction-has-been-successful)
        * [The List of Chain Interaction Interfaces](#the-list-of-chain-interaction-interfaces)
    * [3. Making Zeepin Transactions](#3-making-zeepin-transactions)
        * [Creating Transfers and Sending them ](#creating-transfers-and-sending-them)
        * [Multiple Signatures](#multiple-signatures)
        * [From One to Multiple or from Multiple to One](#from-one-to-multiple-or-from-multiple-to-one)
    * [4. Gala Transfers](#4-gala-transfers)
        * [The Gala transfer interface is similar to ZPT](#the-gala-transfer-interface-is-similar-to-zpt)
        * [Withdrawing Gala](#withdrawing-gala)
  
  

### Managing Account

#### Convert old address to new address：
```java
String oldAddress = "AVBokCnLS9NxUjeXTdL8BMsgN41bwtK89J";//old address starts with "A"
String newAddress = ConvertAddress.convertAddress(oldAddress);//new address starts with "Z"
```

#### Managing using a Private Key (without a wallet):

##### Creating Accounts Randomly:

```java
com.github.zeepin.account.Account acct = new com.github.zeepin.account.Account(zeepinSdk.defaultSignScheme);
acct.serializePrivateKey();//Private Key
acct.serializePublicKey();//Public Key
acct.getAddressU160().toBase58();//base58 address
```


##### Creating Accounts based on Private Keys

```java
com.github.zeepin.account.Account acct0 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0), zeepinSdk.defaultSignScheme);
com.github.zeepin.account.Account acct1 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey1), zeepinSdk.defaultSignScheme);
com.github.zeepin.account.Account acct2 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey2), zeepinSdk.defaultSignScheme);

```

#### Managing with a Wallet:


```java

#### Generating a Batch of Accounts in Your Wallet:
zeepinSdk.getWalletMgr().createAccounts(10, "password");
zeepinSdk.getWalletMgr().writeWallet();

Creating Accounts Randomly:
AccountInfo info0 = zeepinSdk.getWalletMgr().createAccountInfo("password");

Creating Accounts using Private Keys:
AccountInfo info = zeepinSdk.getWalletMgr().createAccountInfoFromPriKey("password","00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cf");

Getting Accounts
com.github.zeepin.account.Account acct0 = zeepinSdk.getWalletMgr().getAccount(info.addressBase58,"password");

```



### Generating Addresses

```
Generating Single-Signature Addresses:
String privatekey0 = "privatekey";
String privatekey1 = "privatekey";
String privatekey2 = "privatekey";

//accounts creating, addresses generating
com.github.zeepin.account.Account acct0 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0), zeepinSdk.defaultSignScheme);
Address sender = acct0.getAddressU160();

//base58 address decoding
sender = Address.decodeBase58("ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp")；

Generating Multisig Addresses:
Address recvAddr = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());


```

| Approach                | Parameters                | Parameter Description          |
| :---------------------- | :------------------------ | :----------------------------- |
| addressFromMultiPubkeys | int m,byte\[\]... pubkeys | The minimum number of verifying signatures (<=the number of public keys)，public key |



### Transferring ZPT and Gala

**Before transferring on the MainNet, please set gaslimit to 20000, gasprice to 1.**


#### 1. Initialization

```
String ip = "http://test1.zeepin.net";
String rpcUrl = ip + ":" + "20336";
ZPTSdk zeepinSdk = ZPTSdk.getInstance();
zeepinSdk.setRpc(rpcUrl);
zeepinSdk.setDefaultConnect(zptSdk.getRpc());

```

#### 2. Information Query

##### Query Zeepin\Gala Balance

```
zeepinSdk.getConnect().getBalance("ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp");

Zeepin Information Query:
System.out.println(zeepinSdk.nativevm().zpt().queryName());
System.out.println(zeepinSdk.nativevm().zpt().querySymbol());
System.out.println(zeepinSdk.nativevm().zpt().queryDecimals());
System.out.println(zeepinSdk.nativevm().zpt().queryTotalSupply());

Gala information Query:
System.out.println(zeepinSdk.nativevm().gala().queryName());
System.out.println(zeepinSdk.nativevm().gala().querySymbol());
System.out.println(zeepinSdk.nativevm().gala().queryDecimals());
System.out.println(zeepinSdk.nativevm().gala().queryTotalSupply());



```

##### Query whether the transaction is still in the transaction pool

```
zeepinSdk.getConnect().getMemPoolTxState("00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4")


response when the transactions exists in the transaction pool:

{
    "Action": "getmempooltxstate",
    "Desc": "SUCCESS",
    "Error": 0,
    "Result": {
        "State":[
            {
              "Type":1,
              "Height":451,
              "ErrCode":0
            },
            {
              "Type":0,
              "Height":0,
              "ErrCode":0
            }
       ]
    },
    "Version": "0.1.0"
}

Or the transaction does not exist in the transaction pool

{
    "Action": "getmempooltxstate",
    "Desc": "UNKNOWN TRANSACTION",
    "Error": 44001,
    "Result": "",
    "Version": "0.1.0"
}

```

##### Query whether the transaction has been successful

Query Push Notifications from a Smart Contract

```
zeepinSdk.getConnect().getSmartCodeEvent("00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4")


response:
{
    "Action": "getsmartcodeeventbyhash",
    "Desc": "SUCCESS",
    "Error": 0,
    "Result": {
        "TxHash": "00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4",
        "State": 1,
        "GasConsumed": 0,
        "Notify": [
            {
                "CzeepinractAddress": "0100000000000000000000000000000000000000",
                "States": [
                    "transfer",
                    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
                    "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
                    300000
                ]
            }
        ]
    },
    "Version": "0.1.0"
}

```

Query Smart Contract Events based on the Block Height. Return Transactions with Events

```
zeepinSdk.getConnect().getSmartCodeEvent(10)

response:
{
    "Action": "getsmartcodeeventbyhash",
    "Desc": "SUCCESS",
    "Error": 0,
    "Result": {
        "TxHash": "00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4",
        "State": 1,
        "GasConsumed": 0,
        "Notify": [
            {
                "CzeepinractAddress": "0100000000000000000000000000000000000000",
                "States": [
                    "transfer",
                    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
                    "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
                    300000
                ]
            }
        ]
    },
    "Version": "0.1.0"
}

```

#### Status Reading via “State”:
- 1 stands for successful transaction
- 0 stands for failed transaction


#### Parsing Arrays via Notify:

##### ContractAddress：
- 0100000000000000000000000000000000000000 for ZPT				        
- 0200000000000000000000000000000000000000 for Gala

​     States：Array

	transfer	 stands for the transaction
	from		 sender address
	to		 recipient address
	The forth line is the transfer amount (Both ZPT and Gala have 4 decimal places, and the actual amount of ZPT and Gala should be divided by 10000)
	
Filtering the recipient address through the recharge address that the exchange generates for the user, one could access the user recharge record.




##### The List of Chain Interaction Interfaces:

| No   |                    Main   Function                     |     Description      |
| ---- | :----------------------------------------------------: | :------------------: |
| 1    |       zeepinSdk.getConnect().getGenerateBlockTime()       |   Query When VBFT Generated the Block   |
| 2    |           zeepinSdk.getConnect().getNodeCount()           |     Query Number of Nodes     |
| 3    |            zeepinSdk.getConnect().getBlock(15)            |        Query Block Info        |
| 4    |          zeepinSdk.getConnect().getBlockJson(15)          |        Query Block Info        |
| 5    |       zeepinSdk.getConnect().getBlockJson("txhash")       |        Query Block Info        |
| 6    |         zeepinSdk.getConnect().getBlock("txhash")         |        Query Block Info        |
| 7    |          zeepinSdk.getConnect().getBlockHeight()          |     Query Current Block Height     |
| 8    |      zeepinSdk.getConnect().getTransaction("txhash")      |       Query Transaction Info       |
| 9    | zeepinSdk.getConnect().getStorage("zeepinConractAddress", key) |   Query Smart Contract Storage   |
| 10   |       zeepinSdk.getConnect().getBalance("address")        |       Query Balance Info       |
| 11   | zeepinSdk.getConnect().getContractJson("czeepinractaddress") |     Query Smart Contract Info     |
| 12   |       zeepinSdk.getConnect().getSmartCodeEvent(59)        |   Query Smart Contract Events   |
| 13   |    zeepinSdk.getConnect().getSmartCodeEvent("txhash")     |   Query Smart Contract Events   |
| 14   |  zeepinSdk.getConnect().getBlockHeightByTxHash("txhash")  |   Query Block Height by Transaction Hash   |
| 15   |      zeepinSdk.getConnect().getMerkleProof("txhash")      |    Obtain a Merkle Proof    |
| 16   | zeepinSdk.getConnect().sendRawTransaction("txhexString")  |       Send the Transaction       |
| 17   |  zeepinSdk.getConnect().sendRawTransaction(Transaction)   |       Send the Transaction       |
| 18   |    zeepinSdk.getConnect().sendRawTransactionPreExec()     |    Send the pre-execution transaction    |
| 19   |  zeepinSdk.getConnect().getAllowance("zeepin","from","to")   |    Query Available Balance    |
| 20   |        zeepinSdk.getConnect().getMemPoolTxCount()         | Query the Total Transaction Volume in the Transaction Pool |
| 21   |        zeepinSdk.getConnect().getMemPoolTxState()         | Query Transaction Status in the Transaction Pool |

#### 3. Making Zeepin Transactions

##### Creating Transfers and Sending them

```
Sender and Recipient Addresses:
Address sender = acct0.getAddressU160();
Address recvAddr = acct1;
//Generating Multisig Addresses
//Address recvAddr = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());

Creating Transfers:
long amount = 1000;
Transaction tx = zeepinSdk.nativevm().zpt().makeTransfer(sender.toBase58(),recvAddr.toBase58(), amount,sender.toBase58(),30000,0);


Generating Signatures for Transactions:
zeepinSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct0}});
//How to sign with multisig addresses
zeepinSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct1, acct2}});
//When the sender and the network fee payer do not share the same address, the signature from the network fee payer is required


Sending Transactions:
zeepinSdk.getConnect().sendRawTransaction(tx.toHexString());


```



| Approach       | Parameters                                                     | Parameter Descriptions                                                  |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| makeTransfer | String sender，String recvAddr,long amount,String payer,long gaslimit,long gasprice | Sender Address, Recipient Address, Network Fee Payer Address, Amount, Gaslimit, Gasprice |
| makeTransfer | State\[\] states,String payer,long gaslimit,long gasprice    | A transaction consists of multiple transfers                                 |

##### Multiple Signatures

When the sender and the network fee payer do not share the same address, the signature from the network fee payer is required

```
1. Adding Single Signatures
zeepinSdk.addSign(tx,acct0);

2. Adding Multiple Signatures
zeepinSdk.addMultiSign(tx,2,new com.github.zeepin.account.Account[]{acct0,acct1});

```


##### From One to Multiple or from Multiple to One

1. Creating shared-state transactions
2. Signature
3. A transaction constitutes of up to 1024 transfers

```
Address sender1 = acct0.getAddressU160();
Address sender2 = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());
int amount = 10;
int amount2 = 20;

State state = new State(sender1, recvAddr, amount);
State state2 = new State(sender2, recvAddr, amount2);
Transaction tx = zeepinSdk.nativevm().zpt().makeTransfer(new State[]{state1,state2},sender1.toBase58(),30000,0);

//The first sender is a single-signature address, while the second is a multisig address:
zeepinSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct0}});
zeepinSdk.addMultiSign(tx,2,new com.github.zeepin.account.Account[]{acct1, acct2});

```




#### 4. Gala Transfers

##### The Gala transfer interface is similar to ZPT:

```
zeepinSdk.nativevm().gala().makeTransfer...
```

##### Withdrawing Gala

- Query whether there are withdrawable Gala
- Create a Transaction and a Signature
- Send the Transaction of Gala Withdraw

```
Query Gala that has not been withdrawn:
String addr = acct0.getAddressU160().toBase58();
String gala = zeepinSdk.nativevm().gala().unboundGala(addr);

//Withdraw Gala
zeepinSdk.signatureScheme);
String hash = sdk.nativevm().gala().withdrawGala(account,toAddr,64000L,payerAcct,20000,1);

```

| Approach    | Parameter                                                     | Parameter Description                                                |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| makeClaimGala | String claimer,String to,long amount,String payer,long gaslimit,long gasprice | Claim Withdrawer, Claim Recipient, Amount, Address of Network Fee Payer, gaslimit, gasprice |


