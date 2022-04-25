# PACER-client Windows Server 2019 Installation

Contact:<br/>
* Branch Head: Tia Pope (tia.pope@gtri.gatech.edu)<br/>
* Tech Lead: Myung Choi (myung.choi@gtri.gatech.edu)<br/>
* Project Manager: Jordan Chandler (Jordan.Chandler@gtri.gatech.edu)

## Background and Preparation: 
PACER-client uses a wrapper to run the Java application as a window service. Windows Service Wrapper (WinSW) is used for the wrapper.exe. All PACER-client components in this repository already have this wrapper application. Thus, nothing needs to be done for this wrapper. If you want to learn about the WinSW, please refer to https://github.com/winsw/winsw

### OpenJDK installation.
Java is required to run this service. Thus, either JRE or JDK needs to be installed. Go to https://docs.microsoft.com/en-us/java/openjdk/download and download OpenJDK17 msi file (microsoft-jdk-17.0.2.8.1-windows-x64.msi) to install OpenJDK. After installation, type "java --version" at the command line (or powershell) to verify its installation

#### Authorization library installation.
After installation, authorization library must be downloaded and installed in JDK bin folder. download the following dll file.

```
mssql-jdbc_auth-10.2.0.x64.dll
```

from https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-ver15

zip or tar.gz file is available from the link above. Once uncompressed, go to auth/ folder. and choose the one meets your VM configuration. 

Then, copy the file, mssql-jdbc_auth-10.2.0.x64.dll, to JDK's bin folder. If installation msi file is used for the Java installation, then the JDK bin folder should be **C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\bin**.

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

In each foler, there is an xml file. Open the XML file and make necessary changes for the environment variables. After all the environment variables are set correctly, run the executable (exe) file. Details for each application are provided below.

### PACER-INDEX-API
At Powershell (in Admin mode), go to pacer-index-api/ folder. And open pacer-index-api.xml file. Then, check the environment variables and change them as needed. JAVA_HOME should work as is if the same version of JDK in this README is used. If you are running this in the environment that security needs to be tightened, please change BASIC Auth parameters. SERVER_PORT can also be changed. Please note these variables as these will be used in another application. When everything is done, please run the followin command at the Powershell.

```
>> .\pacer-index-api.exe install
```
This will install the pacer-index-api as a service. After the installation, open 'services' application (built-in app in Windows). From the list of services, locate the PACER Index API service. Right click on it and choose Properties. There, go to 'Log On' tab and choose 'this account' option. Then, add username and password. Please note that this account should have a permission to access local harddrive, otherwise the application will have an issue writing data to PIDB.db file.

#### pacer-index-api service configuration
pacer-index.api is used by ecr-manager. In order for ecr-manager to talk to PACER-server, we need to populate the pacer-index-api with PACER-server information. From a Chome browser, go to "http://localhost:8086/pacer-index-api/1.0.0/" And, use the 'manage-api-controller' option to add the following entry. Use POST option. 

```
 {
   "providerName":"John Duke",
   "identifier":"ORDPROVIDER|P49430",
   "pacerSource":{
      "name":"PACER test",
      "serverUrl":"https://musctest.hdap.gatech.edu/JobManagementSystem/List",
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
Change the entry for each provider and PACER server. You can create one per provider. 

### ECR-MANAGER
Now, move to ecr-manager/ folder. Open ecr-manager.xml file. Example xml file is shown below.
```
<service>
  <id>ecrmanager</id>
  <name>ECR Manager</name>
  <description>This manages ECR data from lab report and EHR data</description>
  <env name="JAVA_HOME" value="C:\Program Files\Microsoft\jdk-17.0.2.8-hotspot\"/>
  <env name="JDBC_DRIVER" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
  <env name="JDBC_URL" value="jdbc:sqlserver://<host>:1433;databaseName=ecr;integratedSecurity=true"/>
  <env name="LOCAL_BULKDATA_PATH" value="C:\workspace\PACER-client-win\ecr-manager\bulkdata"/>
  <env name="LOCAL_PACER_SECURITY" value="Basic username:password"/>
  <env name="LOCAL_PACER_URL" value="http://musctest.hdap.gatech.edu:8082/JobManagementSystem/List"/>
  <env name="PACER_INDEX_SERVICE" value="http://localhost:8086/pacer-index-api/1.0.0/search"/>
  <env name="TRUST_CERT" value="true"/>
  <env name="SERVER_PORT" value="8085"/>
  <executable>java</executable>
  <arguments>-jar "%BASE%\ecr-manager-0.0.3.jar"</arguments>
  <log mode="roll"></log>
</service>
``` 
In the ecr-manager.xml, JDBC_URL must be set to the MS-SQL database where you will be storing the
PACER data. LOCAL_* environment varialbles are mostly place holders. Even though it will not be used,
please set it to correct value. LOCAL_BULKDATA_PATH needs to be pointing to existing folders. If not, 
path not available error message will be shown until the folder is creaed.

After configuring the XML file, save and run the following command,

```
>> .\ecr-manager.exe install
```

This will install the ecr-manager as a service. After the installation, open 'services' application (built-in app in Windows). From the list of services, locate the ECR Manager service. Right click on it and choose Properties. There, go to 'Log On' tab and choose 'this account' option. Then, add username and password. Please note that this account should have a permission to access the MS SQL server.

## End-to-end testing:
Run the follows to make the PACER-client to talk to PACER-server in the GTRI sandbox.

```
>> java -jar elr_sender-0.0.1-SNAPSHOT-jar-with-dependencies.jar
```

You should see the following message.

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Received response:
MSA|AA|20070701132554000008le|20220224110058.233-0500||ACK^R01^ACK|1|P|2.5.1
End of response message
```
### ELR Message
ELR testing message is provided with the following filename,
```
hl7v2msg_fhirpatient.txt
```
This file contains a testing lab file in HL7v2.5.1 format with message type = ORU^R01.
You can create your own HL7v2 message. However, it must be in v.2.5.1 and type should ORU^R01 (please see the below v2 message header for an example). 
```
MSH|^~\&| GT^1234^CLIA|Reliable^1234^CLIA|ELR^2.16.840.1.113883.19.3.2.3^ISO|SPH^2.16.840.1.113883.19.3.2^ISO|20070701132554-0400||ORU^R01^ORU_R01|20070701132554000008|P^T|2.5.1|||NE|NE|USA||||USELR1.0^^2.16.840.1.113883.19.9.7^ISO
```
There must be two additional pieces of information required in order for PACER-client to successfully trigger a request to PACER-server. They are
* provider information (in PID segment)
* patient identifier (in ORC segment)

The provider information (may be more than one provider) should be provided in advance along with PACER-server endpoint so that proper PACER indexing information can be entered at the PACER index api service. Patient identifier should be the one that can be used for queries for EHR data. The formation of patient identifier should be in system and value pair. System should tell the coding system, which includes the value.

The example of HL7v2.5.1 is shown as below.
```
MSH|^~\&| GT^1234^CLIA|Reliable^1234^CLIA|ELR^2.16.840.1.113883.19.3.2.3^ISO|SPH^2.16.840.1.113883.19.3.2^ISO|20070701132554-0400||ORU^R01^ORU_R01|20070701132554000008|P^T|2.5.1|||NE|NE|USA||||USELR1.0^^2.16.840.1.113883.19.9.7^ISO
SFT|1|Level Seven Healthcare Software, Inc.^L^^^^&2.16.840.1.113883.19.4.6^ISO^XX^^^1234|1.2|An Lab System|56734||20080817
PID|1||82713^^^FHIR&http://hl7.org/fhir&HL7^urn:hssc:musc:patientid^A&2.16.840.1.113883.19.3.2.1&ISO~777333333^^^&2.16.840.1.113883.4.1^SS^ISO||RODRICK^HEMAUER^^^^^L^^^^^^^BS|Mum^Martha^M^^^^M|19750602|M||2106-3^White^CDCREC^^^^04/24/2007|2222 Home Street^^Ann Arbor^MI^99999^USA^H||^PRN^PH^^1^555^5552004|^WPN^PH^^1^955^5551009|eng^English^ISO6392^^^^3/29/2007|M^Married^HL70002^^^^2.5.1||||||N^Not Hispanic or Latino^HL70189^^^^2.5.1||||||||N|||200808151000-0700|Reliable^2.16.840.1.113883.19.3.1^ISO
ORC|RE|1205001883|12001805860^LAB||||||20050430000000|||P49430^ATKINSON^D|CR
OBR|2|1205001883|12001805860^LAB|164200^C. trachomatis - PCA^L||20050429170100
OBX|1|ST|164200^C. trachomatis - PCA^L||Positive||Negative|A||F||200505031532
NTE|1|L|Performed At: DA|CR
NTE|2|L|LabCorp Dallas|CR
NTE|3|L|7777 Forest Lane Suite 350C|CR
NTE|4|L|Dallas, TX 752300000|CR
ORC|RE|1205001883|12001805860^LAB||||||20050430000000|||P49430^Duke^John|CR
OBR|3|1205001883|12001805860^LAB|164205^N gonorrhoeae Competition Rflx^L||20050429170100
OBX|1|ST|164205^N gonorrhoeae Competition Rflx^L||Negative||Negative|||F||20050429170100
OBX|2|ST|164212^N gonorrhoeae DNA Probe w/Rflx^L||See Reflex||Negative|||F||20050429170100
NTE|1|L|Performed At: DA|CR
NTE|2|L|LabCorp Dallas|CR
NTE|3|L|7777 Forest Lane Suite 350C|CR
NTE|4|L|Dallas, TX 752300000|CR
```

# PACER-client Update
This repository will be the place where updates are made. You can do git pull to update the PACER-client. As recommended, if a separate folder was used for the actual installation, then new updated files from the git pull will not replace the XML configuration files that were modified for the local environment. 

New updated jar file(s) can be run by restarting the service from 'services'. Do not copy the XML file over to the folder that will be used for the actual deployment. This can overwrite the current settings.
