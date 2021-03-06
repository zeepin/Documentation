# zeepin Signature Server Tutorials

[English|[中文](sigsvr_CN.md)]

zeepin Signature Server - sigsvr is a rpc server for signing transactions. The signature server is bound to the 127.0.0.1 address and only supports signature requests sent by the local machine.

* [zeepin Signature Server Tutorials](#zeepin-signature-server-tutorials)
	* [1. Signature Service Startup](#1-signature-service-startup)
		* [1.1 The Parameters of Signature Service Startup](#11-the-parameters-of-signature-service-startup)
		* [1.2 Startup](#12-startup)
	* [2. Signature Service Method](#2-signature-service-method)
		* [2.1  Signature Service Calling Method](#21-signature-service-calling-method)
		* [2.2 Signature for Data](#22-signature-for-data)
		* [2.3 Signature for Raw Transactions](#23-signature-for-raw-transactions)
		* [2.4 Multiple Signature for Raw Transactions](#24-multiple-signature-for-raw-transactions)
		* [2.5 Signature of Transfer Transaction](#25-signature-of-transfer-transaction)
		* [2.6 Native Contract Invokes Signature](#26-native-contract-invokes-signature)

## 1. Signature Service Startup

### 1.1 The Parameters of Signature Service Startup

--loglevel
The loglevel parameter is used to set the log level for the sigsvr output. Sigsvr supports 7 different log levels - 0:Debug 1:Info 2:Warn 3:Error 4:Fatal 5:Trace 6:MaxLevel. The log level is from low to high, and the output log volume is from high to low. The default value is 1, which means that only output logs at the info level or higher level.

--wallet, -w
The wallet parameter specifies the wallet file path when sigsvr starts. The default value is "./wallet.dat".

--account, -a
The account parameter specifies the account address when sigsvr starts. If the account is null, it uses the wallet default account.

--cliport
The port number to which the signature server is bound. The default value is 20000.

--abi
abi parameter specifies the abi file path when sigsvr starts. The default value is "./abi".

### 1.2 Startup

```
./sigsvr
```

## 2. Signature Service Method

The signature service currently supports signature for data, single signature and multi-signatures for raw transactions, constructing ZPT/GALA transfer transactions and signing, constructing transactions that Native contracts can invoke and signing, and constructing transactions that WASMVM contracts can invoke and signing, and so on.

### 2.1  Signature Service Calling Method

The signature service is a json rpc server and adopts the POST method. The requested service path is:

```
http://localhost:20000/cli
```
Request structure:

```
{
	"qid":"XXX",    //Request ID， the response will bring the same qid
	"method":"XXX", //Requested method name
	"params":{
		//The request parameters that are filled in according to the request method
	}
}
```
Response structure:

```
{
    "qid": "XXX",   //Request ID
    "method": "XXX",//Requested method name
    "result": {     //Response result
        "signed_tx": "XXX"  //Signed transaction
    },
    "error_code": 0,//Error code，zero represents success, non-zero represents failure
    "error_info": ""//Error description
}
```

Error code:

Error code  | Error description
--------|-----------
1001 | Invalid http method
1002 | Invalid http request
1003 | Invalid request parameter
1004 | Unsupported method
1005 | Account is locked
1006 | Invalid transactions
1007 | ABI is not found
1008 | ABI is not matched
9999 | Unknown error

### 2.2 Signature for Data

SigSvr can signature for any data. Note that data must be encode by hex string.

Method Name: sigdata

Request parameters:

```
{
    "raw_data":"XXX"      //Unsigned data, Note that data must be encode by hex string.
}
```
Response result:

```
{
    "signed_data":"XXX"   //Signed data, Note that data was encoded by hex string.
}
```
Examples:

Request: 

```
{
	"qid":"t",
	"method":"sigdata",
	"params":{
		"raw_data":"48656C6C6F20776F726C64" //Hello world
	}
}
```
Response: 

```
{
    "qid": "t",
    "method": "sigdata",
    "result": {
        "signed_data": "cab96ef92419df915902817b2c9ed3f6c1c4956b3115737f7c787b03eed3f49e56547f3117867db64217b84cd6c6541d7b248f23ceeef3266a9a0bd6497260cb"
    },
    "error_code": 0,
    "error_info": ""
}
```

### 2.3 Signature for Raw Transactions

Method Name: sigrawtx

Request parameters:

```
{
    "raw_tx":"XXX"      //Unsigned transaction
}
```
Response result:

```
{
    "signed_tx":"XXX"   //Signed transaction
}
```
Examples:

Request:
```
{
	"qid":"1",
	"method":"sigrawtx",
	"params":{
		"raw_tx":"00d14150175b000000000000000000000000000000000000000000000000000000000000000000000000ff4a0000ff00000000000000000000000000000000000001087472616e736665722a0101d4054faaf30a43841335a2fbc4e8400f1c44540163d551fe47ba12ec6524b67734796daaf87f7d0a0000"
	}
}
```
Response:

```
{
    "qid": "1",
    "method": "sigrawtx",
    "result": {
        "signed_tx": "00d14150175b00000000000000000000000000000000011e68f7bf0aaba1f18213639591f932556eb674ff4a0000ff00000000000000000000000000000000000001087472616e736665722a0101d4054faaf30a43841335a2fbc4e8400f1c44540163d551fe47ba12ec6524b67734796daaf87f7d0a000101231202026940ba3dba0a385c44e4a187af75a34e281b96200430db2cbc688a907e5fb54501014101d3998581639ff873f4e63936ae63d6ccd56d0f756545ba985951073f293b07507e2f1a342654c8ad28d092dd6d8250a0b29f5c00c7866df3c4df1cff8f00c6bb"
    },
    "error_code": 0,
    "error_info": ""
}
```

### 2.4 Multiple Signature for Raw Transactions

Since the private key is in the hands of different people, the multi-signature method needs to be called multiple times.

Method Name: sigmutilrawtx

Request parameters:
```
{
    "raw_tx":"XXX", //Unsigned transaction
    "m":xxx         //The minimum number of signatures required for multiple signatures
    "pub_keys":[
        //Public key list of signature
    ]
}
```
Response result:

```
{
    "signed_tx":"XXX" //Signed transaction
}
```
Examples:

Request:

```
{
	"qid":"1",
	"method":"sigmutilrawtx",
	"params":{
		"raw_tx":"00d12454175b000000000000000000000000000000000000000000000000000000000000000000000000ff4a0000ff00000000000000000000000000000000000001087472616e736665722a01024ce71f6cc6c0819191e9ec9419928b183d6570012fb5cfb78c651669fac98d8f62b5143ab091e70a0000",
		"m":2,
		"pub_keys":[
		    "1202039b196d5ed74a4d771ade78752734957346597b31384c3047c1946ce96211c2a7",
		    "120203428daa06375b8dd40a5fc249f1d8032e578b5ebb5c62368fc6c5206d8798a966"
		]
	}
}
```

Response:

```
{
    "qid": "1",
    "method": "sigmutilrawtx",
    "result": {
        "signed_tx": "00d12454175b00000000000000000000000000000000024ce71f6cc6c0819191e9ec9419928b183d6570ff4a0000ff00000000000000000000000000000000000001087472616e736665722a01024ce71f6cc6c0819191e9ec9419928b183d6570012fb5cfb78c651669fac98d8f62b5143ab091e70a000102231202039b196d5ed74a4d771ade78752734957346597b31384c3047c1946ce96211c2a723120203428daa06375b8dd40a5fc249f1d8032e578b5ebb5c62368fc6c5206d8798a9660201410166ec86a849e011e4c18d11d64ca0afbeaebf8a4c975be1eab4dcb8795abef5c908647294822ddaadaf2e4ae2432c9c8d143ceba8fa6355d6dfe59f846ac5a41a"
    },
    "error_code": 0,
    "error_info": ""
}
```
### 2.5 Signature of Transfer Transaction

In order to simplify the signature process of transfer transaction, a transfer transaction structure function is provided. When transferring, only the transfer parameters need to be provided.

Method Name: sigtransfertx
Request parameters:
```
{
	"gas_price":XXX,  //gasprice
	"gas_limit":XXX,  //gaslimit
	"asset":"zpt",    //asset: zpt or gala
	"from":"XXX",     //Payment account
	"to":"XXX",       //Receipt address
	"amount":XXX      //transfer amount. Note that since the precision of gala is 9, it is necessary to multiply the actual transfer amount by 1000000000 when making gala transfer.
}
```
Response result:

```
{
    "signed_tx":XXX     //Signed transaction
}
```

Examples:

Request:
```
{
	"qid":"t",
	"method":"sigtransfertx",
	"params":{
		"gas_price":0,
		"gas_limit":20000,
		"asset":"zpt",
		"from":"ZTACcJPZ8eECdWS4ashaMdqzhywpRTq3oN",
		"to":"ZDsU8tYS3aSmCsNPHzjCXtiBWqpJybHmUw",
		"amount":10
	}
}
```

Response:

```
{
    "qid": "t",
    "method": "sigtransfertx",
    "result": {
        "signed_tx": "00d1184a175b000000000000000050c3000000000000011e68f7bf0aaba1f18213639591f932556eb674ff4a0000ff00000000000000000000000000000000000001087472616e736665722a0102ae97551187192cdae14052c503b5e64b32013d01397cafa8cc71ae9e555e439fc0f0a5ded12a2a0a000101231202026940ba3dba0a385c44e4a187af75a34e281b96200430db2cbc688a907e5fb545010141016a0097dbe272fd61384d95aaeb02e9460e18078c9f4cd524d67ba431033b10d20de72d1bc819bf6e71c6c17116f4a7a9dfc9b395738893425d684faa30efea9c"
    },
    "error_code": 0,
    "error_info": ""
}
```

### 2.6 Native Contract Invokes Signature

The Native contract invocation transaction is constructed and signed according to the ABI.


Note:
When sigsvr starts, it will default seek the native contract abi under "./abi" in the current directory. If there is no abi for this contract in the naitve directory, then it will return a 1007 error. Can use --abi parameter to change the abi seeking path.


Method Name: signativeinvoketx

Request parameters:

```
{
    "gas_price":XXX,    //gasprice
    "gas_limit":XXX,    //gaslimit
    "address":"XXX",    //The address that invokes native contract
    "method":"XXX",     //The method that invokes native contract
    "version":0,        //The version that invokes native contract
    "params":[
        //The parameters of the Native contract are constructed according to the ABI of calling method. All values ​​are string type.
    ]
}
```
Response result:
```
{
    "signed_tx":XXX     //Signed Transaction
}
```

Constructing a zpt transfer transaction as a example:

Request:

```
{
	"Qid":"t",
	"Method":"signativeinvoketx",
	"Params":{
		"gas_price":0,
		"gas_limit":20000,
		"address":"0100000000000000000000000000000000000000",
		"method":"transfer",
		"version":0,
		"params":[
			[
				[
				"ZDsU8tYS3aSmCsNPHzjCXtiBWqpJybHmUw",
				"ZtiBWqpJyDsHmUwU8tYS3aSmCsNPHzjCXb",
				"1000"
				]
			]
		]
	}
}
```
Response:

```
{
    "qid": "t",
    "method": "signativeinvoketx",
    "result": {
        "signed_tx": "00d161b7315b000000000000000050c3000000000000084c8f4060607444fc95033bd0a9046976d3a9f57300c66b147ce25d11becca9aa8f157e24e2a14fe100db73466a7cc814fc8a60f9a7ab04241a983817b04de95a8b2d4fb86a7cc802e8036a7cc86c51c1087472616e736665721400000000000000000000000000000000000000010068164f6e746f6c6f67792e4e61746976652e496e766f6b6500014140c4142d9e066fea8a68303acd7193cb315662131da3bab25bc1c6f8118746f955855896cfb433208148fddc0bed5a99dfde519fe063bbf1ff5e730f7ae6616ee02321035f363567ff82be6f70ece8e16378871128788d5a067267e1ec119eedc408ea58ac"
    },
    "error_code": 0,
    "error_info": ""
}
```