# Introduction

This demo was created to review some general features of JBoss Fuse 6.2.1 <br/>
It was created to help new fuse developers to understand a full development cycle of a project including using it on a HA enviorment. <br/><br/>
ENJOY!!!!

# Demo Architecture (Read)

What this demo does is to provide a client that send sql insert commands to activemq brokers. We will also create a camel rout to read messages from broker and insert the dato to a postgrsql database.

## Objectives

This demo will include information about several topics wich include: 

* Create a JBoss Fuse installation and initial configuration.
* Create a AMQ installation and initial configuration
* Access Web console to manage Fuse and AMQ
* Learn how a Spring client sent messages to a Broker
* Learn how a camel rout on Fuse recieve those messages and insert them to a database.
* Learn how to deploy projects profiles using maven plugins
* Review how camel read data from AMQ JMS Brokers and how to ejecute commands on a Postgresql database


## Pre-Requisites

1. JBoss Fuse 7.8 jar installation file 
2. JBoss AMQ  7.8.1 zip of tar.gz file
3. Java JDK 11 or 15 installed
4. Apache Maven 3.1.1 version installed
5. Web Browser
6. Basic Linux commands understanding
7. SOAP UI or another web service client
8. PostgreSQL 9.3 or superior installed
9. PostgreSQL administration understanding (Create users, databases and running scripts)
10. Internet connection
11. GIT installed

# Setup JBoss Fuse

## Install JBoss EAP 7.3

1. Download jboss-eap-7.3.0.zip file from Red Hat Customer Portal or Developer Portal.

2. Download jboss-eap-7.3.5-patch.zip patch (or above) from Red Hat Customer Portal or Developer Portal.

3. Open a command Terminal

4. Unzip jboss-eap-7.3.0.zip:
   - `unzip jboss-eap-7.3.0.zip`<br/>

5. Copy jboss-eap-7.3.5-patch.zip to new unziped folder:
   - `cp /path/to/jboss-eap-7.3.5-patch.zip /path/to/jboss/jboss-eap-7.3/`

6. Apply the patch:
   - `<PathToJBossFolder>/bin/jboss-cli.sh "patch apply jboss-eap-7.3.x-patch.zip"`

7. Add an admin user to JBoss EAP:
   - `<PathToJBossFolder>/bin/add-user.sh`
   - Select Management User
   - Write a username
   - Write a password and confirm the password
   - Leave blank the groups question (just press enter)
   - Confirm the creation question
   - Write 'No' to "Is this new user going to be used for one AS process to connect to another AS process?" question.

## Install JBoss Fuse

1. Download fuse-eap-installer-7.8.0.jar file from Customer Portal of Developer Portal.

2. Open a command terminal

3. Copy fuse-eap-installer-7.8.0.jar to jboss folder installed on the preview step
   - `cp /path/to/fuse-eap-installer-7.8.0.jar /<PathToJBossFolder>/jboss/jboss-eap-7.3/`

4. Run the installer jar file:
   - `java -jar <PathToJBossFolder>/fuse-eap-installer-7.8.0.jar`

Thats it!!!, JBoss Fuse is already install!!!
 

## Running JBoss Fuse

1. On opened terminal and start JBoss EAP: `<PathToJBossFolder>/bin/standalone.sh`

2. Access Fuse console using a web browser (http://localhost:8080/hawtio):

3. Access the console using the user and password created when JBoss EAP was installed

## Installing AMQ

1. Download amq-broker-7.8.1-bin.zip file from Customer Portal of Developer Portal.

2. Unzip the file an a folder: `unzip amq-broker-7.8.1-bin.zip`.

3. Create a new AMQ broker:
	- `cd /path/to/unzipped/amqfolder`
	- `./bin/artemis create`
	- Give a name 'amq01' to the new broker and press 'Enter'.
	- When prompt, add a userename 'admin' for the new admin user and press 'Enter'.
	- Write the password 'admin' for the new admin user and press 'Enter'

## Running JBoss AMQ

1. Start the created AMQ borker by running:
	- `cd /path/to/unzipped/amqfolder/nameofcreatedbroker/`
	- `./bin/artemis run`

2. Access the AMQ admin portal at http://localhost:8161

# Configure Camel project

There are two projects:
 * hainserter: Fuse route that reads sql commands and execute them on the real JDBC driver
 * spring-db-populator: Batch testing client that insert data using spring

1. Create PostgreSQL database:
	- Create a Postgresql user called **fusedemo** with password **12345678**
	- Create database **jdbcpoc** and assign it to **fusedemo** user 
	- Create table and sequence with provided script at `/<projects_dir>/spring-db-populator/src/main/resources/DATABASE_SCRIPT.sql`

2. Install ha-inserter to Fuse. The next steps will deploy a new **camel-jdbcpoc** route into fuse. For more information about how this is done check Readme.md file inside ha-inserter project.
	- Edit file `/<projects_dir>/hainserter/src/main/resources/META-INF/spring/jboss-camel-context.xml` and set datasource properties and activemq properties. (No need if using demo defaults and postgresql fusedemo user)
	- cd <projects_dir>/ha-inserter/
	- Execute: `mvn clean package wildfly:deploy`. This will compile and deploy fuse projet into a fuse server.
    - Wait for build success
    - Wait for success deployment. You can see the new created profile using web console. 

# Testing Inserts

6. Compile and run spring-db-populator client
	- `cd <projects_dir>/spring-db-populator/`
	- Edit file `./src/main/resources/jdbc-spring-context.xml` and change beans activemq and activemq2 to point at both brokers groups ports 
	- `mvn clean package`
	- `java -jar ./target/spring-db-populator-0.0.1-SNAPSHOT.jar 1 100 5` <br/> 
	Note: On the client the parameters are 1 = start index, 100 = how many inserts to execute, 5 total parallel threads.
    
    This brokers now are listening at two master/slave groups. jdbcpoc-broker1 is listening at messages arriving at JMS Brokers 61616 port.<br/>
    Run the client and check how fast its inserting data arriving from client.
    The client will insert using multithreading to both JMS ports.
        
Thats all folks

    
	


