# PACER-client Windows Server 2019 Installation

Contact:<br/>
* Branch Head: Tia Pope (tia.pope@gtri.gatech.edu)<br/>
* Tech Lead: Myung Choi (myung.choi@gtri.gatech.edu)<br/>
* Project Manager: Jordan Chandler (Jordan.Chandler@gtri.gatech.edu)

## Preparation: 
PACER-client uses a wrapper to run the Java application as a window service. Windows Service Wrapper (WinSW) is used for the wrapper.exe. For more information about the WinSW, please refer to https://github.com/winsw/winsw

### MS Sql Server Database.
Any relational database can be used. If you prefer another database such as PostgreSQL, refer to https://www.postgresql.org/download/windows/ and download the installer to install PostgreSQL database. 

ECR Manager needs to have a database to persist ECR data from electronic lab reports and EHR data. Whether you use MS SQL or PostgreSQL, "ecr" must be used for the name of **database** and **schema**. Tables will automatically be created. 

In order to use Windows Authentication for MS SQL, make sure "ecr" database has the account that PACER-client will be running under as a db writer/reader/owner. Please note that the schema name also needs to be "ecr"

### OpenJDK installation.
Go to https://docs.microsoft.com/en-us/java/openjdk/download and download OpenJDK17 msi file
to install OpenJDK. After installation, type java --version to verify its installation

## PACER-client deployment
There are three folders. Those are the components for PACER client. The components must be deployed in the following order
   - pacer-index-api
   - ecr-manager
   - elr-receiver

In each foler, there is an xml file. Open the XML file and make necessary changes for the environment variables. After all the environment variables are set correctly, run the executable file. See below for pacer-index-api.exe example,

```
>> .\pacer-index-api.exe install
```
This will install the pacer-index-api as a service. Then, open services application (built-in app in Windows). From the list of services, locate the PACER Index API service. Right click on it and choose Properties. There, go to 'Log On' tab and choose 'this account' option. Then, add username and password.

Repeat this for ecr-manager and elr-receiver. This is critical for the ecr-manager as the ecr-manager will use this account to talk to MS SQL server with windows authentication mode.

### pacer-index-api service deployment
We need to populate the PACER index information for ECR Manager. From a Chome browser, go to "http://localhost:8086/pacer-index-api/1.0.0/" And, use manage-api-controller to add the following entry. Use POST option. 

```
 {
   "providerName":"John Duke",
   "identifier":"ORDPROVIDER|P49430",
   "pacerSource":{
      "name":"PACER test",
      "serverUrl":"http://musctest.hdap.gatech.edu:8082/JobManagementSystem/List",
      "security":{
         "type":"basic",
         "username":"<username of list manager in the PACER server>",
         "password":"<password of list manager in the PACER server>>"
      },
      "version":"1.0.0",
      "type":"ECR"
   }
}
```

## End-to-end testing:
Run the follows to make the PACER-client to talk to PACER-server in the GTRI sandbox.

1. .\env_config.bat
2. java -jar elr_sender-0.0.1-SNAPSHOT-jar-with-dependencies.jar

You should see the following message.

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Received response:
MSA|AA|20070701132554000008le|20220224110058.233-0500||ACK^R01^ACK|1|P|2.5.1
End of response message
```

