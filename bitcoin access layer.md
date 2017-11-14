Bitcoin Access Layer
====================

Introduction
------------

This document describes the bridge between the UniquID library and Bitcoin: how UniquID library can use Bitcoin BlockChain to provide its services.


Cryptographic Identity
----------------------

The cryptographic ID of each Entity is represented by the binary Seed that is automatically generated on the first start.<br>
The Seed follows the same principles used for Hierarchical Deterministic Wallets of Bitcoin. See [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)


BIP32
-----

Keys (unique identities) are derived from the Seed in a hierarchical deterministic way. The procedure strictly follows [BIP32 Standard](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki). <br>

Each key is never used more than once: each Contract requires a new key generated.

The derivation process of the keys follows the [BIP44 Standard](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) with some adjustments.<br>

This is how the hierarchy is used inside the Uniquid Framework.

* **m/44'/0'** Path for the **XPub** exported in Imprinting phase.
* **m/44'/0'/0/0** is the point from where starts the hierarchy dedicated to the Provider
* **m/44'/0'/0/0/0** constant 0 is used for external chain: addresses that are meant to be visible outside of the wallet (e.g. for receiving payments)
* **m/44'/0'/0/0/1** constant 1 is used for internal chain: addresses which are not meant to be visible outside of the wallet and is used for return transaction change
* **m/44'/0'/0/0/0/0** is the address where is payed the **first charge** (Imprinting). Is also the address for the **first contract like Provider** with change address set on **m/44'/0'/0/0/1/0**.
* **m/44'/0'/0/0/0/n** are the address where send the charge. UTXO on these addresses can be used as second input in a contract. 
* **m/44'/0'/0/0/1/n** are the addresses for the Change of the contracts. From these addresses are generated all contracts after the first.
* **m/44'/0'/0/1** is the point from where start the hierarchy dedicated to the User
* **m/44'/0'/0/1/0** constant 0 is used for external chain: addresses that are meant to be visible outside of the wallet (e.g. for receiving contracts)
* **m/44'/0'/0/1/0/n** are the addresses where the token for the User's contracts are sent.

This schema is **mandatory** like is mandatory that **no address can appear in more than one contract**.

____

Smart Contract
--------------

Currently the UniquID library implement only **Smart-Contract V.0** that puts in relationship three kinds of entites: User, Provider and **Revoker**.
Smart contracts define what the user is allowed to request to the Provider and give to the Revoker the possibility to revoke this grant.
The Contract structure is defined as a bitcoin transaction that has the structure defined on the table below:

|Input    |Output                                                           | 
|:-----|:-------------------------------------------------------------------| 
| 0 - Provider Address    | 0 - User Address(Token) | 
| 1 - Recharge Address (opt.) | 1 - OP_RETURN |
||2 - Revoker Address(Token) |
||3 - Change Address|

The order of definition for inputs and outputs is mandatory. 
Only the recharge address is optional.
This kind of transaction transfer token form the Provider to the User and the Revoker and must be signed by the provider.
The transaction must be considered revoked when the Revoker has spent his token.

Custom smart contract can be defined by business application. In this case the format and parsing is completely left to business application implementors.

___
OP_RETURN
---------
The transaction for a smart contract has, beside the token transfer, defined an **OP_RETURN**.
In the OP_RETURN there resides the access information granted by the contract.
The OP_RETURN structure is of 80 bytes so defined:

|Bytes length    |Description                                                           |Notes| 
|:-----|:-------------------------------------------------------------------|:---:|
|1|Version| 0 for Smart Contract V.0| 
|4|Bitmask for System Reserved functions ||
|14|Bitmask for User Defined functions ||
|1|Number of guarantors needed|Not used|
|20|Guarantor 0 address|Not used|
|20|Guarantor 1 address|Not used|
|20|Guarantor 2 address|Not used|


The **bitmask** define if a function is accessible (**1**) or denied (**0**) from the **User** to the **Provider**.



___

Message and RPC
---
The UniquID framework define the structure of the messages used to request the function's execution.<br>
The channel is not defined inside the framwork to grant the maximum interoperability between all system's actors.<br>
The structure of the messages and the RPC is based on **JSON-RPC**.<br>
In the communication protocol the User send a request and wait a response from the Provider.
All messages are in JSON format.

### Request
```
{
    "sender":"sender-address", 
    "body":{
        "method":n, 
        "params":"param-string", 
        "id":nonce
        }
} 
```

Where:<br>
* **sender**: is the address with wich the User is presented to the Provider
* **method**: is an interger and is the number of the function called
* **params**: is the parameter passed to the function invoked. Coud be a structured JSON. The interpretation of the params value is demanded to the immplementation of the method.
* **id**: is a **nonce** that must correspond with the same parameter on response. Keep a relationship between request and response.


### Response
```
{
	"sender": "sender-address",
	"body": {
		"result": "res-string",
		"error": err,
		"id": nonce
	}
}
```
Where:<br>
* **sender**: is the Provider Address of the contract with wich the Provider has granted the excution of the requested function.
* **result**: is the result of the function invoked.Coud be a structured JSON. The interpretation of the params value is demanded to the immplementation of the method. Errors form the method must be returned in this string.
* **error**: is number that report a message error (i.e. Invalid JSON). If the message is correct, decodified, exist a contract that allow the function execution and the function was executed then this value is 0.
* **id**: is a **nonce** that must correspond with the same parameter on request. Keep a relationship between request and response.

### Process
When the Provider Entity receives a request, it must first verify that there is a contract linking it to the sender address.<br>
If the contract exist, the Provider must verify if the requested method number contains a 1 in the contract's bitmask.<br>
Only if this last check is ok the Provider can send for execution the requested function.<br>
In the reply message the sender must be valued with the address that binds Provider and User in the affected contract. <br>
If the contract does not exist, the Provider does not have to reply.<br>
