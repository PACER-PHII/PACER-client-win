# PACER-client Windows Server 2019 Installation

Contact:<br/>
* Branch Head: Tia Pope (tia.pope@gtri.gatech.edu)<br/>
* Tech Lead: Myung Choi (myung.choi@gtri.gatech.edu)<br/>
* Project Manager: Jordan Chandler (Jordan.Chandler@gtri.gatech.edu)

## Background and Preparation: 
PACER-client uses a wrapper to run the Java application as a window service. Windows Service Wrapper (WinSW) is used for the wrapper.exe. All PACER-client components in this repository already have this wrapper application. Thus, nothing needs to be done for this wrapper. If you want to learn about the WinSW, please refer to https://github.com/winsw/winsw

### OpenJDK installation.
Java is required to run this service. Thus, either JRE or JDK needs to be installed. Go to https://docs.microsoft.com/en-us/java/openjdk/download and download OpenJDK17 msi file to install OpenJDK. After installation, type "java --version" at the command line (or powershell) to verify its installation

### MS Sql Server Database.
Any relational database can be used. If you prefer another database such as PostgreSQL, refer to https://www.postgresql.org/download/windows/ and download the installer to install PostgreSQL database. 

ECR Manager needs to have a database to persist ECR data from electronic lab reports and EHR. Whether you use MS SQL or PostgreSQL, "ecr" must be used for the name of **database** and **schema**. Tables will automatically be created. 

In order to use Windows Authentication for MS SQL, make sure "ecr" database is owned by the account that will run PACER-client or the account has a db writer/reader permission. Please note that the schema name also needs to be "ecr"

## PACER-client deployment
There are three folders in the PACER-client-win repository. It is recommeded to create a separate folder to copy the following three folders. In this way, when updates are made, the original folder can keep the updated version. And, copied version can be modified for the local environment. 

The applications must be deployed or started in the following order
   - pacer-index-api
   - ecr-manager
   - elr-receiver

In each foler, there is an xml file. Open the XML file and make necessary changes for the environment variables. After all the environment variables are set correctly, run the executable (exe) file. See below for pacer-index-api.exe as an example,

```
>> .\pacer-index-api.exe install
```
This will install the pacer-index-api as a service. After the installation, open 'services' application (built-in app in Windows). From the list of services, locate the PACER Index API service. Right click on it and choose Properties. There, go to 'Log On' tab and choose 'this account' option. Then, add username and password.

Repeat the above for ecr-manager and elr-receiver. 'Log On' is critical for the ecr-manager because the ecr-manager will use this account to talk to MS SQL server in the windows authentication mode.

### pacer-index-api service deployment
In order for ecr-manager to talk to PACER-server, we need to populate the pacer-index-api with PACER-server information. From a Chome browser, go to "http://localhost:8086/pacer-index-api/1.0.0/" And, use the 'manage-api-controller' option to add the following entry. Use POST option. 

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
         "password":"<password of list manager in the PACER server>"
      },
      "version":"1.0.0",
      "type":"ECR"
   }
}
```
Change the entries for each provider and PACER server.

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

# PACER-client Update
This repository will be the place where updates are made. You can do git pull to update the PACER-client. As recommended, if a separate folder was used for the actual installation, then new updated files from the git pull will not replace the XML configuration files that were modified for the local environment. 

New updated jar file(s) can be run by restarting the service from 'services'. Do not copy the XML file over to the folder that will be used for the actual deployment. This can overwrite the current settings.
