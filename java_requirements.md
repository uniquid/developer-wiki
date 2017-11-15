Java Core Library Rquirements
==========

In order to develop with Uniquid technologies in Java you need:

* JDK >= 1.7
* git
* maven >= 3.0.0

To install  properly all dipendencies and environment please provide the follow sateps:

* Pull uniquid's bitcoinj fork via git from https://github.com/uniquid/bitcoinj and sync with the branch 
```
'release-0.14-uniquid'
```

* Perform a full build and install artifacts with maven: 
```
mvn clean package install
```

If you have compiling problems related to javafx, comment the  
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
'master'
```
* perform a full build and install artifacts with maven: mvn clean package install
* now you have all pre-requirements: uniquid's bitcoinj, uidcore-java and uniquid-utils libraries ready to be used in your applications: you can start to create an Uniquid Node
