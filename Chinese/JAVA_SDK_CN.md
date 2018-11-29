<h1 align="center">JAVA_SDK_CN</h1>
<h4 align="center">Version 1.0 </h4>


[English](/English/JAVA_SDK_EN.md) | [中文](/Chinese/JAVA_SDK_CN.md)

## 目录
  
  
* [账号管理](#账号管理)
    * [旧地址转换为新地址](#旧地址转换为新地址)
    * [纯私钥管理（无钱包模式）](#纯私钥管理无钱包模式)
        * [随机创建账号](#随机创建账号)
        * [根据私钥创建账号](#根据私钥创建账号)
    * [使用钱包管理](#使用钱包管理)
    * [创建地址](#创建地址)
* [数字身份管理](#数字身份管理)
    * [1. 注册身份](#1-注册身份)
    	* [创建数字身份](#创建数字身份)
    	* [向链上注册身份](#向链上注册身份)
    * [2. 身份管理](#2-身份管理)
        * [移除身份](#移除身份)
        * [设置默认身份](#设置默认身份)
    * [3. 查询链上身份信息](#3-查询链上身份信息)
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
        * [使用签名机签名](#使用签名机签名)
    * [4. Gala转账](#4-gala转账)
        * [Gala转账接口与ZPT类似](#gala转账接口与zpt类似)
        * [提取Gala](#提取gala)  
    * [5. 原生数字资产接口](#5-原生数字资产接口)
    
    
  
  

### 账号管理
账户是基于公私钥创建的，地址是公钥转换而来。

#### 旧地址转换为新地址：
```java
String oldAddress = "AVBokCnLS9NxUjeXTdL8BMsgN41bwtK89J";//A开头的旧地址
String newAddress = ConvertAddress.convertAddress(oldAddress);//Z开头的新地址
```

#### 纯私钥管理（无钱包模式）：
是指自己存储，账户信息保存在用户数据库或其他地方，而不存储在遵循钱包规范的文件中。
##### 随机创建账号：

```java
com.github.zeepin.account.Account acct = new com.github.zeepin.account.Account(zeepinSdk.defaultSignScheme);
acct.serializePrivateKey();//私钥
acct.serializePublicKey();//公钥
acct.getAddressU160().toBase58();//base58地址
zptSdk.getWalletMgr().writeWallet();//写入钱包
```


##### 根据私钥创建账号

```java
com.github.zeepin.account.Account acct0 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0), zptSdk.defaultSignScheme);
com.github.zeepin.account.Account acct1 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey1), zptSdk.defaultSignScheme);
com.github.zeepin.account.Account acct2 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey2), zptSdk.defaultSignScheme);

```

#### 使用钱包管理：


```java

#### 在钱包中批量创建账号:
zptSdk.getWalletMgr().createAccounts(10, "11");//随机生成10个密码为“11”的账户
zptSdk.getWalletMgr().writeWallet();//写入钱包文件

随机创建:
AccountInfo info0 = zptSdk.getWalletMgr().createAccountInfo("11");//随机创建一个密码为“11”的账号

通过私钥创建:
AccountInfo info = zptSdk.getWalletMgr().createAccountInfoFromPriKey("11", "6aea72296201fbf18eaaa095ad8d21d13e9ed3b7cf2db8070f5b38973e7bf07c");//通过私钥创建一个密码为“11”的账号
获取账号
com.github.zeepin.account.Account acct0 = zeepinSdk.getWalletMgr().getAccount(info.addressBase58,"11");//获取上面info的账号

```



### 创建地址
包括单签地址和多签地址,单签地址是由一个公钥转换而来，多签地址是由多个公钥转换而来。
```
单签地址生成：
String privatekey0 = "c19f16785b8f3543bbaf5e1dbb5d398dfa6c85aaad54fc9d71203ce83e505c07";
String privatekey1 = "49855b16636e70f100cc5f4f42bc20a6535d7414fb8845e7310f8dd065a97221";
String privatekey2 = "1094e90dd7c4fdfd849c14798d725ac351ae0d924b29a279a9ffa77d5737bd96";

//生成账号，获取地址
com.github.zeepin.account.Account acct0 = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0), zptSdk.defaultSignScheme);
Address sender = acct0.getAddressU160();

//base58地址解码
sender = Address.decodeBase58("ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp")；

多签地址生成：
Address recvAddr = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());


```

| 方法名                  | 参数                      | 参数描述                       |
| :---------------------- | :------------------------ | :----------------------------- |
| addressFromMultiPubkeys | int m,byte\[\]... pubkeys | 最小验签个数(<=公钥个数)，公钥 |



## 数字身份管理

#### 1. 注册身份
##### 创建数字身份
```java
Identity identity = zptSdk.getWalletMgr().createIdentity("passwordtest");
//创建的账号或身份只在内存中，如果要写入钱包文件，需调用写入接口
zptSdk.getWalletMgr().writeWallet();
```
##### 向链上注册身份
###### 只有向区块链链成功注册身份之后，该身份才可以真正使用。
```java
Identity identity = zptSdk.getWalletMgr().createIdentity("passwordtest");
zptSdk.nativevm().GId().sendRegister(identity, "passwordtest", acct0, 20000, 1);
```
#### 2. 身份管理
##### 移除身份
```java
zptSdk.getWalletMgr().getWallet().removeIdentity("GID:ZPT:ZDtCV5yAeJzV3L6NF2pdaJCB837W9YQFyz"); //输入gid
zptSdk.getWalletMgr().writeWallet();
```
##### 设置默认身份
```java
zptSdk.getWalletMgr().getWallet().setDefaultIdentity(1);   //通过index设置默认identity
zptSdk.getWalletMgr().writeWallet();
```
```java
zptSdk.getWalletMgr().getWallet().setDefaultIdentity("GID:ZPT:ZNuWYd5r8g6ZhrBeEctc1P1Q8hAiZwq1LA"); //通过gid设置默认identity
zptSdk.getWalletMgr().writeWallet();
```
#### 3. 查询链上身份信息
```java
String ddo = zptSdk.nativevm().GId().sendGetDDO("GID:ZPT:ZaD1yigDDYNoFPPDt3ogWSUo9YnT7Gxqky"); //输入gid
```




### ZPT和Gala转账

**对于在主网转账，请将gaslimit 设为20000，gasprice设为1。**


#### 1. 初始化

```
String ip = "http://test1.zeepin.net";
String rpcUrl = ip + ":" + "20336";
ZPTSdk zptSdk = ZPTSdk.getInstance();
zeepinSdk.setRpc(rpcUrl);
zeepinSdk.setDefaultConnect(zptSdk.getRpc());
//设置钱包
wm.openWalletFile("wallet.dat"); //如果不存在钱包文件，会自动创建钱包文件。

```

#### 2. 查询
当发完交易之后可能需要查询交易是否已经落账，还可能需要查询账户余额。
##### 查询zeepin\Gala余额

```
zptSdk.getConnect().getBalance("ZFkd49K9Q4DwPUbNdGzfz9cJbfiEGbqJPy");        //查询zpt，Gaga两个余额
zptSdk.nativevm().zpt().queryBalanceOf("ZFkd49K9Q4DwPUbNdGzfz9cJbfiEGbqJPy"); //查询zpt余额
zptSdk.nativevm().gala().queryBalanceOf("ZFkd49K9Q4DwPUbNdGzfz9cJbfiEGbqJPy");//查询gala余额

//查ZPT信息
System.out.println(zptSdk.nativevm().zpt().queryName());             //功能说明： 查询资产名信息----返回值：资产名称
System.out.println(zptSdk.nativevm().zpt().querySymbol());          //功能说明： 查询资产Symbol信息-----返回值：Symbol信息
System.out.println(zptSdk.nativevm().zpt().queryDecimals());       //功能说明： 查询资产的精确度----返回值：精确度
System.out.println(zptSdk.nativevm().zpt().queryTotalSupply());    //功能说明： 查询资产的总供应量----返回值：总供应量

//查Gala信息
System.out.println(zptSdk.nativevm().gala().queryName());
System.out.println(zptSdk.nativevm().gala().querySymbol());
System.out.println(zptSdk.nativevm().gala().queryDecimals());
System.out.println(zptSdk.nativevm().gala().queryTotalSupply());



```

##### 查询交易是否在交易池中

```
zptSdk.getConnect().getMemPoolTxState("00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4")

打印相关信息
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

根据交易Hash查询智能合约推送内容，代表交易执行成功，如果没有成功States中不会有transfer的事件。

```
zptSdk.getConnect().getSmartCodeEvent("00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4")

打印相关信息
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
zeepinSdk.getConnect().getSmartCodeEvent(10)//查询第10块高度的智能合约事件

打印相关信息
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
| 1    |           zptSdk.getConnect().getNodeCount()           |     查询节点数量     |
| 2    |            zptSdk.getConnect().getBlock(15)            |     根据块高查询块    |
| 3    |          zptSdk.getConnect().getBlockJson(15)          | 根据块高查询块的Json信息 |
| 4    |       zptSdk.getConnect().getBlockJson("txhash")       |根据块的Hash查询块的Json信息|
| 5    |         zptSdk.getConnect().getBlock("txhash")         |   根据块的Hash查询块  |
| 6    |          zptSdk.getConnect().getBlockHeight()          |     查询当前块高     |
| 7    |      zptSdk.getConnect().getTransaction("txhash")      | 根据交易Hash查询交易信息 |
| 8    | zptSdk.getConnect().getStorage("czeepinractaddress", key) |   查询智能合约存储   |
| 9   |       zptSdk.getConnect().getBalance("address")        |       查询余额       |
| 10   | zptSdk.getConnect().getCzeepinractJson("czeepinractaddress") |     查询智能合约     |
| 11   |       zptSdk.getConnect().getSmartCodeEvent(59)        |   查询智能合约事件   |
| 12   |    zptSdk.getConnect().getSmartCodeEvent("txhash")     |   查询智能合约事件   |
| 13   |  zptSdk.getConnect().getBlockHeightByTxHash("txhash")  |   查询交易所在高度   |
| 14   |      zptSdk.getConnect().getMerkleProof("txhash")      |    获取merkle证明    |
| 15   | zptSdk.getConnect().sendRawTransaction("txhexString")  |       发送交易       |
| 16   |  zptSdk.getConnect().sendRawTransaction(Transaction)   |       发送交易       |
| 17   |    zptSdk.getConnect().sendRawTransactionPreExec()     |    发送预执行交易    |
| 18   |  zptSdk.getConnect().getAllowance("zeepin","from","to")   |    查询允许使用值    |
| 19   |        zptSdk.getConnect().getMemPoolTxCount()         | 查询交易池中交易总量 |
| 20   |        zptSdk.getConnect().getMemPoolTxState()         | 查询交易池中交易状态 |

#### 3. zeepin转账
ZPT和Gala转账可以一对一，也可以一对多，多对多，多对一。
##### 构造转账交易并发送

```
转出方与收款方地址：
Address sender = acct0.getAddressU160();
Address recvAddr = acct1.getAddressU160(); 
//多签地址生成
//Address recvAddr = Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());

构造转账交易：
long amount = 2000;
Transaction tx = zptSdk.nativevm().zpt().makeTransfer(sender.toBase58(), recvAddr.toBase58(), amount, sender.toBase58(), 30000, 1);
//转出方，转入方，金额，网络费付款人地址，所需付费=30000*1

对交易做签名：
zptSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct0}});
//多签地址的签名方法：
zptSdk.signTx(tx, new com.github.zeepin.account.Account[][]{{acct1, acct2}});
//如果转出方与网络费付款人不是同一个地址，需要添加网络费付款人的签名

//发送预执行（余额不足返回异常，余额充足则成功返回） (可选)
Object obj = zptSdk.getConnect().sendRawTransactionPreExec(tx.toHexString());

发送交易：
zptSdk.getConnect().sendRawTransaction(tx.toHexString());


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

2.添加多签签名-----多签签名分多次签
zptSdk.addMultiSign(tx, 2,new byte[][]{acct.serializePublicKey(),acct2.serializePublicKey()},acct1);   //acct1签名
zptSdk.addMultiSign(tx, 2,new byte[][]{acct.serializePublicKey(),acct2.serializePublicKey()},acct2);  //acct2签名

```


##### 一转多或多转多

1. 构造多个state的交易
2. 签名
3. 一笔交易上限为1024笔转账

```
Address sender1 = acct0.getAddressU160();  //转出方
Address sender2 =  Address.addressFromMultiPubKeys(2, acct1.serializePublicKey(), acct2.serializePublicKey());    //转出方的多签
Address recvAddr = acct1.getAddressU160(); //接收方
int amount1 = 10000;//定义交易数量
int amount2 = 20000;//定义交易数量

//构造交易
State state1 = new State(sender1, recvAddr, amount1);  //第一笔交易消息
State state2 = new State(sender2, recvAddr, amount2);//第二笔交易信息
Transaction tx = zptSdk.nativevm().zpt().makeTransfer(new State[]{state1, state2}, sender1.toBase58(), 30000,1);   //2转1交易

//第一个转出方是单签地址，第二个转出方是多签地址：
zptSdk.signTx(tx, new com.github.zeepin.account.Account[][] {{acct0}});   //转出方的单签地址
zptSdk.addMultiSign(tx,2,new byte[][]{acct1.serializePublicKey(),acct2.serializePublicKey()},acct1);//转出方多签地址 acct1签名
zptSdk.addMultiSign(tx,2,new byte[][]{acct1.serializePublicKey(),acct2.serializePublicKey()},acct2);//acct2签名  

//发送交易
Object obj = zptSdk.getConnect().syncSendRawTransaction(tx.toHexString());


```

##### 使用签名机签名
构造交易并签名
* 构造交易，序列化交易，发送交易给签名机
* 签名机接收到交易，反序列化，检查交易，添加签名
* 发送交易

```java
Transaction tx = zptSdk.nativevm().zpt().makeTransfer(sender.toBase58(), recvAddr.toBase58(), amount, sender.toBase58(), 30000, 1);

String txHex = tx.toHexString();//序列化交易发送给签名机

Transaction txRx = Transaction.deserializeFrom(Helper.hexToBytes(txHex));//接收方反序列化交易并签名

zptSdk.addSign(txRx, acct0);//签名   
```


#### 4. Gala转账

##### Gala转账接口与ZPT类似：

```
Transaction tx =  zptSdk.nativevm().gala().makeTransfer(sender.toBase58(), recvAddr.toBase58(), amount, sender.toBase58(), 30000, 1);  //构造一笔gala交易 
zptSdk.signTx(tx, new com.github.zeepin.account.Account[][] {{acct0}}); //签名
zptSdk.getConnect().syncSendRawTransaction(tx.toHexString());//发送gala交易
```

##### 提取Gala

- 查询是否有Gala可以提取
- 构造交易和签名
- 发送提取Gala交易

```
查询未提取Gala:
String addr = acct0.getAddressU160().toBase58();
String gala = zptSdk.nativevm().gala().unboundGala(addr);//查询未提取的Gala

//提取Gala
com.github.zeepin.account.Account account = new com.github.zeepin.account.Account(Helper.hexToBytes(privatekey0),zptSdk.defaultSignScheme);
String hash = zptSdk.nativevm().gala().withdrawGala(account,recvAddr.toBase58(),100000,account,20000,1);

```

| 方法名       | 参数                                                         | 参数描述                                                     |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| makeClaimGala | String claimer,String to,long amount,String payer,long gaslimit,long gasprice | claim提取者，提给谁，金额，网络付费人地址，gaslimit，gasprice |


#### 5. 原生数字资产接口
原生数字资产包括zpt和Gala。封装了构造交易、交易签名、发送交易。

```
转账:
zptSdk.nativevm().zpt().sendTransfer(acct0,acct2.getAddressU160().toBase58(),10000,acct0,zptSdk.20000,1); //从发送方转移一定数量的zpt到接收方账户
ptSdk.nativevm().gala().sendTransfer(acct0,acct2.getAddressU160().toBase58()10000,acct0,zptSdk.20000,1);//从发送方转移一定数量的Gala到接收方账户

授权转移资产:
zptSdk.nativevm().zpt().sendApprove(acct0, acct2.getAddressU160().toBase58(),10000, acct0, zptSdk.20000,1);//sendAddr账户允许recvAddr转移amount数量的资产

zptSdk.nativevm().zpt().sendTransferFrom(acct0, acct0.getAddressU160().toBase58(),acct1.getAddressU160().toBase58(), 10000, acct0, zptSdk.20000,1);// sendAcct账户从fromAddr账户转移amount数量的资产到toAddr账户


```

