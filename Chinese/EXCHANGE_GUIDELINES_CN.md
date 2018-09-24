<h1 align="center">EXCHANGE_GUIDELINES_CN</h1>
<h4 align="center">Version 1.0 </h4>

[English](EXCHANGE_GUIDELINES.md) | [中文](EXCHANGE_GUIDELINES_CN.md) | [한글](EXCHANGE_GUIDELINES_KO.md)


**ZEEPIN区块链资产分为：**

* **原生资产：**
  * ZPT：主币、治理、资产等
  * Gala：交易转账、合约部署、系统燃料等
  
* **合约资产：**
  * 通过合约产生的token
  * 如MOX等

交易所和ZEEPIN区块链对接时，主要是处理这两种类型资产的：充值、提现、交易查询、账户创建等操作。  


**本文大纲：**  

* [1. ZEEPIN同步节点的部署](#1zeepin同步节点的部署)
	* [获取zeepin](#获取zeepin)
		* [从release获取](#从release获取)
	* [服务器部署](#服务器部署)
		* [MainNet同步节点部署](#mainnet同步节点部署)
* [2. ZEEPIN CLI客户端使用](#2zeepin-cli客户端使用)
	* [安全策略](#安全策略)
	* [创建ZEEPIN的钱包](#创建zeepin的钱包)
	* [为交易所用户生成充值地址](#为交易所用户生成充值地址)
		* [充值地址有两种生成方式](#充值地址有两种生成方式)
* [3. 交易所对接资产交易](#3交易所对接资产交易)
	* [需开发的对接程序](#需开发的对接程序包括)
	* [用户充值](#用户充值)
		* [通过查看“State” 判断](#通过查看state-判断)
		* [通过Notify解析数组](#notify解析数组如下)
	* [充值记录](#充值记录)
	* [处理用户提现请求](#处理用户提现请求)
* [4. 给用户分发Gala](#4-给用户分发gala)
	* [什么是Gala](#什么是gala)
	* [计算可提取的Gala总量](#计算可提取的gala总量)
	* [给用户分发Gala](#给用户分发gala)
	* [Gala分配公式](#gala分配公式)
	* [用户提现Gala](#用户提现gala)  
	
* [5. 附 native 合约地址](#附-native-合约地址)

* [6. FAQ（持续更新）](#faq-持续更新)    
  
    



## 1、ZEEPIN同步节点的部署  

### 获取zeepin
#### 从release获取
- 从[下载页面](https://github.com/zeepin/zeepinChain/releases)获取

### 服务器部署
#### MainNet同步节点部署

目录结构如下

   ```
	$ tree -L 1
	.
	├── zeepin
	└── wallet.dat
   ```



运行zeepin节点

   ```
	./zeepin
   ```
   

默认关闭websocket和rest端口，可以配置以下参数启动端口：


   ```
RESTFUL OPTIONS:
  --rest            Enable restful api server
  --restport value  Restful server listening port (default: 20334)

WEB SOCKET OPTIONS:
  --ws            Enable websocket server
  --wsport value  Ws server listening port (default: 20335)
   ```
  
zeepin -h 查看更多命令，如参数：

   ```
	--loglevel=0 日志参数
   ```
  

## 2、ZEEPIN CLI客户端使用  

### 安全策略

强制要求交易所使用白名单和防火墙，隔绝外部请求，否则会有重大安全隐患。

ZEEPIN CLI 自身不提供远程开关钱包功能，打开钱包时也没有验证过程。因此，安全策略由交易所根据自身情况制定。由于钱包要一直保持打开状态以便处理用户的提现，因此，从安全角度考虑，钱包必须运行在独立的服务器上，并参考下表配置好端口防火墙。

|   port type   | Mainnet default port |
| ------------- | -------------------- |
| Rest Port     | 20334                |
| Websorcket    | 20335                |
| Json RPC port | 20336                |
| Node port     | 20338                |


### 创建ZEEPIN的钱包


创建zeepin钱包


   ```
	./zeepin account add -d
   ```
   
接着输入密码，执行完输出：

```
Use default setting '-t ecdsa -b 256 -s SHA256withECDSA'
	signature algorithm: ecdsa
	curve: P-256
	signature scheme: SHA256withECDSA
Password:
Re-enter Password:

Index: 1
Label:
Address: ZT047K36grEi5H6BF7gLb2Z0JwBFMQRRCU
Public key: 02c7fed64a315c664034bae1257f45c9fdf8c24033f0904ce7b47b0090232323
Signature scheme: SHA256withECDSA

```

- zeepin钱包统一为Z开头，请务必保存好钱包密码和私钥，钱包地址大小写敏感，请务必注意！

- zeepin钱包私钥生成算法和NEO一致，同一个私钥对应的ZPT和NEO的公钥地址不相同。

- 交易所不需要为每个地址创建一个钱包，一个钱包可以存储所有用户的充值地址。也可以使用一个离线的冷钱包作为更安全的存储方式。

  

###  为交易所用户生成充值地址  


####  充值地址有两种生成方式： 

- 动态生成：用户创建账户时通过 Java SDK 实现动态创建 ZPT/Gala 地址并返回（优点：自动维护 / 缺点：备份不太方便）

- 批量创建：交易所批量生成，用户创建账户后分配给用户 ZPT 地址（优点：方便备份 / 缺点：定期人工生成）

  
  批量创建方法： ZEEPIN CLI 执行：
  
  ```
  ./zeepin account add -d -n [数量]  -w [钱包文件名]
  ```
  
  -d 默认值为1，即调用默认设置
  -n 批量创建的地址数量
  -w 指定钱包文件，默认为wallet.dat
  
  例如要一次创建10个zeepin地址:

```
	$ ./zeepin account add -d -n 10 -w walletEx.dat
	
	Use default setting '-t ecdsa -b 256 -s SHA256withECDSA'
		signature algorithm: ecdsa
		curve: P-256
		signature scheme: SHA256withECDSA
		
	Password:
	Re-enter Password:

	Index: 1
	Label:
	Address: ZDahjrxYu2vFUPQMDZx6bRtF7RDJs4Xkxb
	Public key: 03e38e4944da8a7bba7a12e88a679b0b0063e2362b4b82cf6bdfdc49b5c595418f
	Signature scheme: SHA256withECDSA

	Index: 2
	Label:
	Address: ZW7gQeK9o4Axs6Dg17E7yiBsUnnsRaGWbF
	Public key: 03c86673df828cc90f03497dd72eb785daf5f0b1561e4d8250eea05114a6652343
	Signature scheme: SHA256withECDSA

	Index: 3
	Label:
	Address: ZEzF5pBkfamfHBSDYvLSJbatM4iSLsZF3c
	Public key: 02f10b9250391a309e26481faa3b7c5f041dd0a7f62277cdd81527d8defce63e20
	Signature scheme: SHA256withECDSA
	
	....... .......

	Index: 10
	Label:
	Address: ZQm7vGM5ezFdJhUigbLgv2ekzvxpCp3Eqv
	Public key: 036220266424bcf381cfae8233e6422a9761ae05cd31036ede89301a940af3b353
	Signature scheme: SHA256withECDSA

	Create account successfully.
```


## 3、交易所对接资产交易  



### 需开发的对接程序包括

- 用CLI或API监控新区块
- 根据交易信息完成用户充值
- 存储相关交易记录


### 用户充值

关于用户充值，请注意以下几点：

- ZEEPIN钱包地址中包含 ZPT 和 Gala 两种资产，交易所记录用户充值时需要判断充值的资产类型，以免搞混充值的资产；

- ZEEPIN钱包是一个全节点钱包，保持在线才能同步区块，可以在 ZEEPIN CLI curblockheight 命令查看当前区块高度来判断节点状态：

  ```
	./zeepin info curblockheight
	CurrentBlockHeight:5749
  ```


举例如下：

1. 用户用zeepin钱包向交易所中的钱包地址进行充值：

```
	 $ ./zeepin asset transfer --asset zpt --from ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz --to ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM --amount 2
	Password:
	Transfer ZPT
	  From:ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
	  To:ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM
	  Amount:2
	  TxHash:2d2b866018c3d1572dd681f30d54ab2d982ece9c5915e3b778fc3d63cef66e4e

	Tip:
	  Using './zeepin info status 2d2b866018c3d1572dd681f30d54ab2d982ece9c5915e3b778fc3d63cef66e4e' to query transaction status
```  

2. 通过 ZEEPIN CLI 监控区块信息


   ```
	$ ./zeepin info curblockheight
	CurrentBlockHeight:5762
	

	$ ./zeepin info block 5762
	{
	   "Hash": "553308eb7eb5769e2624d7164962157ab427c4f33ab7af542fc2a98c3e4c4409",
	   "Size": 1375,
	   "Header": {
	      "Version": 0,
	      "PrevBlockHash": "386c4251223e569ac34c76541c9d612b0d7ce20b4eb045d57368a8e4d6c1f5a7",
	      "TransactionsRoot": "2d2b866018c3d1572dd681f30d54ab2d982ece9c5915e3b778fc3d63cef66e4e",
	      "BlockRoot": "a0a96e1b97c248cc16bb893dcf78d85ea64748707925e8271825c9c71efe88ff",
	      "Timestamp": 1535005866,
	      "Height": 5762,
	      "ConsensusData": 5950638681301161907,
	      "ConsensusPayload": "7b226c6561646572223a31342c227672665f76616c7565223a22424c46497730747476712b68566e68537652394b6d366f6f77346a4b34473247345a4c71324f34707a4a7566557430716d65774d654337374833474e53363575583835716d2b5355336c6452522b3732773264627066513d222c227672665f70726f6f66223a226e427765362f74645a484b62416e6864705364636154466647417445346d315a623462546d38633331666b653779724e70534e42312f7174693443327a796674737a494667785347517133496f522b7a38776c4630413d3d222c226c6173745f636f6e6669675f626c6f636b5f6e756d223a333235372c226e65775f636861696e5f636f6e666967223a6e756c6c7d",
	      "NextBookkeeper": "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcTvb44rA",
	      "Bookkeepers": [
		 "02dfb488eb1f0116bb099584b1f058c525db1b45b24378314ba7f5cc2da180d724",
		 "037a53745eb295a6263d00e87b5f6641f22b44fc2436811a6415a69b816bea3571",
		 "0203b36fa517ac4751b37652e0cef6b730cca8c5540d9cddfb469c26b78f7836e3",
		 "023ea9e036521ed242c6102f8cdc112d901025ef3fb883213112e30da0590ab1fc",
		 "02dfb488eb1f0116bb099584b1f058c525db1b45b24378314ba7f5cc2da180d724",
		 "029c5ecc8400530cc410288496feda44542b6f5aaeff8b925d9b8c6d12a65d4bc3",
		 "035ad9d8b8350b113cbb3d541e0a89dfd10c981702eb59bce4d1a2bbd13b103a39"
	      ],
	      "SigData": [
		 "a8d9dcc7cd3878122ca841efb60fd1c470a81b9a12c96c95c69cd9ce8876754db1d2c24ca9b592c525c5e98e3dc0df5aaf66278dae2b14679d80edd47953f66c",
		 "9f372d29c829c88b8f0e55fa44cb1543fbe1f7042dbd55479fec5f3f47d10ebeb473a0d5fe41eda1f1510671706ef8b339117470fee6cffe9dfba150537ed2f5",
		 "5b6a82c3591e17193d121f00dde54fe1e2fdf4640484e0d2201089f5df63157768383667a0bfb40c4c711d4e6a92a3ba75e6a8fd6c14c5c2257e7ae2fe75b090",
		 "f177287021afe2b4393e52ff2b630cf1759b5800a6c7a41834d4febad83e506b0c5584895fcd5793c242b9124116d4dbdc87d3c3f86b1c15454be3d3a68ddc27",
		 "28d18cf67db32d146b404b1b8507003ffa240f274abc96f36f6c58ede1043ba807d359346d32fc9ba7edc53841ebf7a102e6dce063bbfc48588ca1cbaed8daef",
		 "81f338f39e76ef56e366c340f580ebc998a4063db39b939e890887c2f2bb49fd2d0a6d6fcc6845d771f9769a996ab9197c95c0fdd553ca603b956770bcc0759d",
		 "955e9e348be138636c078d08f3cf7f350cf6021a1cefd03176dda9e459f1632234a71fc8fd2fe93dbe334bf5c98d08384541d63bfbe499efe2acfb0ab3e2d8ee"
	      ],
	      "Hash": "553308eb7eb5769e2624d7164962157ab427c4f33ab7af542fc2a98c3e4c4409"
	   },
	   "Transactions": [
	      {
		 "Version": 0,
		 "Nonce": 1535005865,
		 "GasPrice": 1,
		 "GasLimit": 20000,
		 "Payer": "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		 "TxType": 209,
		 "Payload": {
		    "Code": "00c66b14a34d23aead66348272a8b5a6bdc235fd5a90ab5f6a7cc814a8e9a5aef175ccdd6503f0336701414a9c2009636a7cc802204e6a7cc86c51c1087472616e736665721400000000000000000000000000000000000000010068195a656570696e436861696e2e4e61746976652e496e766f6b65",
		    "GasLimit": 0
		 },
		 "Attributes": [],
		 "Sigs": [
		    {
		       "PubKeys": [
			  "02c35cfe4126b7a56c63f75e89d8482bc2d7bdcda44c64172e829efe76d1f57295"
		       ],
		       "M": 1,
		       "SigData": [
			  "bbdc4f00ed4d03905344b54ecdd657c73f530cf811e88299966d6f378fbfba517d4ca395a0fc323edac08338a7784d7942de68dede79582cd1aa18b158cedca4"
		       ]
		    }
		 ],
		 "Hash": "2d2b866018c3d1572dd681f30d54ab2d982ece9c5915e3b778fc3d63cef66e4e",
		 "Height": 0
	      }
	   ]
	}

   ```

3. 根据Transaction Hash 取得block中的所有Transaction信息

```
	$ ./zeepin info status 2d2b866018c3d1572dd681f30d54ab2d982ece9c5915e3b778fc3d63cef66e4e
	Transaction states:
	{
	   "TxHash": "2d2b866018c3d1572dd681f30d54ab2d982ece9c5915e3b778fc3d63cef66e4e",
	   "State": 1,
	   "GasConsumed": 20000,
	   "Notify": [
	      {
		 "ContractAddress": "0100000000000000000000000000000000000000",
		 "States": [
		    "transfer",
		    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		    "ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM",
		    20000
		 ]
	      },
	      {
		 "ContractAddress": "0200000000000000000000000000000000000000",
		 "States": [
		    "transfer",
		    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		    "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
		    20000
		 ]
	      }
	   ]
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



### 充值记录

原理同用户充值相同，交易所需要写代码监控每个区块的每个交易，
在数据库中记录下所有充值和提现交易，如果有充值交易就要修改数据库中的用户余额。



### 处理用户提现请求

当用户提现时，交易所需要完成以下操作：

1、 数据库中记录用户提现，修改用户账户余额。
2、 使用CLI命令对用户提现地址进行转账：

   ```
	   $ ./zeepin asset transfer --asset gala --from ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz --to ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM --amount 100
	Password:
	Transfer GALA
	  From:ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
	  To:ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM
	  Amount:100
	  TxHash:00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4

	Tip:
	  Using './zeepin info status 00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4' to query transaction status

   ```

   zeepin asset transfer 命令的参数如下：

   --wallet, -w  
   wallet指定转出账户钱包路径，默认为根目录下的"wallet.dat"

   --gasprice  
   zeepin网络中gasprice最小为1；
   gasprice * gaslimit 为账户实际支付的 Gala 燃料费用（每笔转账的最小燃料数值为2个Gala）；
   gasprice参数指定转账交易的gas price。交易的gas price不能小于接收节点交易池设置的最低gas price，否则交易会被拒绝。默认值为0。
   交易池会按照gas price由高到低排序，gas price高的交易会被优先处理。

   --gaslimit  
   zeepin网络中gaslimit最小值为20000（4位精度，即2个Gala）；
   gaslimit参数指定最大的gas使用上限。但实际gas花费由VM执行的步数与API决定，假定以下2种情况:  
   1. gaslimit>=实际花费，交易将执行成功，并退回未消费的gas；
   2. gaslimt<实际所需花费，交易将执行失败，并消费掉VM已执行花费的gas;  
   
   zeepin网络中gaslimit最小值为20000（4位精度，即2个Gala），少于这个数量交易将无法被打包。
   

   --asset  
   asset 	参数指定转账的资产类型，zpt表示ZPT，gala表示Gala。默认值为zpt

   --from   
   from		参数指定转出账户地址

   --to  
   to		参数指定转入目标账户地址

   --amount   
   amount	参数指定转账金额。
   
   注意：由于ZPT和Gala的精度是4，如果输入超出4位小数，超出部分的数值会被丢弃；
   
   
   确认交易结果：

   - 使用返回的交易hash直接查询并过滤交易所地址向用户转账的记录：

     ```
	$ ./zeepin info status 00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4
	Transaction states:
	{
	   "TxHash": "00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4",
	   "State": 1,
	   "GasConsumed": 20000,
	   "Notify": [
	      {
		 "ContractAddress": "0200000000000000000000000000000000000000",
		 "States": [
		    "transfer",
		    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		    "ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM",
		    1000000
		 ]
	      },
	      {
		 "ContractAddress": "0200000000000000000000000000000000000000",
		 "States": [
		    "transfer",
		    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		    "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
		    20000
		 ]
	      }
	   ]
	}

     ```


3. 从返回的 Json 格式交易详情中提取交易ID记录在数据库中

4. 等待区块确认后将提现记录标志为提现成功

   类似充值时对区块链的监控，提现也一样，监控时若发现区块中的某个交易 ID 与提现记录中的交易 ID 相等，则该交易已经确认，即提现成功。

5. 如果交易始终没有得到确认，即通过交易hash查询不到对应的event log,则需要

   - 通过rpc/SDK接口查询交易是否在交易池中（参照Java Sdk）若在，需要等待共识节点打包出块后再查询

   - 若不在，则可认为该交易失败，需要重新进行转账操作。

   - 若该交易长时间没有被打包，可能是由于gasprice设置过低。

 



## 4. 给用户分发Gala

交易所可以选择是否给用户分发Gala， Gala用于支付zeepin区块链的转账交易、合约部署、记账费用和网络等附加服务。


### 什么是Gala

在Zeepin经济模型中，ZPT总发行量恒定为10亿，Gala总量恒定为1000亿，800亿处于锁定状态，ZPT和Gala的精度都是4，对应分发的Gala总量为200亿，其中20亿已经空投至ZPT持有者，其余180亿将逐步解绑至ZPT持有者（最小解绑单位为1个ZPT）。
当一笔ZPT交易在区块链网络中产生，该笔交易将触发Gala解绑，此部分Gala将由智能合约自动转账至发起人与接收人，此时ZPT持有者能获得的Gala奖励将与持有量成正比。
如果特定地址的交易一直不被触发，该地址的Gala将持续累积；当下一笔交易触发时，一次性发放所有Gala，该Gala数量可以在Zeepin钱包应用ZeeWallet中通过Claim挖取。


### 计算可提取的Gala总量

180亿Gala将通过时间段调整解绑总数量，时间段的单位为年（具体为31536000秒），解绑数量的规则遵从斐波那契数列。为了补偿网络节点与早期持有者，前两年解绑数量为最高值，之后呈递减状，具体数值为[89, 89, 55, 55, 55, 34, 34, 34, 21, 21, 21, 13, 13,13, 8, 8,5,5,]。经过约18年后，所有Gala将解绑完毕，此后不会有新的Gala产生。

按照该解绑比例，第一年与第二年将会解绑31.18%的Gala，而前4年这一比例将增加至50.46%，大幅度增加了早期持有者的收益。
假设一名用户持有10000个ZPT，在第一年他将获得76.9 Gala/天，2338.9 Gala/月，28067.0 Gala/年。

[详细解绑规则请参照Zeepin经济模型](https://zeescan.io/tool/documents/economicModel)


### 给用户分发Gala

通过CLI查看未解绑Gala余额：


```
$ ./zeepin asset unboundgala ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
Unbound GALA:
  Account:ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
  GALA:144521.6129

```

通过CLI提取解绑的Gala：


```
$ ./zeepin asset withdrawgala ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
Password:
Withdraw GALA:
  Account:ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
  Amount:144521.6129
  TxHash:3612d3cbb4a58956258f0aa7dce35da673aedf3af3f5271ce68bcfb1ed2755d4

Tip:
  Using './zeepin info status 3612d3cbb4a58956258f0aa7dce35da673aedf3af3f5271ce68bcfb1ed2755d4' to query transaction status
```

查询解绑状态

```
$ ./zeepin info status 3612d3cbb4a58956258f0aa7dce35da673aedf3af3f5271ce68bcfb1ed2755d4
Transaction states:
{
   "TxHash": "3612d3cbb4a58956258f0aa7dce35da673aedf3af3f5271ce68bcfb1ed2755d4",
   "State": 1,
   "GasConsumed": 20000,
   "Notify": [
      {
         "ContractAddress": "0200000000000000000000000000000000000000",
         "States": [
            "transfer",
            "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcTzHMV8a",
            "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
            1445216129
         ]
      },
      {
         "ContractAddress": "0200000000000000000000000000000000000000",
         "States": [
            "transfer",
            "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
            "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
            20000
         ]
      }
   ]
}
```

查询balance可以看到Gala已经解绑到账户中：

```
$ ./zeepin asset balance ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
BalanceOf:ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
  ZPT:9989996.9989
  GALA:10143405.6129
```


### Gala分配公式

假设交易所每日凌晨0点进行分发Gala清算，根据用户昨日0点持仓快照，计算得出未调整前用户当日应分发的Gala总量 *Gala_Unbound*，扣除或加上当日客户交易和冲提币等操作造成的持仓变动影响的总量 *Gala_Adj*。

根据客户交易和冲提币流水记录计算，根据记录发生时间距清算时间的时间跨度（以秒为单位）*Delta*、发生交易的ZPT数量 *Amount* 和该时间段内的分配系数*Beta* ，计算该比交易流水造成的Gala分配调整额度，期间交易为卖出或转出则扣除这部分Gala，期间交易为买人或转入则加上这部分Gala：

				Gala_Adj =（±）delta * Amount * Beta
                
               
应该分配Gala数量为:


				Gala_D = Gala_Unbound + Sum(Gala_Adj_1 , Gala_Adj_2 ... Gala_Adj_n)


### 用户提现Gala

用户提现Gala的流程和提现ZPT的流程一致，只需指定asset 参数为gala即可：

```
$ ./zeepin asset transfer --asset gala --from ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz --to ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM --amount 10000
Password:
Transfer Gala
  From:ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz
  To:ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM
  Amount:10000
  TxHash:3612d3cbb4a58956258f0aa7dce35da673aedf3af3f5271ce68bcfb1ed2755d4

Tip:
  Using './zeepin info status 3612d3cbb4a58956258f0aa7dce35da673aedf3af3f5271ce68bcfb1ed2755d4' to query transaction status

```

使用Java SDK 提现Gala，请参照[Java SDK:Gala转账](JAVA_SDK_CN.md)



## 附 native 合约地址

合约名称 | 合约地址 
---|---
ZPT Token | 0100000000000000000000000000000000000000
Gala Token | 0200000000000000000000000000000000000000 
Zeepin Network GID(Galaxy ID) | 0300000000000000000000000000000000000000 
Global Environment | 0400000000000000000000000000000000000000  
Oracle Machine | 0500000000000000000000000000000000000000 
Authorization Contract | 0600000000000000000000000000000000000000 
Governance(Consensus) | 0700000000000000000000000000000000000000 





# FAQ （持续更新） 

- ***如何连接测试网？***

  运行zeepin测试节点：

   ```
	./zeepin --networkid 2
   ```
   *可通过 ./zeepin -h 查询更多参数设置*  
   
  <br/>   
  
- ***节点白名单运行方式***  
    
    
  为了保障交易所数据安全，避免恶意节点干扰，可以采用指定peers白名单的方式运行节点，peer.rsv内容格式和peers.recent一致，启动方式如下：

   ```
	./zeepin  --reservedonly --reservedfile=./peers.rsv
   ```  
   如有需要，可向官方联系获得白名单节点地址  
     
  <br/> 
  
- ***zeepin官方由区块链浏览器吗？***

  [https://zeescan.io](https://zeescan.io)  

  <br/>  
  
- ***ZPT和Gala的精度是多少？***

  ZPT和Gala的精度是4，如果输入超出4位小数，超出部分的数值会被丢弃  

  <br/>  
  
- ***转账手续费需要多少？***

  
  ```
  gasprice * gaslimit = gala cost
  ```
   ***--gasprice***
   zeepin网络中gasprice最小为1；
   gasprice * gaslimit 为账户实际支付的 Gala 燃料费用（每笔转账的最小燃料数值为2个Gala）；
   gasprice参数指定转账交易的gas price。交易的gas price不能小于接收节点交易池设置的最低gas price，否则交易会被拒绝。默认值为0。
   交易池会按照gas price由高到低排序，gas price高的交易会被优先处理。

   ***--gaslimit*** 
   zeepin网络中gaslimit最小值为20000（4位精度，即2个Gala）；
   gaslimit参数指定最大的gas使用上限。但实际gas花费由VM执行的步数与API决定，假定以下2种情况:  
   1. gaslimit>=实际花费，交易将执行成功，并退回未消费的gas；
   2. gaslimt<实际所需花费，交易将执行失败，并消费掉VM已执行花费的gas;  
   
   zeepin网络中gaslimit最小值为20000（4位精度，即2个Gala），少于这个数量交易将无法被打包。  
   
  <br/>  
  
- ***转账交易发生时，如何区别转账的 amount 和 gas ？***
  
  识别方法如下：
  交易手续费在 Transaction states 中直接标注在 GasConsumed 中，
  在 Notify 中问最后一组
  同时该手续费转向治理合约对应的唯一地址：ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp（示例为testnet）

	```
	$ ./zeepin info status 00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4
	Transaction states:
	{
	   "TxHash": "00d9336a5e83754815fdd609f7ecce31135428d4fcc40469082658cfdb8b62c4",
	   "State": 1,
	   "GasConsumed": 20000,
	   "Notify": [
	      {
		 "ContractAddress": "0200000000000000000000000000000000000000",
		 "States": [
		    "transfer",
		    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		    "ZTSPC1PEhXHZZDTFtvRDjoKSZrgYboBwDM",
		    1000000
		 ]
	      },
	      {
		 "ContractAddress": "0200000000000000000000000000000000000000",
		 "States": [
		    "transfer",
		    "ZSviKhEgka2fZhhoUjv2trnSMtjUhm3fyz",
		    "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
		    20000
		 ]
	      }
	   ]
	}
	```  
  <br/>  
  
- ***NEP5上的 ZPT 怎么和主网的 ZPT 兑换？***

  NEP5上的私钥通过SDK或ZeeWallet激活，获得新的Zeepin钱包地址。  
  Gala同样按此处理  
  
  <br/>

- 
- 
- 
- 







