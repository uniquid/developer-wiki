UniquID Library a quick tutorial
====================

This tutorial want give a quick knowledge on the UniquID technologies with particoular focus on the UICore Library throught building an example application from scratch.

## Setup Environment

Before start to write our application a proper setup is mandatory.
The minimum requirment are: 

* JDK 1.7 or above
* Maven 3.0 or above

Setup steps:
* Pull uniquid's bitcoinj fork via git from https://github.com/uniquid/bitcoinj and sync with the branch 
```
'release-0.14-uniquid'
```

* Perform a full build and install artifacts with maven: 
```
mvn clean package install
```

* If you have compiling problems related to javafx, comment the  
```
<module>wallettemplate</module>
```
 inside the pom.xml

* now you have the bitcoinj library improved with uniquid's enhancements
* pull uniquid's uniquid-utils project via git from https://github.com/uniquid/uniquid-utils and sync with branch 
```
'master'
```
* perform a full build and install artifacts with maven: 
```
mvn clean package install
```
* pull uniquid's uidcore-java project via git from https://github.com/uniquid/uidcore-java and sync with branch:
```
'develop_refactory_state'
```
* perform a full build and install artifacts with maven: mvn clean package install
* now you have all pre-requirements: uniquid's bitcoinj, uidcore-java and uniquid-utils libraries ready to be used in your applications: you can start to create an Uniquid Node

## Source Code
The complete cose described in this tutorial is available on https://github.com/uniquid/tank-java.<br/>
To perform and run build : 
```
mvn clean package install
```


## Case Study
Immaginiamo che il nostro migliore amico abbia la necessità di poter gestire in modo sicuro la sua preziosa scorta di birra.<br/>
Vuole essere sicuro che solo lui possa riempire ma sopratutto bere dal suo personalissimo barile.<br/>
Fino ad oggi l'unico modo che aveva era portarla sempre con se.

![alt text](../img/barney_in.jpg "Barney in")



## Basic Steps

The java uidcore library is composed by many modules: connector, core, node and register.

The setup and configuration process can be a little tricky so we developed the UniquidSimplifier that is a wrapper that configure the library using standard configuration (mqtt, default uniquid functions, etc).

However some basic configuration is still needed in order to create a working node.

This steps are:

* Instantiate a **RegisterFactory** concrete implementation (the module that manage contract persistence)<br/>
* Instantiate a **UniquidNode** concrete implementation<br/>
* Instantiate a **Connector** concrete implementation <br/>
* Instantiate an **UniquidSimplifier** class that wraps the RegisterFactory, Connector and UniquidNode instances previously created<br/>
* We can register custom functionalities on available user level slots<br/>
* Execute the **start()** method on UniquidSimplifier to start the library: it will start the synchronization of the node on the blockchain, it will start to receive messages on mqtt and it will use the registerfactory to interact with the persistence layer<br/>



