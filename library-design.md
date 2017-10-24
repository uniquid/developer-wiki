Library Design
---

The present section is actually focused on Java library. When other platform become available the diferrence between platform went higlighted.

The Uniquid Library is divided in three components:

* **Alfred**
* **Node**
* **Register**

Below is reported a graphic schema  that illustrate the library's logical areas.

![alt text](../img/C5tschema_cut.png "Library Schema")

### Alfred
As a good butler, it takes care of accessing and communicating various protocols (MQTT, HTTP, etc...) with the library.<br>
Alfred offers the ProviderCore interface for message exchange with library and additional functionality management.<br>
Before making a request, verify that the sender has permission to do so. Allows you to instantiate:

* Request and Response interfaces
* Additional functionality management interfaces
    
ProviderRequest provides methods for a request. Similarly, ProviderResponse provides methods for responding to a received request.

### Register
The Register is the component to which device data management and persistence is demanded.<br> 
It is independent of the type of storage you choose (SQLite, text files, etc...).<br>
Provides many interfaces for handling and persist of storage data.<br> 

* RegisterFactory
* ProviderRegister
* UserRegister
* TransactionManager

It also provide 2 beans:

* ProviderChannel
* UserChannel

### Node
It's the interface from the library that implements the SPV Node handling key creation and usage, communication with the blockchain, and all those features of Wallet SPV.<br>
The interface provide the layers with the Wallet library chosen by the user (BitcoinJ, BitcoinSPV, etc...).
It has an UniquidNode interface for managing Wallet features, from communicating with the Blockchain to receiving contracts.<br> 
The UniquidNode also offer a callback mechanism (UniquidNodeEventListener) to trigger the arrival of user/provider contracts, whether they are created or revoked.<br>
To get information about a contract, there are the ProviderChannel and UserChannel classes that provides the methods that you need.

The Node is identified by a state:

* Created
* Imprinting
* Ready
___
