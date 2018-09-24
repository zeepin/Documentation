<h1 align="center">JAVA_SDK_CN</h1>
<h4 align="center">Version 1.0 </h4>


## Java SDK的使用

**本文大纲：**  
  
  
* [账号管理](#账号管理)
    * [旧地址转换为新地址](#旧地址转换为新地址)
    * [纯私钥管理（无钱包模式）](#纯私钥管理无钱包模式)
        * [随机创建账号](#随机创建账号)
        * [根据私钥创建账号](#根据私钥创建账号)
    * [使用钱包管理](#使用钱包管理)
* [创建地址](#创建地址)
* [ZPT和Gala转账](#zpt和gala转账)
    * [1. 初始化](#1-初始化)
    * [2. 查询](#2-查询)
        * [查询zeepin\Gala余额](#查询zeepingala余额)
        * [查询交易是否在交易池中](#查询交易是否在交易池中)
        * [查询交易是否调用成功](#查询交易是否调用成功)
        * [其他与链交互接口列表](#其他与链交互接口列表)
    * [3. zeepin转账](#3-zeepin转账)
        * [构造转账交易并发送](#构造转账交易并发送)
        * [多次签名](#多次签名)
        * [一转多或多转多](#一转多或多转多)
    * [4. Gala转账](#4-gala转账)
        * [Gala转账接口与ZPT类似](#gala转账接口与zpt类似)
        * [提取Gala](#提取gala)
  
  

### 账号管理

#### 旧地址转换为新地址：
```java
String oldAddress = "AVBokCnLS9NxUjeXTdL8BMsgN41bwtK89J";//A开头的旧地址
String newAddress = ConvertAddress.convertAddress(oldAddress);//Z开头的新地址
```

#### 纯私钥管理（无钱包模式）：

##### 随机创建账号：

```java
com.github.zeepin.account.Account acct = new com.github.zeepin.account.Account(zeepinSdk.defaultSignScheme);
acct.serializePrivateKey();//私钥
acct.serializePublicKey();//公钥
acct.getAddressU160().toBase58();//base58地址
```


##### 根据私钥创建账号

```java
com.github.zeepin.account.Account acct0 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0), zeepinSdk.defaultSignScheme);
com.github.zeepin.account.Account acct1 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey1), zeepinSdk.defaultSignScheme);
com.github.zeepin.account.Account acct2 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey2), zeepinSdk.defaultSignScheme);

```

#### 使用钱包管理：


```java

#### 在钱包中批量创建账号:
zeepinSdk.getWalletMgr().createAccounts(10, "password");
zeepinSdk.getWalletMgr().writeWallet();

随机创建:
AccountInfo info0 = zeepinSdk.getWalletMgr().createAccountInfo("password");

通过私钥创建:
AccountInfo info = zeepinSdk.getWalletMgr().createAccountInfoFromPriKey("password","00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cf");

获取账号
com.github.zeepin.account.Account acct0 = zeepinSdk.getWalletMgr().getAccount(info.addressBase58,"password");

```



### 创建地址

```
单签地址生成：
String privatekey0 = "privatekey";
String privatekey1 = "privatekey";
String privatekey2 = "privatekey";

//生成账号，获取地址
com.github.zeepin.account.Account acct0 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0), zeepinSdk.defaultSignScheme);
Address sender = acct0.getAddressU160();

//base58地址解码
sender = Address.decodeBase58("ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp")；

多签地址生成：
Address recvAddr = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());


```

| 方法名                  | 参数                      | 参数描述                       |
| :---------------------- | :------------------------ | :----------------------------- |
| addressFromMultiPubkeys | int m,byte\[\]... pubkeys | 最小验签个数(<=公钥个数)，公钥 |



### ZPT和Gala转账

**对于在主网转账，请将gaslimit 设为20000，gasprice设为1


#### 1. 初始化

```
String ip = "http://test1.zeepin.net";
String rpcUrl = ip + ":" + "20336";
ZPTSdk zeepinSdk = ZPTSdk.getInstance();
zeepinSdk.setRpc(rpcUrl);
zeepinSdk.setDefaultConnect(zptSdk.getRpc());

```

#### 2. 查询

##### 查询zeepin\Gala余额

```
zeepinSdk.getConnect().getBalance("ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp");

查ZPT信息：
System.out.println(zeepinSdk.nativevm().zpt().queryName());
System.out.println(zeepinSdk.nativevm().zpt().querySymbol());
System.out.println(zeepinSdk.nativevm().zpt().queryDecimals());
System.out.println(zeepinSdk.nativevm().zpt().queryTotalSupply());

查Gala信息：
System.out.println(zeepinSdk.nativevm().gala().queryName());
System.out.println(zeepinSdk.nativevm().gala().querySymbol());
System.out.println(zeepinSdk.nativevm().gala().queryDecimals());
System.out.println(zeepinSdk.nativevm().gala().queryTotalSupply());



```

##### 查询交易是否在交易池中

```
zeepinSdk.getConnect().getMemPoolTxState("00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4")


response 交易池存在此交易:

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

或 交易池不存在此交易

{
    "Action": "getmempooltxstate",
    "Desc": "UNKNOWN TRANSACTION",
    "Error": 44001,
    "Result": "",
    "Version": "0.1.0"
}

```

##### 查询交易是否调用成功

查询智能合约推送内容

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

根据块高查询智能合约事件，返回有事件的交易

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

#### 通过查看“State” 判断：
- 1 代表交易成功
- 0 代表交易失败


#### “Notify"解析数组如下：

##### ContractAddress：合约地址
- 0100000000000000000000000000000000000000 为ZPT					        
- 0200000000000000000000000000000000000000 为Gala

​     States：数组

	transfer	代表转账操作
	from		转出地址
	to		目标地址
	第四行为转账数量（ZPT和Gala的精度为4，所以这里ZPT和Gala的实际数量因除以10000）
	
过滤 to 地址为交易所为用户生成的充值地址，即可取得用户的充值记录




##### 其他与链交互接口列表：

| No   |                    Main   Function                     |     Description      |
| ---- | :----------------------------------------------------: | :------------------: |
| 1    |       zeepinSdk.getConnect().getGenerateBlockTime()       |   查询VBFT出块时间   |
| 2    |           zeepinSdk.getConnect().getNodeCount()           |     查询节点数量     |
| 3    |            zeepinSdk.getConnect().getBlock(15)            |        查询块        |
| 4    |          zeepinSdk.getConnect().getBlockJson(15)          |        查询块        |
| 5    |       zeepinSdk.getConnect().getBlockJson("txhash")       |        查询块        |
| 6    |         zeepinSdk.getConnect().getBlock("txhash")         |        查询块        |
| 7    |          zeepinSdk.getConnect().getBlockHeight()          |     查询当前块高     |
| 8    |      zeepinSdk.getConnect().getTransaction("txhash")      |       查询交易       |
| 9    | zeepinSdk.getConnect().getStorage("czeepinractaddress", key) |   查询智能合约存储   |
| 10   |       zeepinSdk.getConnect().getBalance("address")        |       查询余额       |
| 11   | zeepinSdk.getConnect().getCzeepinractJson("czeepinractaddress") |     查询智能合约     |
| 12   |       zeepinSdk.getConnect().getSmartCodeEvent(59)        |   查询智能合约事件   |
| 13   |    zeepinSdk.getConnect().getSmartCodeEvent("txhash")     |   查询智能合约事件   |
| 14   |  zeepinSdk.getConnect().getBlockHeightByTxHash("txhash")  |   查询交易所在高度   |
| 15   |      zeepinSdk.getConnect().getMerkleProof("txhash")      |    获取merkle证明    |
| 16   | zeepinSdk.getConnect().sendRawTransaction("txhexString")  |       发送交易       |
| 17   |  zeepinSdk.getConnect().sendRawTransaction(Transaction)   |       发送交易       |
| 18   |    zeepinSdk.getConnect().sendRawTransactionPreExec()     |    发送预执行交易    |
| 19   |  zeepinSdk.getConnect().getAllowance("zeepin","from","to")   |    查询允许使用值    |
| 20   |        zeepinSdk.getConnect().getMemPoolTxCount()         | 查询交易池中交易总量 |
| 21   |        zeepinSdk.getConnect().getMemPoolTxState()         | 查询交易池中交易状态 |

#### 3. zeepin转账

##### 构造转账交易并发送

```
转出方与收款方地址：
Address sender = acct0.getAddressU160();
Address recvAddr = acct1;
//多签地址生成
//Address recvAddr = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());

构造转账交易：
long amount = 1000;
Transaction tx = zeepinSdk.nativevm().zpt().makeTransfer(sender.toBase58(),recvAddr.toBase58(), amount,sender.toBase58(),30000,0);


对交易做签名：
zeepinSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct0}});
//多签地址的签名方法：
zeepinSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct1, acct2}});
//如果转出方与网络费付款人不是同一个地址，需要添加网络费付款人的签名


发送交易：
zeepinSdk.getConnect().sendRawTransaction(tx.toHexString());


```



| 方法名       | 参数                                                         | 参数描述                                                     |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| makeTransfer | String sender，String recvAddr,long amount,String payer,long gaslimit,long gasprice | 发送方地址，接收方地址，金额，网络费付款人地址，gaslimit，gasprice |
| makeTransfer | State\[\] states,String payer,long gaslimit,long gasprice    | 一笔交易包含多个转账。                                       |

##### 多次签名

如果转出方与网络费付款人不是同一个地址，需要添加网络费付款人的签名

```
1.添加单签签名
zeepinSdk.addSign(tx,acct0);

2.添加多签签名
zeepinSdk.addMultiSign(tx,2,new com.github.zeepin.account.Account[]{acct0,acct1});

```


##### 一转多或多转多

1. 构造多个state的交易
2. 签名
3. 一笔交易上限为1024笔转账

```
Address sender1 = acct0.getAddressU160();
Address sender2 = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());
int amount = 10;
int amount2 = 20;

State state = new State(sender1, recvAddr, amount);
State state2 = new State(sender2, recvAddr, amount2);
Transaction tx = zeepinSdk.nativevm().zpt().makeTransfer(new State[]{state1,state2},sender1.toBase58(),30000,0);

//第一个转出方是单签地址，第二个转出方是多签地址：
zeepinSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct0}});
zeepinSdk.addMultiSign(tx,2,new com.github.zeepin.account.Account[]{acct1, acct2});

```




#### 4. Gala转账

##### Gala转账接口与ZPT类似：

```
zeepinSdk.nativevm().gala().makeTransfer...
```

##### 提取Gala

- 查询是否有Gala可以提取
- 构造交易和签名
- 发送提取Gala交易

```
查询未提取Gala:
String addr = acct0.getAddressU160().toBase58();
String gala = zeepinSdk.nativevm().gala().unboundGala(addr);

//提取Gala
zeepinSdk.signatureScheme);
String hash = sdk.nativevm().gala().withdrawGala(account,toAddr,64000L,payerAcct,20000,1);

```

| 方法名       | 参数                                                         | 参数描述                                                     |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| makeClaimGala | String claimer,String to,long amount,String payer,long gaslimit,long gasprice | claim提取者，提给谁，金额，网络付费人地址，gaslimit，gasprice |


