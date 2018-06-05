UniquID Architecture
====================

Introduction
------------
The UniquID framework defines the concepts of **Entity** and **Contract**.
An Entity is a software component that contains:

* cryptographic-ID: the unique identity - is a mechanism that uses ECDSA to digitally sign data. This component must wrap digital keys and avoid to leak them outside. Keys are used to sign Contracts and for authentication
* contract exchanger and validator: a component that allow to exchange Contracts between Entities and validate their content.
* contract repository: a local data store for contracts that allow easy contract lookup/manipulation to the Entity.
* communication helper: the interface of the Entity with the external world. It allows to exposes Entity functionalities to user code such as signing, signature verification and contract verification

A Contract represents an agreement between Entities and defines how they can interact. The contract contains Entities' signatures that help to provide authentication mechanism.

This is a representation of Entity building blocks:

```
 ----------------------------------------------------------------------------------------------------
|                                                                                                    |
|                     communication helper                                                           |
|                                                                                                    |
 ----------------------------------------------------------------------------------------------------
|                                             |                            |                         |
|       contract exchanger and validator      |      active contract DB    |     identity            |
|		( blockchain wallet )         |    (contract register)     | (bip32 secure element)  |
|                                             |                            |                         |
 --------------------------------------------- ---------------------------- ------------------------ |

```

The BlockChain is used to transport and validate **Contract** between the Entities.

If the Entity performs a request then it's called **User**. If the Entity receives a requests it's called **Provider**.

Every Entity intrinsically has always the provider functionality in order to allow the signing of Contracts from the UniquID framework.


Functions
---------

In order to allow the UniquID framework to work correctly, a special kind of Contract between the Entities was defined. This contract permit the Entity to sign Contracts and broadcast them on the BlockChain.

The default implementation was done using an **RPC** mechanism: the Provider receives a requests from the User asking to perform a defined set of functions. The Provider will check if there is a valid Contract that authorize the User to perform the request.

The functions are identified by a number in range  between 0 and 143 and are divided into two separated groups:

* [**System Reserved**](../Documents/systemreserved.md):  range   **[0,31]** are reserved for framework managment 
* **User Defined** : range **[32,143]** can be used users of the library to implement business applications.

Every Entity always has the provider functionality (almost for the System Reserved functions)
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
Identity
--------
Each Entity has an **Identity** that is determined by possession of the **private key** that is never exposed out of the entity's boundaries.<br>
This Private Key is represented by the bitcoin address present in transaction.<br>
The addresses generation is defined trought the BIP32 standard: for each transaction a new pair of keys for the Provider and the User should be created.<br>

___
BIP32
---
The addresses generation is based on [BIP32 Standard](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki). <br>
Starting from a **Seed** the system will generate keys in a deterministic way. However, from the outside, all the  keys will be unrelated from each other.<br>
This Seed is generated at the first run of the Entity and is based on a strong **RNG**.<br>
This is also the reason why the Seed could be considered the **Identity of the Entity**.

Below is reported how the BIP32's hierarchy is used inside the Uniquid Framework.

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

___
Imprinting
----------
When an Entity has created his identity, no transaction on the block chain involve his addresses.<br>
**Imprinting** is the name of the process that allow another Entity, called **Imprinter**, to obtain the control of the newly created Entity.<br>

The Imprinter take, from the new Entity, the Xpub at path **m/44'/0'**  and the **Name** used to identify the entity on the communication channel for message delivery.<br>
Obtained the Xpub the Imprinter send some token at address **m/44'/0'/0/0/0/0**.

This first transaction is called **Imprinting Contract** 'cause the Entity that send token acquire the access to all **system reserved functions**.

From this moment the Imprinter is the only owner of the entity just imprinted.<br>
The imprinter later sends to the newly imprinted entity a contract to  **sign**  that specify that a third entity (called **orchestrator** ) is allowed to request him the system reserved functions.<br>

After that, the Xpub and the unique name are transferred **from** imprinter **to** the orchestrator that now become the new owner of the Entity. The original **imprinting contract** is automatically revoked.

The orchestrator is, from this moment, allowed to manage the creation of the Entity's contracts.
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
* **result**: is the result of the function invoked. Coud be a structured JSON. The interpretation of the params value is demanded to the immplementation of the method. Errors form the method must be returned in this string.
* **error**: is a number that represents the status of the execution. It's 0 in case everything is ok, 3 in case the requested function is not available in the provider, 4 in case of a generic error. In case of error, the **result** field contains the reason.
* **id**: is a **nonce** that must correspond with the same parameter on request. Keep a relationship between request and response.

### Process
When the Provider Entity receives a request, it must first verify that there is a contract linking it to the sender address.<br>
If the contract exist, the Provider must verify if the requested method number contains a 1 in the contract's bitmask.<br>
Only if this last check is ok the Provider can send for execution the requested function.<br>
In the reply message the sender must be valued with the address that binds Provider and User in the affected contract. <br>
If the contract does not exist, the Provider does not have to reply.<br>
