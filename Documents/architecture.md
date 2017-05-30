Uniquid Architecture
====================

Introduction
------------
The UniquID framwork is builded over the **Entities**.
An Entity is any device or system that runnin the UniquID library.
In every entity we can detect two kind of functionality: **Provider** and **User**.
The Proveder send, across an **RPC** functionality to the user.

The functions are identified by a number in range  between 0 and 143.
The functions are divided into two separeted groups:

* [**System Reserved**](Documents/systemreserved.md):  range   **[0,31]** are reserved for framework managment 
* **User Defined** : range **[32,143]**  are reserved to be implemented in the applicaiton business layer.

Every Entity has  the provider functionality (almost for the System Reserved functions)
____

Smart Contract
--------------

At the moment the UniquID library implement only **Smart-Contract V.0** that put in relationship three kinds of entites: User, Provider and **Revoker**.
Are definied how functions of the provider can be called by user and give to the Revoker the pèossibility to revoke this autotitation.
The Contract structure is defined ad a transaciton that have the structure defined on the table below:

|Input    |Output                                                           | 
|:-----|:-------------------------------------------------------------------| 
| 0 - Provider Address    | 0 - User Address(Token) | 
| 1 - Recharge Address (opt.) | 1 - OP_RETURN |
||2 - Revoker Address(Token) |
||3 - Change Address|

The order of definition for inputs and outputs si mandatory. 
Only the recarghe address is optionally.
This kind of transaction transfer token form the Provider tp the User and the Revoker and must be signed by the provider.
The transactrion must be considered revoked when the Revoker has spent his token.
___
OP_RETURN
---------
The transaction for a smart contract has, beside the token transfer, defnied an **OP_RETURN**.
In the OP_RETURN are contained the access informations granted by the contract.
The OP_RETURN structure is fo 80 bytes so defined:

|Bytes lenght    |Description                                                           |Notes| 
|:-----|:-------------------------------------------------------------------|:---:|
|1|Version| 0 for Smart Contract V.0| 
|4|Bitmask for System Reserved functions ||
|14|Bitmask for User Defined functions ||
|1|Number of guarantors needed|Not used|
|20|Guarantor 0 address|Not used|
|20|Guarantor 1 address|Not used|
|20|Guarantor 2 address|Not used|


The **bitmask** define is a function is accessible (**1**) or denied (**0**) from the **User** to the **Provider**.

___
Identity
--------
Each Entity has an **Identity** that is determined by possession of the **private key** that is never exposed out of the entity's boundaries.<br>
This Private Key can generate the bitcoin address present in transaction.<br>
For each transaction the address must be a new one both for the Provider and the User.<br>
The addresses generation is defined trought the BIP32 standard.

___
BIP32
---
The addresses generation is base on BIP Standard.<br>
This way allow the system to generate a deterministic hierarchy of key starting from a **Seed**.<br>
This Seed is generated at the first run of the Entity and is based on a strong **RNG**.<br>
This is also the reason cause that Seed could be cosindered the **Identity of the Entity**.

Below is reported how the BIP32's hierarchy is used inside the Uniquid Framework.

* **m/44'/0'** Path for the **XPub** exported in Imprinting phase.
* **m/44'/0'/0/0** is the point from where start the hierarchy dedicated to the Provider
* **m/44'/0'/0/0/0/0** is the address where is payed the **first charge** (Imprinting). Is also the address for the **first contract like Provider** with change address set on **m/44'/0'/0/0/1/0**.
* **m/44'/0'/0/0/0/n** are the address where send the charge. UTXO on these addresses can be used as second input in a contract. 
* **m/44'/0'/0/0/1/n** are the addresses for the Change of the contracts. From these addresses are generated all conracts after the first.
* **m/44'/0'/0/1/0/n** are the addresses where the token for the User's contracts are sened.


This schema is **mandatory** like is mandatory that **no address can be appear in more than one contract**.

___
Imprinting
----------
When an Entity has created his identity, no one trnasaction on the block chain envolve his addresses.<br>
**Imprinting** is the name of the processthat allow an other Entity, called **Imprinter**, that obtain the control of the new Entity.<br>

The Imprinter take, from the new Entity, the Xpub at path **m/44'/0'**  and the **Name** used to identify the entity on the communication channel use for the message delivery.<br>
Obtained the Xpub the Imprinter send some token at address **m/44'/0'/0/0/0/0**.

This first transaction is called **Imprinting Contract** 'cause the Entity that send token acquire the access to all **system reserved functions**.

From this moment the Imprinter is the only owner of the entity imprinted.<br>
To allow a third  entity to manage the one just imprinted the imprinter send to imprinted entity a contract to  **sign**  where this third entity (called **orchestrator** ) is allowed to manage the system reserved functions. <br>

After that the Xpub and the unique name are transferred **from** imprinter **to** the orchestrator.

So, once that the first contract was signed, the **imprinting contract** went, implictly, revoked.

The orchestrator is, foornm this moment, allowed to manage the creation of the Entity's contracts.
___

Message and RPC
---
The UniquID framework define the structure of the message used to request the function's execution.<br>
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
* **result**: is the result of the function invoked.Coud be a structured JSON. The interpretation of the params value is demanded to the immplementation of the method.Errors form the method must be returned in this string.
* **error**: is number that report a message error (i.e. Invalid JSON). If the message is correct, decodified, exist a contract that allow the function execution and the function was executed then this value is 0.
* **id**: is a **nonce** that must correspond with the same parameter on request. Keep a relationship between request and response.

###Process
When the Provider Entity receive a reqeust, it must first verify that there is a contract kinking it to the sender address.
If the conract exist, it must verify if the bit correspondent to the bit correponding to the requested method on the bitmask of the contract is setted to 1 (one).
Only if this last check is ok the provider can send in execution the requested funciton.
In the reply message the sender must be valued with the address that binds provider and user in the affected contract. 
If the contract doesnot exist, the providere does not have to reply.