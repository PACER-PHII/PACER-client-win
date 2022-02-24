# PACER-client Windows Server 2019 Installation

## Preparation: 
PACER-client uses Docker to deploy some of its components. 
You need to install Docker for Windows. Please refer to the following
two linkx for more information about the Docker installation for Windows

https://blog.sixeyed.com/getting-started-with-docker-on-windows-server-2019/
https://blog.foldersecurityviewer.com/how-to-install-docker-and-run-docker-containers-on-windows-server-2019/

### PostgreSQL database installation.
Go to https://www.postgresql.org/download/windows/ and download the installer to install
PostgreSQL database. 

1. Once the PostgreSQL is installed, launch paAdmin 4 from windows start section. This is GUI management
tool for PostgreSQL. 
2. On the left panel, right click on PostgreSQL 14 icon to create a new database called "ecr".
Then, right click on the "ecr" database and create a schema called "ecr". 
3. As a default setting, the windows server 2019 has a local firewall turned on. We need to add a
inbound rule for the postgreSQL server port. Use Server Manager dashboard and run Windows Defender Firewall
from Tool menu.

The PostgreSQL server itself listens to local traffic. So, go to the following file and add entry for host.
C:\Program Files\PostgreSQL\14\data\pg_hba.conf

Under IPv4 local connections section, add a new entry as follow,

```host    all             all             xxx.xxx.xxx.xxx/32            scram-sha-256```

xxx.xxx.xxx.xxx is your IP address of Windows Server 2019 VM.

After you save and exit this file, please restart the PostgreSQL service from services.

### OpenJDK installation.
Go to https://docs.microsoft.com/en-us/java/openjdk/download and download OpenJDK17 msi file
to install OpenJDK. After installation, type java --version to verify its installation

## PACER-client deployment
1. Run “Start-Service Docker” if you haven’t started docker
2. Create a network within docker. 
   Run "docker network ls" to see if you already have a network called "pacer". 
   If it does not exist, run the following to create "pacer" network
	docker network create --driver nat pacer

### pacer-index-api service deployment
1. docker pull artifactory.icl.gtri.org:443/pacer-platform/pacer_index_api
2. docker run --name pacer_index_api -p 8086:8080 --env-file env_pacer_index_api --network pacer -d artifactory.icl.gtri.org:443/pacer-platform/pacer_index_api:latest
From Chome browser, go to "http://localhost:8086/pacer-index-api/1.0.0/" And, use manage-api-controller to add
the following entry. Use POST option. Username and Password are specified in "env_packer_index_api" file.

 {
   "providerName":"John Duke",
   "identifier":"ORDPROVIDER|P49430",
   "pacerSource":{
      "name":"PACER test",
      "serverUrl":"http://musctest.hdap.gatech.edu:8082/JobManagementSystem/List",
      "security":{
         "type":"basic",
         "username":"username",
         "password":"password"
      },
      "version":"1.0.0",
      "type":"ECR"
   }
}

### elr_receiver deployment
1. docker pull artifactory.icl.gtri.org/pacer-platform/elr_receiver
2. docker run --name elr_receiver -p 8087:8888 --env-file env_elr_receiver --network pacer -d elr_receiver:latest

### ecr-manager deployment
1. Open "env_ecr_manager" file and put your windows server VM host name (or IP) in [your win server host] part.
2. docker pull artifactory.icl.gtri.org/pacer-platform/ecr_manager
3. docker run --name ecr_manager -p 8085:8080 --env-file env_ecr_manager --network pacer -d ecr_manager:latest

Now run "docker ps" to see if all three components are running. Go to pgAdmin 4 and check tables under "ecr" schema.
You should see "ecr_data" and "ecr_job" tables.

## End-to-end testing:
Run the follows to make the PACER-client to talk to PACER-server in the GTRI sandbox.

1. .\env_config.bat
2. java -jar elr_sender-0.0.1-SNAPSHOT-jar-with-dependencies.jar

You should see the following message.

```SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Received response:
MSA|AA|20070701132554000008le|20220224110058.233-0500||ACK^R01^ACK|1|P|2.5.1
End of response message```

