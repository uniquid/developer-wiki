UniquID Architecture
====================

Introduction
------------
The UniquID framework defines the concepts of **Entity** and **Contract**.

An Entity is a software component designed to be embeddedable inside an application that offer an advanced and customizable framework for authentication. It is composed by:

* cryptographic ID: the unique identity - is a mechanism that uses ECDSA to digitally sign data. Keys are used to sign Contracts (and generally any digest) and verify digital signatures. This component should use a secure element to manage securely the cryptographic keys.
* contract exchanger and validator: a component that allow to exchange Contracts between Entities and validate their content.
* contract repository: a local data store for contracts that allow easy contract lookup/manipulation to the Entity.
* communication helper: the interface of the Entity with the external world. It allows to expose Entity functionalities to user code such as signing, signature verification and contract verification

This is a representation of Entity building blocks:

```

Uniquid Entity

 ----------------------------------------------------------------------------------------------------
|                                                                                                    |
|                                      communication helper                                          |
|                                                                                                    |
 ----------------------------------------------------------------------------------------------------
|                                             |                            |                         |
|       contract exchanger and validator      |     contract repository    |    cryptographic ID     |
|         (blockchain access layer)           |     (contract register)    | (bip32 secure element)  |
|                                             |                            |                         |
 --------------------------------------------- ---------------------------- ------------------------- 
```

Entities interacts with each other in order to perform authentication on behalf of the user. Entities use data present in Contracts to enforce security policies.


A Contract represents an agreement between two Entities and defines how they can interact. The contract contains the public identities of the parties involved in the authentication mechanism and a payload. The payload contains what the provider application code needs to enforce.

This is a representation of a Contract:

```
 -------------------------------------------------------
| Provider public identity                              |
| ------------------------------------------------------|
| User public identity                                  |
|-------------------------------------------------------|
| Revoker public identity                               |
|-------------------------------------------------------|
|                                                       |
| Payload                                               |
|                                                       |
 -------------------------------------------------------|
```

The BlockChain is used to transport and validate **Contract** between the Entities in a trustless manner.

An Entity can embody two functionalities: if the Entity performs a request then it's called **User**; if the Entity receives a requests it's called **Provider**.

Every Entity intrinsically has always the provider functionality in order to allow the signing of Contracts from the UniquID framework.

Another special Entity is the **Revoker**. It's the only party authorized to destroy the contract.


Functions
---------

To allow the UniquID framework to create and distribute contracts a special kind of payload and provider functionality is needed. The Contract carrying this payload permit the Provider to sign new Contracts and broadcast them on the BlockChain only if the User is granted this priviledge.

The special payload defines a bitmask: each bit of the mask represents the n-th function. If the bit is 1 then the function is granted otherwise the function is denied.

The functions are identified by a number in range between 0 and 143 and are divided into two separated groups:

* [**System Reserved**](../Documents/systemreserved.md):  range   **[0,31]** are reserved for framework managment
* **User Defined** : range **[32,143]** can be used users of the library to implement business applications

The default provider functionality implementation is similar to an **RPC** mechanism: the Provider receives a request from the User asking to perform a specified function. The Provider will check if there is a Contract that link it to the User, and then checks the bitmask in the payload to verify that the function is enabled.

The default library, hence, has a new representation: the Function layer defines a series of functions that can be plugged in automatically and must be exposed outside by the user code.

```
 ------------------------
|                        |
|    Function layer      |
|                        |
 ----------------------------------------------------------------------------------------------------
|                                                                                                    |
|                                      communication helper                                          |
|                                                                                                    |
 ----------------------------------------------------------------------------------------------------
|                                             |                            |                         |
|       contract exchanger and validator      |     contract repository    |    cryptographic ID     |
|         (blockchain access layer)           |     (contract register)    | (bip32 secure element)  |
|                                             |                            |                         |
 --------------------------------------------- ---------------------------- ------------------------- 
```

Please note that this new block on top of the communication helper is not a core part UniquID library, but it is needed to implement the overall framework functionality.
For simple implementations the library user is encuraged to just implement this layer. This is why UniquID ships this layer as part of the library.

As a consequence every Entity always has the Provider functionality for the System Reserved functions to allow the framework to work correctly.


Imprinting
----------

When an Entity 'A' borns, it creates its own cryptographic identity. Since there isn't yet any Contract that link any other Entity with 'A', no one is allowed to interact with 'A'.
**Imprinting** is the name of the process that allows another Entity 'B', called **Imprinter**, to enroll 'A' identity into the system.

The Imprinter receives from the Entity 'A' the public part of the cryptographic identity. It then save it into its persistence and creates a special Contract that link itself (the Imprinter) and the Entity 'A' and publish it on the BlockChain. This contract allows the Imprinter to delegate 'A' to another Entity 'C'.

This special Contract is received by the Entity 'A' from the BlockChain, is verified and finally stored in the contract repository.

From this moment, the Imprinter is the only authorized User of 'A' that can ask 'A' to create a management contract with 'C'.


Orchestration
-------------

**Orchestration** is the name of the process that allows the Imprinter to delegate the management of Entity 'A' to another Entity 'B' called **Orchestrator**.

The Imprinter creates a Contract where 'A' is the Provider and 'B' is the User and then send it to the Entity 'A' to be signed and broadcasted on the BlockChain.

This Contract sets to 1 the bits that allows the User to request Contract creations.

When this Contract is received by the Entity 'A' from the BlockChain, it is verified and finally stored in the contract repository.

When this Contract is received by the Orchestrator from the BlockChain, it is verified and stored in the contract repository. Then the Orchestrator contacts the Imprinter requesting the public cryptographic identity of 'A'.

The Orchestrator is, from this moment, allowed to manage the creation of the A's contracts.
