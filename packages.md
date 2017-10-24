# Packages List

Below are listed all packages that you can use to experiment the UniquID Blast platform.  
This page is coinitnous improvment.   
Keep in mind.

### Massive Imprinter Dashboard \[[zip](../attachments/massive-imprinter-dashboard.zip) - [tar](../attachments/massive_imprinter_dashboard.tar.gz)\]

Is a Node.js app that provide a GUI to monitor your massive imprinter backend.  
Requirment : node.js and npm installed  
To run:

* Explode the archive in the directory that you want use 
* Edit the file config.js and specify the address of your Massive Imprinter Backend
* Execute:  
       npm install

* To launch :

  ```
   npm start
  ```

You can use with screen, forever or what you prefer to keep running the process.

### Massive Imprinter Backend \[[zip](../attachments/massive_imprinter_backend.zip) - [tar](../attachments/massive_imprinter_backend.tar.gz)\]

Is a jar file that you can doanload and start ni any environment provided of a JVM.   
The

* Explode the archive in any directory that you want
* Edit the appconfig.properties files and replace

  ```
    [path where the massive imprinter is installed]
  ```

  with the path where the massive imprinter is installed.

* Edit the **mqtt.broker** property in the appconfig.properties file and replace

  ```
    [select a public broker like tcp://broker.hivemq.com:1883]
  ```

  with the path of a public broker or any other MQTT Broker reachable from the machine where you have installed the Massive Imprinter.

* Make sure that **start.sh** and **clean.sh** scripts are executable with **chmod +x start.sh** and **chmod +x clean.sh**

* To start the massive imprinter you can launch the script **./start.sh**  You can use it with screen or what you prefer to keep running the process.

#### Important Notice

The first time that you launch the massive imprinter an UniquID identiti is created.

You need to be charged by some token.

To do that you must send your tpub to UniquID \(stay tuned great imrovment for this process are in development\).

To retrieve the tpub for your massive imprinter identity you must go with a browser to the address:

```
    http://[address of your massive imprinter]:8080/status.jsp
```

then scroll the page until you can see a page like this:

![alt text](../img/tpub_example.png "TPUB Example")

For that you can contact your direct reference on UniquID team.

