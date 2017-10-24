## LibraryDesign

ThepresentsectionisactuallyfocusedonJavalibrary.Whenotherplatformbecomeavailablethediferrencebetweenplatformwenthiglighted.

TheUniquidLibraryisdividedinthreecomponents:

* **Alfred**
* **Node**
* **Register**

Belowisreportedagraphicschemathatillustratethelibrary'slogicalareas.

![](blob:https://www.gitbook.com/b4adbda9-6eea-46c5-a342-867098494a47)

### Alfred

As a good butler, it takes care of accessing and communicating various protocols \(MQTT, HTTP, etc...\) with the library.  
  
Alfred offers the ProviderCore interface for message exchange with library and additional functionality management.  
  
Before making a request, verify that the sender has permission to do so. Allows you to instantiate:

* Request and Response interfaces
* Additional functionality management interfaces

ProviderRequestprovidesmethodsforarequest.Similarly,ProviderResponseprovidesmethodsforrespondingtoareceivedrequest.

### Register

The Register is the component to which device data management and persistence is demanded.  
  
It is independent of the type of storage you choose \(SQLite, text files, etc...\).  
  
Provides many interfaces for handling and persist of storage data.  


* RegisterFactory
* ProviderRegister
* UserRegister
* TransactionManager

Italsoprovide2beans:

* ProviderChannel
* UserChannel

### Node

It's the interface from the library that implements the SPV Node handling key creation and usage, communication with the blockchain, and all those features of Wallet SPV.  
  
The interface provide the layers with the Wallet library chosen by the user \(BitcoinJ, BitcoinSPV, etc...\).  
It has an UniquidNode interface for managing Wallet features, from communicating with the Blockchain to receiving contracts.  
  
The UniquidNode also offer a callback mechanism \(UniquidNodeEventListener\) to trigger the arrival of user/provider contracts, whether they are created or revoked.  
  
To get information about a contract, there are the ProviderChannel and UserChannel classes that provides the methods that you need.

TheNodeisidentifiedbyastate:

* Created
* Imprinting
* Ready



