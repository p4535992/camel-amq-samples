# camel-amq-samples
Camel Routes to manage messaging systems (Active-MQ)


# Install Red Hat JBoss Fuse

## Download
TODO: link to download it:

## Install Red Hat JBos Fuse

~~~~
[root@rhel7jboss01 redhat]# unzip jboss-fuse-full-6.2.1.redhat-084.zip
[root@rhel7jboss01 redhat]# cd jboss-fuse-6.2.1.redhat-084
[root@rhel7jboss01 jboss-fuse]# ./bin/fuse
Please wait while JBoss Fuse is loading...
100% [========================================================================]

_     _ ____                  ______
_    | |  _ \                |  ____|             
_    | | |_) | ___  ___ ___  | |__ _   _ ___  ___
 _   | |  _ < / _ \/ __/ __| |  __| | | / __|/ _ \ 
| |__| | |_) | (_) \__ \__ \ | |  | |_| \__ \  __/
 \____/|____/ \___/|___/___/ |_|   \__,_|___/\___|

  JBoss Fuse (6.2.1.redhat-084)
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
~~~~
	
## Create Fuse Fabric

~~~~
JBossFuse:karaf@root> fabric:create --clean --bind-address rhel7jboss01 --new-user admin --new-user-password admin --zookeeper-password zookeeper --resolver localhostname --wait-for-provisioning 
Waiting for container: root
Waiting for container root to provision.
JBossFuse:karaf@root> status
[profile]                                [instances]    [health]
fabric                                   1              100%
fabric-ensemble-0000-1                   1              100%
jboss-fuse-full                          1              100%
JBossFuse:karaf@root> ensemble-list
[id]
root
JBossFuse:karaf@root> container-list
[id]   [version]  [type]  [connected]  [profiles]              [provision status]
root*  1.0        karaf   yes          fabric                  success           
                                       fabric-ensemble-0000-1                    
                                       jboss-fuse-full                           
~~~~

Removing jboss-fuse-full profile from Fabric containers

~~~~
JBossFuse:karaf@root> container-remove-profile root jboss-fuse-full 
~~~~

## Create MQ-Broker

Broker Original
~~~~
JBossFuse:karaf@root> mq-create --group demo --kind StandAlone --create-container fuse01-amq amq-broker
MQ profile mq-broker-demo.amq-broker ready
Creating new instance on SSH port 8102 and RMI ports 1100/44445 at: /opt/redhat/jboss-fuse-6.2.1.redhat-084/instances/fuse01-amq
~~~~


Status of containers
~~~~
JBossFuse:karaf@root> container-list
[id]          [version]  [type]  [connected]  [profiles]                 [provision status]
root*         1.0        karaf   yes          fabric                     success           
                                              fabric-ensemble-0000-1                       
  fuse01-amq  1.0        karaf   yes          mq-broker-demo.amq-broker  success           
~~~~


Second Broker Original

~~~~
mq-create --group demo2 --kind StandAlone --create-container fuse01-amq2 amq-broker2
JBossFuse:karaf@root> mq-create --group demo2 --kind StandAlone --create-container fuse01-amq2 amq-broker2
MQ profile mq-broker-demo2.amq-broker2 ready
Creating new instance on SSH port 8107 and RMI ports 1105/44450 at: /opt/redhat/jboss-fuse-6.2.1.redhat-084/instances/fuse01-amq2
~~~~



## Create Gateways Containers

~~~~
JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper --profile gateway-mq root fuse01-gw
Creating new instance on SSH port 8103 and RMI ports 1101/44446 at: /opt/redhat/jboss-fuse-6.2.1.redhat-084/instances/fuse01-gw
The following containers have been created successfully:
	Container: fuse01-gw.
~~~~

~~~~
JBossFuse:karaf@root> container-list
[id]          [version]  [type]  [connected]  [profiles]                 [provision status]
root*         1.0        karaf   yes          fabric                     success           
                                              fabric-ensemble-0000-1                       
  fuse01-amq  1.0        karaf   yes          mq-broker-demo.amq-broker  success           
  fuse01-gw   1.0        karaf   yes          gateway-mq                 success        
~~~~


## Create Service Containers

~~~~
JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper root fuse01-srv-producer
Creating new instance on SSH port 8104 and RMI ports 1102/44447 at: /opt/redhat/jboss-fuse-6.2.1.redhat-084/instances/fuse01-srv-producer
The following containers have been created successfully:
	Container: fuse01-srv-producer.
JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper root fuse01-srv-consumer
Creating new instance on SSH port 8105 and RMI ports 1103/44448 at: /opt/redhat/jboss-fuse-6.2.1.redhat-084/instances/fuse01-srv-consumer
The following containers have been created successfully:
	Container: fuse01-srv-consumer.
JBossFuse:karaf@root> container-create-child --zookeeper-password zookeeper root fuse01-srv-forwarder
Creating new instance on SSH port 8106 and RMI ports 1104/44449 at: /opt/redhat/jboss-fuse-6.2.1.redhat-084/instances/fuse01-srv-forwarder
The following containers have been created successfully:
	Container: fuse01-srv-forwarder.
~~~~

~~~~
JBossFuse:karaf@root> container-list
[id]                    [version]  [type]  [connected]  [profiles]                 [provision status]
root*                   1.0        karaf   yes          fabric                     success           
                                                        fabric-ensemble-0000-1                       
  fuse01-amq            1.0        karaf   yes          mq-broker-demo.amq-broker  success           
  fuse01-gw             1.0        karaf   yes          gateway-mq                 success           
  fuse01-srv-consumer   1.0        karaf   yes          default                    success           
  fuse01-srv-forwarder  1.0        karaf   yes          default                    success           
  fuse01-srv-producer   1.0        karaf   yes          default                    success           
~~~~


# Deploy Services into Fabric Environment

## Review Joolokia properties

Define serverId section in $MAVEN_REPO/settings.xml file


		<!-- Fabric8 - Demo VM -->
		<server>
			<id>fabric8.server.demo</id>
			<username>admin</username>
			<password>admin</password>
		</server>

	
Change to be used for each environment
	 
camel-amq-samples/pom.xml Review de following properties

		<!-- Joolokia Environments -->
		<joolokia.local>http://localhost:8181/jolokia</joolokia.local>
		<joolokia.remote>http://rhel7jboss01:8181/jolokia</joolokia.remote>
		<joolokia.remote.serverId>fabric8.server.demo</joolokia.remote.serverId>


![Alt](/img/wp.png "Containers")



### Maven Profiles

There is a Maven Profile to be used to deploy these components to a remote Fuse Fabric.

	id: fabric8-remote		
		
## Build artifacts

amq.producer.brokerURL=discovery:(fabric:demo)

~~~~
mvn package
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
[INFO] Building camel-amq-parent 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
	[Omitted for reduce documentation]
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
~~~~

## Deploy AMQ Producers

~~~~
[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] cd camel-amq-producer
[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer] mvn -Pfabric8-remote clean install fabric8:deploy
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building camel-amq-producer 1.0.30-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
...
[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621084:zip (zip) @ camel-amq-producer ---
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
[INFO] --- fabric8-maven-plugin:1.2.0.redhat-621084:deploy (default-cli) @ camel-amq-producer ---
[INFO] Found class: org.apache.camel.CamelContext so adding the parent profile: feature-camel
[INFO] Adding needed remote repository: http://repository.jboss.org/nexus/content/groups/fs-public/
[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer/pom.xml
Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1.pom
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1.pom (3 KB at 68.2 KB/sec)
Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/maven-metadata.xml
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml (613 B at 19.3 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/maven-metadata.xml (295 B at 16.9 KB/sec)
[INFO] Uploading file /home/rmarting/.m2/repository/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-SNAPSHOT.jar
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1.jar
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1.jar (31 KB at 1132.6 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1.pom
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1.pom (3 KB at 96.4 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml (787 B at 38.4 KB/sec)
[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer/target/profile.zip
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1-profile.zip
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/camel-amq-producer-1.0.30-20160905.111405-1-profile.zip (4 KB at 178.9 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-producer/1.0.30-SNAPSHOT/maven-metadata.xml (1002 B at 44.5 KB/sec)
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
~~~~

~~~~
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
	mvn:com.redhat.camel/camel-amq-producer/1.0.30-SNAPSHOT

Agent Properties : 
	  lastRefresh.com.redhat.camel-camel-amq-producer = 1473074135462


Configuration details
----------------------------
PID: io.fabric8.web.contextPath
  com.redhat.camel/camel-amq-producer camel-amq-producer


PID: com.redhat.camel.environment
  amq.producer.brokerURL discovery:(fabric:demo)
  common.prop01 Value for common.prop01
  env.prop01 Value for env.prop01
  amq.producer.userName admin
  amq.producer.password admin



Other resources
----------------------------
Resource: ReadMe.md
~~~~
	
	
	
	JBossFuse:karaf@root> container-add-profile fuse01-srv-producer com.redhat.camel-camel-amq-producer
	
JBossFuse:karaf@root> container-list
[id]                    [version]  [type]  [connected]  [profiles]                           [provision status]
root*                   1.0        karaf   yes          fabric                               success           
                                                        fabric-ensemble-0000-1                                 
  fuse01-amq            1.0        karaf   yes          mq-broker-demo.amq-broker            success           
  fuse01-gw             1.0        karaf   yes          gateway-mq                           success           
  fuse01-srv-consumer   1.0        karaf   yes          default                              success           
  fuse01-srv-forwarder  1.0        karaf   yes          default                              success           
  fuse01-srv-producer   1.0        karaf   yes          default                              success           
                                                        com.redhat.camel-camel-amq-producer                    
	
	
	
	Assing to fuse01-srv-producer


## Deploy AMQ Consumers

~~~~
[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] cd camel-amq-consumer
[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer] mvn -Pfabric8-remote clean install fabric8:deploy
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building camel-amq-consumer 1.0.22-SNAPSHOT
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
Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1.pom
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1.pom (3 KB at 90.2 KB/sec)
Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/maven-metadata.xml
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml (613 B at 31.5 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/maven-metadata.xml (295 B at 16.0 KB/sec)
[INFO] Uploading file /home/rmarting/.m2/repository/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-SNAPSHOT.jar
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1.jar
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1.jar (29 KB at 1256.7 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1.pom
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1.pom (3 KB at 133.2 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml (787 B at 40.5 KB/sec)
[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-consumer/target/profile.zip
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1-profile.zip
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/camel-amq-consumer-1.0.22-20160905.112128-1-profile.zip (4 KB at 194.4 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-consumer/1.0.22-SNAPSHOT/maven-metadata.xml (1002 B at 48.9 KB/sec)
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
~~~~

~~~~
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
	mvn:com.redhat.camel/camel-amq-consumer/1.0.22-SNAPSHOT

Agent Properties : 
	  lastRefresh.com.redhat.camel-camel-amq-consumer = 1473074561145


Configuration details
----------------------------
PID: io.fabric8.web.contextPath
  com.redhat.camel/camel-amq-consumer camel-amq-consumer


PID: com.redhat.camel.environment
  amq.producer.brokerURL discovery:(fabric:default)
  common.prop01 Value for common.prop01
  env.prop01 Value for env.prop01
  amq.producer.userName admin
  amq.producer.password admin



Other resources
----------------------------
Resource: ReadMe.md
~~~~


JBossFuse:karaf@root> container-add-profile fuse01-srv-consumer com.redhat.camel-camel-amq-consumer



## Deploy AMQ Forwarder

~~~~

[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples] cd camel-amq-forwarder
[rmarting@rhel7 ~/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-producer] mvn -Pfabric8-remote clean install fabric8:deploy
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building camel-amq-forwarder 1.0.3-SNAPSHOT
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
Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1.pom
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1.pom (3 KB at 87.4 KB/sec)
Downloading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/maven-metadata.xml
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml (612 B at 35.2 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/maven-metadata.xml (295 B at 16.9 KB/sec)
[INFO] Uploading file /home/rmarting/.m2/repository/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-SNAPSHOT.jar
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1.jar
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1.jar (29 KB at 1447.5 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1.pom
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1.pom (3 KB at 164.5 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml (785 B at 47.9 KB/sec)
[INFO] Uploading file /home/rmarting/Workspaces/github/camel-amq-samples/camel-amq-samples/camel-amq-forwarder/target/profile.zip
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1-profile.zip
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/camel-amq-forwarder-1.0.3-20160905.112537-1-profile.zip (4 KB at 209.5 KB/sec)
Uploading: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml
Uploaded: http://rhel7jboss01:8181/maven/upload/com/redhat/camel/camel-amq-forwarder/1.0.3-SNAPSHOT/maven-metadata.xml (999 B at 69.7 KB/sec)
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
~~~~


~~~~
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
	mvn:com.redhat.camel/camel-amq-forwarder/1.0.3-SNAPSHOT

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
  amq.source.brokerURL discovery:(fabric:default)
  env.prop01 Value for env.prop01
  amq.source.userName admin
  amq.destination.brokerURL discovery:(fabric:demo)



Other resources
----------------------------
Resource: ReadMe.md
~~~~



JBossFuse:karaf@root> container-add-profile fuse01-srv-forwarder com.redhat.camel-camel-amq-forwarde


			 


 

