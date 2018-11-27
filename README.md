# camel-amq-samples
Camel Routes to manage messaging systems (Active-MQ)

These projects show you some route samples to manage JMS messages using the following capabilities:

__Red Hat JBoss Fuse 6.2.1__:
* Install Red Hat JBoss Fuse 6.2.1
* How to create a Fuse Fabric
* How to create different A-MQ Brokers and their containers
* How to create Gateway Containers
* How to create Service Containers

__Camel Context and Camel Routes__:
* How to use ActiveMQComponent in a Camel Context
* Sending (Producer) messages to a Queue in a Camel Route
* Reading (Consumer) messages from a Queue in a Camel Route
* Forward messages from a Messages Broker to another
* Use discovery/fabric capabilities to connect to a Message Broker
* Deploy bundles as profiles in a Fuse Fabric environment

# Project Structure
This project has the following structure

* __camel-amq-parent__:
* __camel-amq-producer__:
* __camel-amq-consumer__:
* __camel-amq-forwarder__:

# Install Red Hat JBoss Fuse

## Download
Download [Red Hat JBoss Fuse 6.2.1][1] binaries (subscription needed) from Red Hat Customer Portal

## Install
To install Red Hat JBoss Fuse follow the next instructions:

* Unzip .zip file into a specific folder
* Start Fuse with _bin/fuse_ or _bin/start_ shell scripts (.bat in Windows OS cases) 

	[root@rhel7jboss01 redhat]# unzip jboss-fuse-full-6.2.1.redhat-159.zip
	[root@rhel7jboss01 redhat]# cd jboss-fuse-6.2.1.redhat-159
	[root@rhel7jboss01 jboss-fuse]# ./bin/fuse
	Please wait while JBoss Fuse is loading...
	100% [========================================================================]
	  JBoss Fuse (6.2.1.redhat-159)
	  http://www.redhat.com/products/jbossenterprisemiddleware/fuse/
	
	Hit '<tab>' for a list of available commands
	and '[cmd] --help' for help on a specific command.
	
	Open a browser to http://localhost:8181 to access the management console
	
	Create a new Fabric via 'fabric:create'
	or join an existing Fabric via 'fabric:join [someUrls]'
	
	Hit '<ctrl-d>' or 'osgi:shutdown' to shutdown JBoss Fuse.
	
	No user found in etc/users.properties. Please use the 'esb:create-admin-user'
	command to create one.
	JBossFuse:karaf@root> 
	
## Create Fuse Fabric
_fabric:create_ is the Karaf command to create and define a new Fabric environment:

	JBossFuse:karaf@root> fabric:create --clean --bind-address 127.0.0.1 --new-user admin --new-user-password admin --zookeeper-password zookeeper --resolver localhostname --wait-for-provisioning 
	Waiting for container: root
	Waiting for container root to provision.
	JBossFuse:karaf@root> status
	[profile]                                [instances]    [health]
	fabric                                   1              100%
	fabric-ensemble-0000-1                   1              100%
	jboss-fuse-full                          1              100%

The _ensemble-list_ and _container-list_ commands will show us the status of the Fabric after its creation:

	JBossFuse:karaf@root> ensemble-list
	[id]
	root
	JBossFuse:karaf@root> container-list
	[id]   [version]  [type]  [connected]  [profiles]              [provision status]
	root*  1.0        karaf   yes          fabric                  success
	                                       fabric-ensemble-0000-1
	                                       jboss-fuse-full

To reduce the resources used by the Fabric container, it is a good practice to remove the _jboss-fuse-full_ profile in Fabric containers:

	JBossFuse:karaf@root> container-remove-profile root jboss-fuse-full 

## Create MQ-Broker
We will create two Message Brokers:

* __amq-broker-one__: Broker to send and to receive messages. This broker will be used by _camel-amq-producer_ and _camel-amq-consumer_ projects
* __amq-broker-two__: Broker to receive messages from the previous broker. This broker will be used by _camel_amq_forwarder_ project

	JBossFuse:karaf@root> mq-create --group demo-one --kind StandAlone --create-container amq-broker-one amq-broker-one
	MQ profile mq-broker-demo-one.amq-broker-one ready
	JBossFuse:karaf@root> mq-create --group demo-two --kind StandAlone --create-container amq-broker-two amq-broker-two
	MQ profile mq-broker-demo-two.amq-broker-two ready

We can review the status with the _container-list_ command: 

	JBossFuse:karaf@root> container-list
    [id]                     [version]  [type]  [connected]  [profiles]                         [provision status]
    root*                    1.0        karaf   yes          fabric                             success           
                                                             fabric-ensemble-0000-1                               
      amq-broker-one         1.0        karaf   yes          mq-broker-demo-one.amq-broker-one  success           
      amq-broker-two         1.0        karaf   yes          mq-broker-demo-two.amq-broker-two  success           

If you review using de Fuse Management Web Console, the screenshot will be similar to:

![Fuse Fabric Service MQ](./img/Fuse-Fabric-Services-MQ.png "Services - MQ")

[More Information about MQ Brokers][2]

## Create Gateway Containers
The _Gateway Containers_ will work as the load balancers to access to the different services working on Fuse Fabric.

	JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper --profile gateway-mq root gateway
	The following containers have been created successfully:
		Container: fuse01-gw.
	JBossFuse:karaf@root> container-list
    [id]                     [version]  [type]  [connected]  [profiles]                         [provision status]
    root*                    1.0        karaf   yes          fabric                             success           
                                                             fabric-ensemble-0000-1                               
      amq-broker-one         1.0        karaf   yes          mq-broker-demo-one.amq-broker-one  success           
      amq-broker-two         1.0        karaf   yes          mq-broker-demo-two.amq-broker-two  success           
      gateway                1.0        karaf   yes          gateway-mq                         success           

[More Information about Gateway][3]

## Create Service Containers
Service Containers will be used to deploy the Camel Contexts and Camel Routes to manage the messages:

	JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper root esb-producer
	The following containers have been created successfully:
		Container: esb-producer.
	JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper root esb-consumer
	The following containers have been created successfully:
		Container: esb-consumer.
	JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper root esb-forwarder
	The following containers have been created successfully:
		Container: esb-forwarder.

After some minutes we will have the three containers ready to be used:

	JBossFuse:karaf@root> container-list
    [id]                     [version]  [type]  [connected]  [profiles]                         [provision status]
    root*                    1.0        karaf   yes          fabric                             success           
                                                             fabric-ensemble-0000-1                               
      amq-broker-one         1.0        karaf   yes          mq-broker-demo-one.amq-broker-one  success           
      amq-broker-two         1.0        karaf   yes          mq-broker-demo-two.amq-broker-two  success           
      gateway                1.0        karaf   yes          gateway-mq                         success           
      esb-consumer           1.0        karaf   yes          default                            success           
      esb-forwarder          1.0        karaf   yes          default                            success           
      esb-producer           1.0        karaf   yes          default                            success           

If you review using the Fuse Management Web Console, the screenshot will be similar to:

![Fuse Fabric Service Containers](./img/Fuse-Fabric-Services-Containers.png "Services - Containers")

Now our Fuse Fabric is ready to execute the services created in each Maven Project.

# Deploy Services into Fabric Environment

This section describes the steps needed to deploy these Camel Contexts into the Fuse Fabric environment created before. Basically the steps are:

* Review Fuse Fabric environment properties
* Build and install Maven artifacts
* Deploy into Fuse Fabric environment using _Fabric8 Maven Plug-In_  

[More informatio about Maven Fabric8 Plug-In][4]

## Review Fuse Fabric environment properties
To connect to our new Fuse Fabric Environment it is needed to define a _<server/>_ section in our __settings.xml__ file. These file is located into our `$MAVEN_REPO` folder (usually in `$USER_HOME/.m2` folder).

`fabric8.server.demo` is the name used to define the Fuse Fabric environment. Check it to use the right credentials:

	<!-- Fabric8 - Demo VM -->
	<server>
		<id>fabric8.server.demo</id>
		<username>admin</username>
		<password>admin</password>
	</server>

Also the __camel-amq-samples/pom.xml__ file has some properties to define the following Fuse Fabric Environment:

* local: Used to deploy into a local Fuse Fabric Environment
* remote: Used to deploy into a remote Fuse Fabric Environment. This environment will be used by `fabric8-remote` Maven profile defined in the parent project. 
	
Check them and set up the right values for each environment:
	 
	<!-- Joolokia Environments -->
	<joolokia.local>http://localhost:8181/jolokia</joolokia.local>
	<joolokia.remote>http://rhel7jboss01:8181/jolokia</joolokia.remote>
	<joolokia.remote.serverId>fabric8.server.demo</joolokia.remote.serverId>

## Build Maven Artifacts
To build the Maven artifacts we will use the `install` Maven goal: 

	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] mvn package
	[INFO] Scanning for projects...
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Build Order:
	[INFO] 
	[INFO] camel-amq-parent
	[INFO] camel-amq-producer
	[INFO] camel-amq-consumer
	[INFO] camel-amq-forwarder
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building camel-amq-parent 6.2.1-SNAPSHOT
	[INFO] ------------------------------------------------------------------------
	[INFO] 
	...
	[INFO] Created profile zip file: target/profile.zip
	[INFO] Attaching aggregated zip /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/target/profile.zip to root project camel-amq-parent
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Summary:
	[INFO] 
	[INFO] camel-amq-parent ................................... SUCCESS [  2.692 s]
	[INFO] camel-amq-producer ................................. SUCCESS [  5.290 s]
	[INFO] camel-amq-consumer ................................. SUCCESS [  0.380 s]
	[INFO] camel-amq-forwarder ................................ SUCCESS [  3.737 s]
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 12.904 s
	[INFO] Finished at: 2016-09-05T12:58:23+02:00
	[INFO] Final Memory: 43M/491M
	[INFO] ------------------------------------------------------------------------

## Deploy Camel Context Producer
Move to __camel-amq-producer__ folder and execute the `fabric8:deploy` Maven goal: 

	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] cd camel-amq-producer
	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer] mvn -Pfabric8-remote clean install fabric8:deploy
	[INFO] Scanning for projects...
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building camel-amq-producer 6.2.1-SNAPSHOT
	[INFO] ------------------------------------------------------------------------
	[INFO] 
	...
	[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621159:zip (zip) @ camel-amq-producer ---
	[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
	[INFO] Writing /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer/target/generated-profiles/com.redhat.camel/camel/amq/producer.profile/dependencies/com.redhat.camel/camel-amq-producer-requirements.json
	[INFO] zipping file com.redhat.camel/camel/amq/producer.profile/ReadMe.md
	[INFO] zipping file com.redhat.camel/camel/amq/producer.profile/com.redhat.camel.environment.properties
	[INFO] zipping file com.redhat.camel/camel/amq/producer.profile/ReadMe.txt
	[INFO] zipping file com.redhat.camel/camel/amq/producer.profile/dependencies/com.redhat.camel/camel-amq-producer-requirements.json
	[INFO] zipping file com.redhat.camel/camel/amq/producer.profile/io.fabric8.agent.properties
	[INFO] Created profile zip file: /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer/target/profile.zip
	[INFO] 
	...
	[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621159:deploy (default-cli) @ camel-amq-producer ---
	[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
	[INFO] Adding needed remote repository: http://repository.jboss.org/nexus/content/groups/fs-public/
	[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer/pom.xml
	Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1.pom
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1.pom (3 KB at 68.2 KB/sec)
	Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/maven-metadata.xml
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml (613 B at 19.3 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/maven-metadata.xml (295 B at 16.9 KB/sec)
	[INFO] Uploading file /home/rmarting/.m2/repository/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-SNAPSHOT.jar
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1.jar
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1.jar (31 KB at 1132.6 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1.pom
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1.pom (3 KB at 96.4 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml (787 B at 38.4 KB/sec)
	[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer/target/profile.zip
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1-profile.zip
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/camel-amq-producer-6.2.1-20160905.111405-1-profile.zip (4 KB at 178.9 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/6.2.1-SNAPSHOT/maven-metadata.xml (1002 B at 44.5 KB/sec)
	[INFO] Updating profile: com.redhat.camel-camel-amq-producer with parent profile(s): [feature-camel] using OSGi resolver
	[INFO] About to invoke mbean io.fabric8:type=ProjectDeployer on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] 
	[INFO] Profile page: http://rhel7jboss01:8181/hawtio/index.html#/wiki/branch/1.0/view/fabric/profiles/com.redhat.camel/camel/amq/producer.profile
	[INFO] 
	[INFO] Uploading file ReadMe.md to invoke mbean io.fabric8:type=Fabric on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] Uploading file com.redhat.camel.environment.properties to invoke mbean io.fabric8:type=Fabric on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] Performing profile refresh on mbean: io.fabric8:type=Fabric version: 1.0 profile: com.redhat.camel-camel-amq-producer
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 02:11 min
	[INFO] Finished at: 2016-09-05T13:16:03+02:00
	[INFO] Final Memory: 46M/507M
	[INFO] ------------------------------------------------------------------------

In Fuse Fabric we could check the status of this new profile:

	JBossFuse:karaf@root> profile-display com.redhat.camel-camel-amq-producer 
	Profile id: com.redhat.camel-camel-amq-producer
	Version   : 1.0
	Attributes: 
		abstract: false
		parents: feature-camel
	Containers: 
	
	Container settings
	----------------------------
	Features : 
		mq-fabric
		camel-amq
	
	Bundles : 
		mvn:com.redhat.camel/camel-amq-producer/6.2.1-SNAPSHOT
	
	Agent Properties : 
		  lastRefresh.com.redhat.camel-camel-amq-producer = 1473074135462
	
	
	Configuration details
	----------------------------
	PID: io.fabric8.web.contextPath
	  com.redhat.camel/camel-amq-producer camel-amq-producer
	
	
	PID: com.redhat.camel.environment
	  amq.producer.brokerURL discovery:(fabric:demo-one)
	  common.prop01 Value for common.prop01
	  env.prop01 Value for env.prop01
	  amq.producer.userName admin
	  amq.producer.password admin
	
	
	
	Other resources
	----------------------------
	Resource: ReadMe.md

Finally we assign this profile to the Service Container created to execute its Camel Context:	

	JBossFuse:karaf@root> container-add-profile esb-producer com.redhat.camel-camel-amq-producer

After some minutes we could check the Camel Routes executions in the log files of this container. If there is not errors, the Camel Routes will show the following log messages:

    2016-11-02 17:41:05,485 | INFO  | ://amq-sender-vt | sendMessageToVirtualTopic        | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | Sending Message '#7381' to virtual topic at discovery:(fabric:demo-one) Broker
    2016-11-02 17:41:15,540 | INFO  | //amq-sender-two | sendMessagetoQueueTwo            | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | Sending Message '#1612' to 'sampleQueueTwo' at discovery:(fabric:demo-one) Broker
    2016-11-02 17:41:33,625 | INFO  | //amq-sender-one | sendMessagetoQueueOne            | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | Sending Message '#6860' to queue at discovery:(fabric:demo-one) Broker. Credentials: admin

If we review the Service Container status using the Fuse Management Web Console, the screenshoot will be similar to the next one:

![Fuse Fabric Service Containers - Producer](./img/Fuse-Fabric-srv-producer-camel.png "Service Container - Producer")

## Deploy Camel Context Consumers
Move to __camel-amq-consumer__ folder and execute the `fabric8:deploy` Maven goal:

	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] cd camel-amq-consumer
	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer] mvn -Pfabric8-remote clean install fabric8:deploy
	[INFO] Scanning for projects...
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building camel-amq-consumer 6.2.1-SNAPSHOT
	[INFO] ------------------------------------------------------------------------
	[INFO] 
	...
	[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621084:zip (zip) @ camel-amq-consumer ---
	[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
	[INFO] Writing /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-consumer/target/generated-profiles/com.redhat.camel/camel/amq/consumer.profile/dependencies/com.redhat.camel/camel-amq-consumer-requirements.json
	[INFO] zipping file com.redhat.camel/camel/amq/consumer.profile/ReadMe.md
	[INFO] zipping file com.redhat.camel/camel/amq/consumer.profile/com.redhat.camel.environment.properties
	[INFO] zipping file com.redhat.camel/camel/amq/consumer.profile/ReadMe.txt
	[INFO] zipping file com.redhat.camel/camel/amq/consumer.profile/dependencies/com.redhat.camel/camel-amq-consumer-requirements.json
	[INFO] zipping file com.redhat.camel/camel/amq/consumer.profile/io.fabric8.agent.properties
	[INFO] Created profile zip file: /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-consumer/target/profile.zip
	[INFO] 
	...
	[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621084:deploy (default-cli) @ camel-amq-consumer ---
	[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
	[INFO] Adding needed remote repository: http://repository.jboss.org/nexus/content/groups/fs-public/
	[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-consumer/pom.xml
	Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1.pom
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1.pom (3 KB at 90.2 KB/sec)
	Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/maven-metadata.xml
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml (613 B at 31.5 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/maven-metadata.xml (295 B at 16.0 KB/sec)
	[INFO] Uploading file /home/rmarting/.m2/repository/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-SNAPSHOT.jar
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1.jar
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1.jar (29 KB at 1256.7 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1.pom
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1.pom (3 KB at 133.2 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml (787 B at 40.5 KB/sec)
	[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-consumer/target/profile.zip
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1-profile.zip
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/camel-amq-consumer-6.2.1-20160905.112128-1-profile.zip (4 KB at 194.4 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/6.2.1-SNAPSHOT/maven-metadata.xml (1002 B at 48.9 KB/sec)
	[INFO] Updating profile: com.redhat.camel-camel-amq-consumer with parent profile(s): [feature-camel] using OSGi resolver
	[INFO] About to invoke mbean io.fabric8:type=ProjectDeployer on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] 
	[INFO] Profile page: http://rhel7jboss01:8181/hawtio/index.html#/wiki/branch/1.0/view/fabric/profiles/com.redhat.camel/camel/amq/consumer.profile
	[INFO] 
	[INFO] Uploading file ReadMe.md to invoke mbean io.fabric8:type=Fabric on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] Uploading file com.redhat.camel.environment.properties to invoke mbean io.fabric8:type=Fabric on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] Performing profile refresh on mbean: io.fabric8:type=Fabric version: 1.0 profile: com.redhat.camel-camel-amq-consumer
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 01:54 min
	[INFO] Finished at: 2016-09-05T13:23:13+02:00
	[INFO] Final Memory: 41M/497M
	[INFO] ------------------------------------------------------------------------

Display the new profile info:

	JBossFuse:karaf@root> profile-display com.redhat.camel-camel-amq-consumer 
	Profile id: com.redhat.camel-camel-amq-consumer
	Version   : 1.0
	Attributes: 
		abstract: false
		parents: feature-camel
	Containers: 
	
	Container settings
	----------------------------
	Features : 
		mq-fabric
		camel-amq
	
	Bundles : 
		mvn:com.redhat.camel/camel-amq-consumer/6.2.1-SNAPSHOT
	
	Agent Properties : 
		  lastRefresh.com.redhat.camel-camel-amq-consumer = 1473074561145
	
	
	Configuration details
	----------------------------
	PID: io.fabric8.web.contextPath
	  com.redhat.camel/camel-amq-consumer camel-amq-consumer
	
	
	PID: com.redhat.camel.environment
	  amq.producer.brokerURL discovery:(fabric:demo-one)
	  common.prop01 Value for common.prop01
	  env.prop01 Value for env.prop01
	  amq.producer.userName admin
	  amq.producer.password admin
	
	
	
	Other resources
	----------------------------
	Resource: ReadMe.md

Finally we assign this profile to the Service Container created to execute its Camel Context:

	JBossFuse:karaf@root> container-add-profile esb-consumer com.redhat.camel-camel-amq-consumer

After some minutes we could check the Camel Routes executions in the log files of this container. If there is not errors, the Camel Routes will show the following log messages:

    2016-11-02 18:01:45,435 | INFO  | ualTopic.Sample] | consumeMessageFromVirtualTopicA1 | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | [CON-A1] Message '7803' consumed from virtual topic at discovery:(fabric:demo-one)
    2016-11-02 18:01:45,535 | INFO  | ualTopic.Sample] | consumeMessageFromVirtualTopicB2 | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | [CON-B2] Message '5578' consumed from virtual topic at discovery:(fabric:demo-one)
    2016-11-02 18:01:45,952 | INFO  | ualTopic.Sample] | consumeMessageFromVirtualTopicC1 | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | [CON-C1] Message '5578' consumed from virtual topic at discovery:(fabric:demo-one)

If we review the Service Container status using the Fuse Management Web Console, the screenshoot will be similar to the next one:

![Fuse Fabric Service Containers - Consumer](./img/Fuse-Fabric-srv-consumer-camel.png "Service Container - Consumer")

## Deploy Camel Context Forwarder
Move to __camel-amq-forwarder__ folder and execute the `fabric8:deploy` Maven goal:

	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] cd camel-amq-forwarder
	[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer] mvn -Pfabric8-remote clean install fabric8:deploy
	[INFO] Scanning for projects...
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building camel-amq-forwarder 6.2.1-SNAPSHOT
	[INFO] ------------------------------------------------------------------------
	[INFO] 
	...
	[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621084:zip (zip) @ camel-amq-forwarder ---
	[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
	[INFO] Writing /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-forwarder/target/generated-profiles/com.redhat.camel/camel/amq/forwarder.profile/dependencies/com.redhat.camel/camel-amq-forwarder-requirements.json
	[INFO] zipping file com.redhat.camel/camel/amq/forwarder.profile/ReadMe.md
	[INFO] zipping file com.redhat.camel/camel/amq/forwarder.profile/com.redhat.camel.environment.properties
	[INFO] zipping file com.redhat.camel/camel/amq/forwarder.profile/ReadMe.txt
	[INFO] zipping file com.redhat.camel/camel/amq/forwarder.profile/dependencies/com.redhat.camel/camel-amq-forwarder-requirements.json
	[INFO] zipping file com.redhat.camel/camel/amq/forwarder.profile/io.fabric8.agent.properties
	[INFO] Created profile zip file: /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-forwarder/target/profile.zip
	[INFO] 
	...
	[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621084:deploy (default-cli) @ camel-amq-forwarder ---
	[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
	[INFO] Adding needed remote repository: http://repository.jboss.org/nexus/content/groups/fs-public/
	[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-forwarder/pom.xml
	Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1.pom
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1.pom (3 KB at 87.4 KB/sec)
	Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/maven-metadata.xml
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml (612 B at 35.2 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/maven-metadata.xml (295 B at 16.9 KB/sec)
	[INFO] Uploading file /home/rmarting/.m2/repository/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-SNAPSHOT.jar
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1.jar
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1.jar (29 KB at 1447.5 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1.pom
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1.pom (3 KB at 164.5 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml (785 B at 47.9 KB/sec)
	[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-forwarder/target/profile.zip
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1-profile.zip
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/camel-amq-forwarder-6.2.1-20160905.112537-1-profile.zip (4 KB at 209.5 KB/sec)
	Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml
	Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/6.2.1-SNAPSHOT/maven-metadata.xml (999 B at 69.7 KB/sec)
	[INFO] Updating profile: com.redhat.camel-camel-amq-forwarder with parent profile(s): [feature-camel] using OSGi resolver
	[INFO] About to invoke mbean io.fabric8:type=ProjectDeployer on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] 
	[INFO] Profile page: http://rhel7jboss01:8181/hawtio/index.html#/wiki/branch/1.0/view/fabric/profiles/com.redhat.camel/camel/amq/forwarder.profile
	[INFO] 
	[INFO] Uploading file ReadMe.md to invoke mbean io.fabric8:type=Fabric on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] Uploading file com.redhat.camel.environment.properties to invoke mbean io.fabric8:type=Fabric on jolokia URL: http://rhel7jboss01:8181/jolokia with user: admin
	[INFO] Performing profile refresh on mbean: io.fabric8:type=Fabric version: 1.0 profile: com.redhat.camel-camel-amq-forwarder
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 01:36 min
	[INFO] Finished at: 2016-09-05T13:27:04+02:00
	[INFO] Final Memory: 48M/872M
	[INFO] ------------------------------------------------------------------------

Display the new profile info:

	JBossFuse:karaf@root> profile-display com.redhat.camel-camel-amq-forwarder 
	Profile id: com.redhat.camel-camel-amq-forwarder
	Version   : 1.0
	Attributes: 
		abstract: false
		parents: feature-camel
	Containers: 
	
	Container settings
	----------------------------
	Features : 
		mq-fabric
		camel-amq
	
	Bundles : 
		mvn:com.redhat.camel/camel-amq-forwarder/6.2.1-SNAPSHOT
	
	Agent Properties : 
		  lastRefresh.com.redhat.camel-camel-amq-forwarder = 1473074799120
	
	
	Configuration details
	----------------------------
	PID: io.fabric8.web.contextPath
	  com.redhat.camel/camel-amq-forwarder camel-amq-forwarder
	
	
	PID: com.redhat.camel.environment
	  amq.source.password admin
	  amq.destination.userName admin
	  common.prop01 Value for common.prop01
	  amq.destination.password admin
	  amq.source.brokerURL discovery:(fabric:demo-one)
	  env.prop01 Value for env.prop01
	  amq.source.userName admin
	  amq.destination.brokerURL discovery:(fabric:demo-two)
	
	
	
	Other resources
	----------------------------
	Resource: ReadMe.md

Finally we assign this profile to the Service Container created to execute its Camel Context:

	JBossFuse:karaf@root> container-add-profile esb-forwarder com.redhat.camel-camel-amq-forwarder

After some minutes we could check the Camel Routes executions in the log files of this container. If there is not errors, the Camel Routes will show the following log messages:

    2016-11-02 17:38:16,735 | INFO  | [sampleQueueTwo] | forwarderRoute                   | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | Message '#6913' consumed from 'sampleQueueTwo' at discovery:(fabric:demo-one) 
    2016-11-02 17:38:16,762 | INFO  | [sampleQueueTwo] | forwarderRoute                   | 153 - org.apache.camel.camel-core - 2.15.1.redhat-621159 | Message '#6913' sent to 'sampleQueueSecond' at discovery:(fabric:demo-two) 

If we review the Service Container status using the Fuse Management Web Console, the screenshoot will be similar to the next one:

![Fuse Fabric Service Containers - Forwarder](./img/Fuse-Fabric-srv-forwarder-camel.png "Service Container - Forwarder")			 

## Monitoring A-MQ Brokers
If we review the AMQ Broker's status in the Fuse Management Web Console, the following screenshoots will be similar to:

![Fuse Fabric AMQ Broker](./img/Fuse-Fabric-amq-broker-queue.png "AMQ Broker - Status")

![Fuse Fabric AMQ Broker2](./img/Fuse-Fabric-amq-broker2-queue.png "AMQ Broker2 - Status")

We could identify the different queues and topics created and used by the different Camel Routes.

## Fuse Discovery Protocol
The Camel Routes use the _fabric discovery agent_ to connect with the different brokers. This method allows us to avoid to use the final container's ports and its hostnames. Fabric will discover the brokers created and will create the connections to the final containers.

To use this discovery method the broker URL will be similar to: 

	amq.producer.brokerURL=discovery:(fabric:demo-one)

More information about [Fuse Fabric Discovery Agent][5] and [how to connect to a broker][6].

<!-- Reference Links -->

[1]: https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=jboss.fuse&version=6.2.1 "Red Hat JBoss Fuse 6.2.1"
[2]: https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.2.1/html-single/fabric_guide/#MQ "Chapter 8. ActiveMQ Brokers and Clusters"
[3]: https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.2.1/html-single/fabric_guide/#Gateway "Chapter 12. Gateway"
[4]: https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.2.1/html-single/fabric_guide/#F8Plugin "Chapter 7. Fabric8 Maven Plug-In"
[5]: https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_A-MQ/6.0/html/Fault_Tolerant_Messaging/files/FMQNetworksFabricDiscovery.html
[6]: https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Fuse/6.2.1/html/Fabric_Guide/MQ-Connecting.html "Connecting to a Broker"
