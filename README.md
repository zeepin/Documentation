
<h1 align="center">ZEEPIN</h1>
<h4 align="center">Version 0.1.0 </h4>

[English](README.md) | [中文](README_CN.md) | [한글](README_KO.md)


[Zeepin Chain Whitepaper EN](https://www.zeepin.io/pdfs/Zeepin%20Chain%20Tech%20WP%20V1.0%20EN.pdf) | [Zeepin Chain Whitepaper CH](https://www.zeepin.io/pdfs/Zeepin%20Chain_WP%20CN%20V1.0.pdf)


Welcome to see the source repository of Zeepin!

Zeepin Chain is a decentralized chain for cultural and entertainment assets. By providing a standardized infrastructure through blockchain technology, Zeepin Chain offers efficient work solutions to content creators, helps creative organizations improve innovation efficiency, and promotes open, transparent, fair, and effective value circulation in creative industries. Efficient.

Meanwhile, Zeepin Chain provides a blockchain digital entertainment asset issuance platform, blockchain technical support, and construct adoption scenarios in order to tokenize global creative and cultural assets. As an industry-based chain, Zeepin Chain public chain has the capability to integrate third-party entertainment assets and systems to create a free trading market and exchange platform.

Zeepin Chain builds a comprehensive blockchain technology framework. Using the GBFT-POS consensus mechanism (Galaxy Consensus), it offers a Turing-complete virtual machine as the execution environment for smart contracts, as well as customized footstep control support for the application framework. Zeepin Chain supports scripts developed in programming languages ​​such as Java, C#, Python, Javascript, etc., and the virtual machine can integrate and interact with the chain through API.


Zeepin is committed to creating a blockchain infrastructure with free configurations, scalability, and high-performance, which makes deploying blockchain environments and developing dApps much easier. Taking the demands from the cultural industry into consideration, Zeepin Version 0.1.0 is customized based on the Ontology 1.0 core framework. Currently, codes are going through a process of rapid iterative development and testing. We sincerely welcome more developers to join the Zeepin technology community!
  
  
> EXCHANGE GUIDELINES
> * [EXCHANGE GUIDELINES](./English/EXCHANGE_GUIDELINES_EN.md)
> * [JAVA SDK](./English/JAVA_SDK_EN.md)
> * [BlockChain Explorer https://zeescan.io](https://zeescan.io)
  
  
## Outline:

* [Obtaining Zeepin](#Obtaining Zeepin)
    * [Obtaining from release](#Obtaining from release)
* [Server Deployment](#Server Deployment)
    * [Choosing a Net](#Choosing a Net)
        * [Deploying TestNet Synchronized Nodes](#Deploying TestNet Synchronized Nodes)
        * [Deploying MainNet Synchronized Nodes](#Deploying MainNet Synchronized Nodes)
        * [Deploying MainNet Campaigning Nodes](#Deploying MainNet Campaigning Nodes)
    * [Creating Zeepin Wallets](#Creating Zeepin Wallets)
    * [An Examples of ZPT Transfers and Allocation](#An Examples of ZPT Transfers and Allocation)
    * [Query Transaction Results TxHash](#Query Transaction Results TxHash)
    * [An Example on Querying Account Balance](#An Example on Querying Account Balance)
    * [An Example on Querying Unbound Gala](#An Example on Querying Unbound Gala)
    * [An Example on Withdrawing Unbound Gala](#An Example on Withdrawing Unbound Gala)
    * [An Example on Querying Current Block Height](#An Example on Querying Current Block Height)
    * [An Example on Querying Block Information](#An Example on Querying Block Information)
* [Official Community](#Official Community)
    * [Official Website](#Official Website)
    * [Telegram](#Telegram)
* [License](#License)


## Obtaining Zeepin
### Obtaining from release
- Obtain from the[Download Page](https://github.com/zeepin/zeepinChain/releases)

## Server Deployment
### Choosing a Net
Zeepin operation supports the following approach

* Deploying TestNet Synchronized Nodes
* Deploying MainNet Synchronized Nodes
* Deploying MainNet Campaigning Nodes

Parameters: --networkid: 1 is the default MainNet; 2 is the TestNet; 3 is Stand-Alone Operation

#### Deploying TestNet Synchronized Nodes

Run zeepin

   ```
	./zeepin --networkid 2
   ```

#### Deploying MainNet Synchronized Nodes

Run zeepin

   ```
	./zeepin
   ```


#### Creating Zeepin Wallets


   ```
	./zeepin account add -d
   ```
Then enter the password and execute. Following is the display:

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
Please make sure to keep the wallet password and the private key safe. All Zeepin Wallets start with a “Z.”

   
### An Examples of ZPT Transfers and Allocation
   - from: Output Address; - to: Input Address; - amount: Output Amount；

```shell
  ./zeepin asset transfer --from ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G  --to ZTA8f5U8Zd1gELkje7fJmYmdmMMm1WxPyv --amount 5
```

Execute and the display follows:

```
Transfer ZPT
  From:ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G
  To:ZTA8f5U8Zd1gELkje7fJmYmdmMMm1WxPyv
  Amount:5
  TxHash:78cd8097ed58cf06b8f7d7591f05657afb9f32686ef88956c246b3a4146f6ec2

Tip:
  Using './zeepin info status 78cd8097ed58cf06b8f7d7591f05657afb9f32686ef88956c246b3a4146f6ec2' to query transaction status
```
It is possible to query transaction results via TxHash after at least one block time.


### Query Transaction Results TxHash

```shell
./zeepin info status 78cd8097ed58cf06b8f7d7591f05657afb9f32686ef88956c246b3a4146f6ec2
```
Query Results:
```shell
Transaction states:
{
   "TxHash": "78cd8097ed58cf06b8f7d7591f05657afb9f32686ef88956c246b3a4146f6ec2",
   "State": 1,
   "GasConsumed": 10000000,
   "Notify": [
      {
         "ContractAddress": "0100000000000000000000000000000000000000",
         "States": [
            "transfer",
            "ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G",
            "ZTA8f5U8Zd1gELkje7fJmYmdmMMm1WxPyv",
            50000
         ]
      },
      {
         "ContractAddress": "0200000000000000000000000000000000000000",
         "States": [
            "transfer",
            "ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G",
            "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcUiYGcPp",
            10000000
         ]
      }
   ]
}
```
Number as TxHash query results should be divided by 10000, because both ZPT and Gala are with 4 decimal places.


### 查询账户余额示例

```shell
./zeepin asset balance ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G
```
An Example on Querying Account Balance:
```shell
  BalanceOf:ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G
  ZPT:100098945
  GALA:100026953.517
```


### An Example on Querying Unbound Gala

```shell
./zeepin asset unboundgala ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G
```

Query Results:
```shell
  Unbound GALA:
  Account:ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G
  GALA:10129.2123
```

### An Example on Withdrawing Unbound Gala

```shell
./zeepin asset withdrawgala ZJohWxMxiMWHczSCV5ZUybZEf5jh9VQE5G
```


### An Example on Querying Current Block Height

```shell
./zeepin info curblockheight
```

An Example on Querying Block Information:

```
CurrentBlockHeight:1693
```

### Display the Current Block Height:

```shell
./zeepin info block 1693
```

Execute and the display follows:
```
{
   "Hash": "ea0720ca0ad21b408136bf233c4a9e11be26e8d26cee676c23d8463abbc0200a",
   "Size": 1011,
   "Header": {
      "Version": 0,
      "PrevBlockHash": "56f92d7953d9adec9afa8a029310992e15f23ad9308ef27cbe6766a3cb9245d1",
      "TransactionsRoot": "0000000000000000000000000000000000000000000000000000000000000000",
      "BlockRoot": "8ae95b3b5773532804e115acc38e890f64ae26e2242bdc66ba3b486e4ad2a2d9",
      "Timestamp": 1534287845,
      "Height": 1693,
      "ConsensusData": 13335926600357978572,
      "ConsensusPayload": "7b226c6561646572223a31312c227672665f76616c7565223a2242476162765173586334755a7641336e454549656a7853723177566849764833723330613864685471794c573872634b2b3872534246374a67476a4b46553741684238446c505a765a5542757271595364525a544857303d222c227672665f70726f6f66223a225a486457346e756b4443704e616a567454386b47344367442b3539425548324a37384c3571797951507561347335616767467856636b62705475486158304236596d4e676e664f42364d3047434a68743243697346773d3d222c226c6173745f636f6e6669675f626c6f636b5f6e756d223a313638372c226e65775f636861696e5f636f6e666967223a6e756c6c7d",
      "NextBookkeeper": "ZC3Fmgr3oS56Rg9vxZeVo2mwMMcTvb44rA",
      "Bookkeepers": [
         "02127d484040ddb9bba2668530f3d8674877af456e7d3d3f13d290f4b804b90645",
         "0301ddda1e31cb38c2b11a6772ba0c0f74fb0edcd1c92b490ba62803b2a054bc04",
         "0397380e865977ff77123c8f7f89d8d6f6dd3baef854a2253c7544200baf0d9372",
         "02b6da8f38f622b34a5cc0193b01fd7c9fbcfc631f867a81540410e337f7f12944",
         "023ecc0e7103a522e0dab399e40bd53be123b8f7eccfbf3a86ad84224aeed4b132",
         "02f8c7c0bdc8db04d33a78d662a0b3d584942e778bd6ff54df947e4403b54340a5"
      ],
      "SigData": [
         "3a88a57915f1219472a55f1bcb43837ef351f869511b8723cedb2a0f584d568c57ffd6758698da67310258b5e962ec0ed4a3408635ab05fa264da0fc41e24666",
         "abaecf69bdb839a178a321b86a2f35f0e0ee7a89bca3548ebdfa61c76e332d1fe721d76641c3dc8c11f2aee5fdb3bccc2203612b1c2b3f9573845a8ab1c07ce1",
         "4e52ac5729202edb72c802365451b1d25a7c475a98914b67d0c78d7e93ceb07a89cb73d29155bd3df9df70256a478ca92dcce8253539824256860f7b92803844",
         "005f286a0e232298294781bbc97580bd470a79f3f805c6417403836fdda8551a659c4a4b4f34e4d181accf3c9951f6510b462baad85a77b8da3b48c0c8e9c69f",
         "420cb8b17da30cbb9dc05fd3d18ecc0ce62110f3393b5c58f37e80197313103b674785f6e4069b78df713c9f855cde5c3302384478ec55478b206d63cdaeb074",
         "fe7f93c35a186440de6f77410693bfffae9c34a07d2bd0087ac4a91bb1d882872c0d6d205c7291a5f69915feec60f502b4cd40d679bec5783b5572b01ffb33cf"
      ],
      "Hash": "ea0720ca0ad21b408136bf233c4a9e11be26e8d26cee676c23d8463abbc0200a"
   },
   "Transactions": []
}
```

## Official Community

### Official Website

- https://zeepin.io/
- https://twitter.com/ZeepinChain
- https://medium.com/@zeepin
- https://www.reddit.com/r/ZEEPIN/
- https://www.facebook.com/ZeepinChain/

### Telegram

- https://t.me/zeepin
- https://t.me/ZeepinNews


## License

Zeepin complies with the GNU Lesser General Public License, version 3.0. Please find the LICENSE file in the root directory of the project for details.