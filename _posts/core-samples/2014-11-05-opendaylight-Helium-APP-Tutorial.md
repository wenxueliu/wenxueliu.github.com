---
layout: post
category : SDN
tagline: "opendaylight Helium APP 例子"
tags : [network, sdn, opendaylight]
---
{% include JB/setup %}

**作者**

	刘文学

**版本及修订**

	0.1 完成初始化

**预备条件**

	运行此案例，确保你已经有一个完成整的环境，比如 java 1.7 maven mininet 等软件。

**TODO**

	集成到 karaf 控制台

自定义一个 app
=======================

功能说明

   一台机器为客户端h3，两台服务器h1,h2. controller 监控 Packet_in 事件，
   目的 IP 为 10.0.0.100 MAC 为 00:00:00:00:00:64 的请求，会轮询转发到h1 h2.
   具体可参照 PacketHandler.java 代码。

$ mkdir -p ~/test

$ mvn archetype:generate -DgroupId=org.opendaylight.controller.myapp \
-DartifactId=myapp \
-DarchetypeArtifactId=maven-archetype-quickstart \
-Dpackag=org.opendaylight.controller.myapp \
-DinteractiveMode=false

		[INFO] Scanning for projects...
		[INFO]
		[INFO] ------------------------------------------------------------------------
		[INFO] Building Maven Stub Project (No POM) 1
		[INFO] ------------------------------------------------------------------------
		[INFO]
		[INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
		[INFO]
		[INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
		[INFO]
		[INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
		[INFO] Generating project in Batch mode
		[INFO] ----------------------------------------------------------------------------
		[INFO] Using following parameters for creating project from Old (1.x) Archetype: maven-archetype-quickstart:1.0
		[INFO] ----------------------------------------------------------------------------
		[INFO] Parameter: groupId, Value: org.opendaylight.controller.myapp
		[INFO] Parameter: packageName, Value: org.opendaylight.controller.myapp
		[INFO] Parameter: package, Value: org.opendaylight.controller.myapp
		[INFO] Parameter: artifactId, Value: myapp
		[INFO] Parameter: basedir, Value: /home/vagrant/test
		[INFO] Parameter: version, Value: 1.0-SNAPSHOT
		[INFO] project created from Old (1.x) Archetype in dir: /home/vagrant/test/myapp
		[INFO] ------------------------------------------------------------------------
		[INFO] BUILD SUCCESS
		[INFO] ------------------------------------------------------------------------
		[INFO] Total time: 50.041s
		[INFO] Finished at: Thu Nov 06 02:27:33 UTC 2014
		[INFO] Final Memory: 15M/111M
		[INFO] ------------------------------------------------------------------------


$ tree myapp

		myapp/
		|-- pom.xml
		`-- src
			|-- main
			|   `-- java
			|       `-- org
			|           `-- opendaylight
			|               `-- controller
			|                   `-- myapp
			|                       `-- App.java
			`-- test
				`-- java
				    `-- org
				        `-- opendaylight
				            `-- controller
				                `-- myapp
				                    `-- AppTest.java



$ cd myapp

$ rm src/main/java/org/opendaylight/controller/myapp/App.java

$ vim pom.xml

		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>org.opendaylight.controller.myapp</groupId>
		  <artifactId>myapp</artifactId>
		  <packaging>bundle</packaging>
		  <version>0.1</version>
		  <name>myapp</name>
		  <url>http://maven.apache.org</url>
		  <build>
			<plugins>
			  <plugin>
				<groupId>org.apache.felix</groupId>
				<artifactId>maven-bundle-plugin</artifactId>
				<version>2.3.7</version>
				<extensions>true</extensions>
				<configuration>
				  <instructions>
				    <Import-Package>
				      *
				    </Import-Package>
				    <Export-Package>
				      org.opendaylight.controller.myapp
				    </Export-Package>
				    <Bundle-Activator>
				      org.opendaylight.controller.myapp.Activator
				    </Bundle-Activator>
				  </instructions>
				  <manifestLocation>${project.basedir}/META-INF</manifestLocation>
				</configuration>
			  </plugin>
			</plugins>
		  </build>
		  <dependencies>
			<dependency>
			  <groupId>org.opendaylight.controller</groupId>
			  <artifactId>sal</artifactId>
			  <version>0.7.0</version>
			</dependency>
			<dependency>
			  <groupId>org.opendaylight.controller</groupId>
			  <artifactId>switchmanager</artifactId>
			  <version>0.7.0</version>
			</dependency>
			<dependency>
			  <groupId>junit</groupId>
			  <artifactId>junit</artifactId>
			  <version>3.8.1</version>
			  <scope>test</scope>
			</dependency>
		  </dependencies>
		  <repositories>
			<!-- OpenDaylight releases -->
			<repository>
			  <id>opendaylight-mirror</id>
			  <name>opendaylight-mirror</name>
			  <url>http://nexus.opendaylight.org/content/groups/public/</url>
			  <snapshots>
				  <enabled>false</enabled>
			  </snapshots>
			  <releases>
				  <enabled>true</enabled>
				  <updatePolicy>never</updatePolicy>
			  </releases>
			</repository>
			<!-- OpenDaylight snapshots -->
			<repository>
			  <id>opendaylight-snapshot</id>
			  <name>opendaylight-snapshot</name>
			  <url>http://nexus.opendaylight.org/content/repositories/opendaylight.snapshot/</url>
			  <snapshots>
				  <enabled>true</enabled>
			  </snapshots>
			  <releases>
				  <enabled>false</enabled>
			  </releases>
			</repository>
		  </repositories>
		</project>
		</project>


$ vim src/main/java/org/opendaylight/controller/myapp/PacketHandler.java

		package org.opendaylight.controller.myapp;

		import java.net.InetAddress;
		import java.net.UnknownHostException;
		import java.util.LinkedList;
		import java.util.List;

		import org.opendaylight.controller.sal.action.Action;
		import org.opendaylight.controller.sal.action.Output;
		import org.opendaylight.controller.sal.action.SetDlDst;
		import org.opendaylight.controller.sal.action.SetDlSrc;
		import org.opendaylight.controller.sal.action.SetNwDst;
		import org.opendaylight.controller.sal.action.SetNwSrc;
		import org.opendaylight.controller.sal.core.ConstructionException;
		import org.opendaylight.controller.sal.core.Node;
		import org.opendaylight.controller.sal.core.NodeConnector;
		import org.opendaylight.controller.sal.flowprogrammer.Flow;
		import org.opendaylight.controller.sal.flowprogrammer.IFlowProgrammerService;
		import org.opendaylight.controller.sal.match.Match;
		import org.opendaylight.controller.sal.match.MatchType;
		import org.opendaylight.controller.sal.packet.Ethernet;
		import org.opendaylight.controller.sal.packet.IDataPacketService;
		import org.opendaylight.controller.sal.packet.IListenDataPacket;
		import org.opendaylight.controller.sal.packet.IPv4;
		import org.opendaylight.controller.sal.packet.Packet;
		import org.opendaylight.controller.sal.packet.PacketResult;
		import org.opendaylight.controller.sal.packet.RawPacket;
		import org.opendaylight.controller.sal.packet.TCP;
		import org.opendaylight.controller.sal.utils.EtherTypes;
		import org.opendaylight.controller.sal.utils.Status;
		import org.opendaylight.controller.switchmanager.ISwitchManager;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;

		public class PacketHandler implements IListenDataPacket {

			private static final Logger log = LoggerFactory.getLogger(PacketHandler.class);
			private static final String PUBLIC_IP = "10.0.0.100";
			private static final int SERVICE_PORT = 7777;
			private static final String SERVER1_IP = "10.0.0.1";
			private static final String SERVER2_IP = "10.0.0.2";
			private static final byte[] SERVER1_MAC = {0,0,0,0,0,0x01};
			private static final byte[] SERVER2_MAC = {0,0,0,0,0,0x02};
			private static final byte[] SERVICE_MAC = {0,0,0,0,0,0x64};
			private static final String SERVER1_CONNECTOR_NAME = "s1-eth1";
			private static final String SERVER2_CONNECTOR_NAME = "s1-eth2";

			private IDataPacketService dataPacketService;
			private IFlowProgrammerService flowProgrammerService;
			private ISwitchManager switchManager;
			private InetAddress publicInetAddress;
			private InetAddress server1Address;
			private InetAddress server2Address;
			private int serverNumber = 0;

			static private InetAddress intToInetAddress(int i) {
				byte b[] = new byte[] { (byte) ((i>>24)&0xff), (byte) ((i>>16)&0xff), (byte) ((i>>8)&0xff), (byte) (i&0xff) };
				InetAddress addr;
				try {
				    addr = InetAddress.getByAddress(b);
				} catch (UnknownHostException e) {
				    return null;
				}

				return addr;
			}

			public PacketHandler() {
				try {
				    publicInetAddress = InetAddress.getByName(PUBLIC_IP);
				} catch (UnknownHostException e) {
				    log.error(e.getMessage());
				    System.out.println(e.getMessage());
				}

				try {
				    server1Address = InetAddress.getByName(SERVER1_IP);
				} catch (UnknownHostException e) {
				    log.error(e.getMessage());
				    System.out.println(e.getMessage());
				}

				try {
				    server2Address = InetAddress.getByName(SERVER2_IP);
				} catch (UnknownHostException e) {
				    log.error(e.getMessage());
				    System.out.println(e.getMessage());
				}
			}

			/**
			 * Sets a reference to the requested DataPacketService
			 */
			void setDataPacketService(IDataPacketService s) {
				log.info("Set DataPacketService.");
				System.out.println("Set DataPacketService.");

				dataPacketService = s;
			}

			/**
			 * Unsets DataPacketService
			 */
			void unsetDataPacketService(IDataPacketService s) {
				log.info("Removed DataPacketService.");
				System.out.println("Removed DataPacketService.");

				if (dataPacketService == s) {
				    dataPacketService = null;
				}
			}

			/**
			 * Sets a reference to the requested FlowProgrammerService
			 */
			void setFlowProgrammerService(IFlowProgrammerService s) {
				log.info("Set FlowProgrammerService.");
				System.out.println("Set FlowProgrammerService.");

				flowProgrammerService = s;
			}

			/**
			 * Unsets FlowProgrammerService
			 */
			void unsetFlowProgrammerService(IFlowProgrammerService s) {
				log.info("Removed FlowProgrammerService.");
				System.out.println("Removed FlowProgrammerService.");

				if (flowProgrammerService == s) {
				    flowProgrammerService = null;
				}
			}

			/**
			 * Sets a reference to the requested SwitchManagerService
			 */
			void setSwitchManagerService(ISwitchManager s) {
				log.info("Set SwitchManagerService.");
				System.out.println("Set SwitchManagerService.");

				switchManager = s;
			}

			/**
			 * Unsets SwitchManagerService
			 */
			void unsetSwitchManagerService(ISwitchManager s) {
				log.info("Removed SwitchManagerService.");
				System.out.println("Removed SwitchManagerService.");

				if (switchManager == s) {
				    switchManager = null;
				}
			}

			@Override
			public PacketResult receiveDataPacket(RawPacket inPkt) {
				// The connector, the packet came from ("port")
				NodeConnector ingressConnector = inPkt.getIncomingNodeConnector();
				// The node that received the packet ("switch")
				Node node = ingressConnector.getNode();

				log.info("Packet from " + node.getNodeIDString() + " " + ingressConnector.getNodeConnectorIDString());
				System.out.println("Packet from " + node.getNodeIDString() + " " + ingressConnector.getNodeConnectorIDString());

				// Use DataPacketService to decode the packet.
				Packet pkt = dataPacketService.decodeDataPacket(inPkt);

				if (pkt instanceof Ethernet) {
				    Ethernet ethFrame = (Ethernet) pkt;
				    Object l3Pkt = ethFrame.getPayload();

				    if (l3Pkt instanceof IPv4) {
				        IPv4 ipv4Pkt = (IPv4) l3Pkt;
				        InetAddress clientAddr = intToInetAddress(ipv4Pkt.getSourceAddress());
				        InetAddress dstAddr = intToInetAddress(ipv4Pkt.getDestinationAddress());
				        Object l4Datagram = ipv4Pkt.getPayload();

				        if (l4Datagram instanceof TCP) {
				            TCP tcpDatagram = (TCP) l4Datagram;
				            int clientPort = tcpDatagram.getSourcePort();
				            int dstPort = tcpDatagram.getDestinationPort();

				            if (publicInetAddress.equals(dstAddr) && dstPort == SERVICE_PORT) {
				                log.info("Received packet for load balanced service");
				                System.out.println("Received packet for load balanced service");

				                // Select one of the two servers round robin.

				                InetAddress serverInstanceAddr;
				                byte[] serverInstanceMAC;
				                NodeConnector egressConnector;

				                // Synchronize in case there are two incoming requests at the same time.
				                synchronized (this) {
				                    if (serverNumber == 0) {
				                        log.info("Server 1 is serving the request");
				                        System.out.println("Server 1 is serving the request");
				                        serverInstanceAddr = server1Address;
				                        serverInstanceMAC = SERVER1_MAC;
				                        egressConnector = switchManager.getNodeConnector(node, SERVER1_CONNECTOR_NAME);
				                        serverNumber = 1;
				                    } else {
				                        log.info("Server 2 is serving the request");
				                        System.out.println("Server 2 is serving the request");
				                        serverInstanceAddr = server2Address;
				                        serverInstanceMAC = SERVER2_MAC;
				                        egressConnector = switchManager.getNodeConnector(node, SERVER2_CONNECTOR_NAME);
				                        serverNumber = 0;
				                    }
				                }

				                // Create flow table entry for further incoming packets

				                // Match incoming packets of this TCP connection 
				                // (4 tuple source IP, source port, destination IP, destination port)
				                Match match = new Match();
				                match.setField(MatchType.DL_TYPE, (short) 0x0800);  // IPv4 ethertype
				                match.setField(MatchType.NW_PROTO, (byte) 6);       // TCP protocol id
				                match.setField(MatchType.NW_SRC, clientAddr);
				                match.setField(MatchType.NW_DST, dstAddr);
				                match.setField(MatchType.TP_SRC, (short) clientPort);
				                match.setField(MatchType.TP_DST, (short) dstPort);

				                // List of actions applied to the packet
				                List<Action> actions = new LinkedList<Action>();

				                // Re-write destination IP to server instance IP
				                actions.add(new SetNwDst(serverInstanceAddr));

				                // Re-write destination MAC to server instance MAC
				                actions.add(new SetDlDst(serverInstanceMAC));

				                // Output packet on port to server instance
				                actions.add(new Output(egressConnector));

				                // Create the flow
				                Flow flow = new Flow(match, actions);

				                // Use FlowProgrammerService to program flow.
				                Status status = flowProgrammerService.addFlow(node, flow);
				                if (!status.isSuccess()) {
				                    log.error("Could not program flow: " + status.getDescription());
				                    return PacketResult.CONSUME;
				                }

				                // Create flow table entry for response packets from server to client
				                // Match outgoing packets of this TCP connection
				                match = new Match();
				                match.setField(MatchType.DL_TYPE, (short) 0x0800);
				                match.setField(MatchType.NW_PROTO, (byte) 6);
				                match.setField(MatchType.NW_SRC, serverInstanceAddr);
				                match.setField(MatchType.NW_DST, clientAddr);
				                match.setField(MatchType.TP_SRC, (short) dstPort);
				                match.setField(MatchType.TP_DST, (short) clientPort);

				                // Re-write the server instance IP address to the public IP address
				                actions = new LinkedList<Action>();
				                actions.add(new SetNwSrc(publicInetAddress));
				                actions.add(new SetDlSrc(SERVICE_MAC));

				                // Output to client port from which packet was received
				                actions.add(new Output(ingressConnector));
				                flow = new Flow(match, actions);
				                status = flowProgrammerService.addFlow(node, flow);
				                if (!status.isSuccess()) {
				                    log.error("Could not program flow: " + status.getDescription());
				                    return PacketResult.CONSUME;
				                }

				                // Forward initial packet to selected server
				                log.info("Forwarding packet to " + serverInstanceAddr.toString() + " through port " + egressConnector.getNodeConnectorIDString());
				                System.out.println("Forwarding packet to " + serverInstanceAddr.toString() + " through port " + egressConnector.getNodeConnectorIDString());
				                ethFrame.setDestinationMACAddress(serverInstanceMAC);
				                ipv4Pkt.setDestinationAddress(serverInstanceAddr);
				                inPkt.setOutgoingNodeConnector(egressConnector);
				                dataPacketService.transmitDataPacket(inPkt);
				                return PacketResult.CONSUME;
				            }
				        }
				    }
				}

				// We did not process the packet -> let someone else do the job.
				return PacketResult.IGNORED;
			}

		}



$ vim src/main/java/org/opendaylight/controller/myapp/Activator.java

		package org.opendaylight.controller.myapp;

		import java.util.Dictionary;
		import java.util.Hashtable;

		import org.apache.felix.dm.Component;
		import org.opendaylight.controller.sal.core.ComponentActivatorAbstractBase;
		import org.opendaylight.controller.sal.flowprogrammer.IFlowProgrammerService;
		import org.opendaylight.controller.sal.packet.IDataPacketService;
		import org.opendaylight.controller.sal.packet.IListenDataPacket;
		import org.opendaylight.controller.switchmanager.ISwitchManager;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;

		public class Activator extends ComponentActivatorAbstractBase {
			private static final Logger log = LoggerFactory.getLogger(PacketHandler.class);

			public Object[] getImplementations() {
				log.trace("Getting Implementations");

				Object[] res = { PacketHandler.class };
				return res;
			}

			public void configureInstance(Component c, Object imp, String containerName) {
				log.trace("Configuring instance");

				if (imp.equals(PacketHandler.class)) {
				    // Define exported and used services for PacketHandler component.

				    Dictionary<String, Object> props = new Hashtable<String, Object>();
				    props.put("salListenerName", "mypackethandler");

				    // Export IListenDataPacket interface to receive packet-in events.
				    c.setInterface(new String[] {IListenDataPacket.class.getName()}, props);

				    // Need the DataPacketService for encoding, decoding, sending data packets
				    c.add(createContainerServiceDependency(containerName).setService(
				            IDataPacketService.class).setCallbacks(
				            "setDataPacketService", "unsetDataPacketService")
				            .setRequired(true));

				    // Need FlowProgrammerService for programming flows
				    c.add(createContainerServiceDependency(containerName).setService(
				            IFlowProgrammerService.class).setCallbacks(
				            "setFlowProgrammerService", "unsetFlowProgrammerService")
				            .setRequired(true));

				    // Need SwitchManager service for enumerating ports of switch
				    c.add(createContainerServiceDependency(containerName).setService(
				            ISwitchManager.class).setCallbacks(
				            "setSwitchManagerService", "unsetSwitchManagerService")
				            .setRequired(true));
				}

			}
		}

$ mvn clean install

		[INFO] Scanning for projects...
		[INFO]
		[INFO] ------------------------------------------------------------------------
		[INFO] Building myapp 0.1
		[INFO] ------------------------------------------------------------------------
		[INFO]
		[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ myapp ---
		[INFO] Deleting /home/vagrant/test/myapp/target
		[INFO]
		[INFO] --- maven-resources-plugin:2.7:resources (default-resources) @ myapp ---
		[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
		[INFO] skip non existing resourceDirectory /home/vagrant/test/myapp/src/main/resources
		[INFO]
		[INFO] --- maven-compiler-plugin:3.2:compile (default-compile) @ myapp ---
		[INFO] Changes detected - recompiling the module!
		[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
		[INFO] Compiling 2 source files to /home/vagrant/test/myapp/target/classes
		[INFO]
		[INFO] --- maven-resources-plugin:2.7:testResources (default-testResources) @ myapp ---
		[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
		[INFO] skip non existing resourceDirectory /home/vagrant/test/myapp/src/test/resources
		[INFO]
		[INFO] --- maven-compiler-plugin:3.2:testCompile (default-testCompile) @ myapp ---
		[INFO] Changes detected - recompiling the module!
		[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
		[INFO] Compiling 1 source file to /home/vagrant/test/myapp/target/test-classes
		[INFO]
		[INFO] --- maven-surefire-plugin:2.17:test (default-test) @ myapp ---
		[INFO] Surefire report directory: /home/vagrant/test/myapp/target/surefire-reports
		Downloading: http://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit3/2.17/surefire-junit3-2.17.pom
		Downloaded: http://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit3/2.17/surefire-junit3-2.17.pom (2 KB at 0.2 KB/sec)
		Downloading: http://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-providers/2.17/surefire-providers-2.17.pom
		Downloaded: http://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-providers/2.17/surefire-providers-2.17.pom (3 KB at 2.1 KB/sec)
		Downloading: http://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit3/2.17/surefire-junit3-2.17.jar
		Downloaded: http://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit3/2.17/surefire-junit3-2.17.jar (25 KB at 21.6 KB/sec)

		-------------------------------------------------------
		 T E S T S
		-------------------------------------------------------
		Running org.opendaylight.controller.myapp.AppTest
		Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.007 sec - in org.opendaylight.controller.myapp.AppTest

		Results :

		Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

		[INFO]
		[INFO] --- maven-bundle-plugin:2.3.7:bundle (default-bundle) @ myapp ---
		[INFO]
		[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ myapp ---
		[INFO] Installing /home/vagrant/test/myapp/target/myapp-0.1.jar to /home/vagrant/.m2/repository/org/opendaylight/controller/myapp/myapp/0.1/myapp-0.1.jar
		[INFO] Installing /home/vagrant/test/myapp/pom.xml to /home/vagrant/.m2/repository/org/opendaylight/controller/myapp/myapp/0.1/myapp-0.1.pom
		[INFO]
		[INFO] --- maven-bundle-plugin:2.3.7:install (default-install) @ myapp ---
		[INFO] Installing org/opendaylight/controller/myapp/myapp/0.1/myapp-0.1.jar
		[INFO] Writing OBR metadata
		[INFO] ------------------------------------------------------------------------
		[INFO] BUILD SUCCESS
		[INFO] ------------------------------------------------------------------------
		[INFO] Total time: 14.839s
		[INFO] Finished at: Thu Nov 06 02:40:41 UTC 2014
		[INFO] Final Memory: 20M/175M
		[INFO] ------------------------------------------------------------------------

测试 APP
===========================

新建一个窗口连接到环境服务器

	ssh -X username@IP

* 获取 controlle 代码

$ mkdir -p ~/opendaylight

$ git clone https://github.com/opendaylight/controller.git

* 切换controller 到 stable/helium 分支

$ cd ~/opendaylight/controller

$ git checkout -b stable/helium  origin/stable/helium

* 安装 controller

$ cd ~/opendaylight/controller/opendaylight/distribution/opendaylight/

$ mvn clean install

* 进入 osgi 控制台

$ cd target/distribution.opendaylight-osgipackage/opendaylight

$ ./run.sh

* 安装 app

osgi > install file:/home/vagrant/test/myapp/target/myapp-0.1.jar

		Bundle id is 281
		ClassLoader          null
		RegisteredServices   null
		ServicesInUse        null
		LoaderProxy          org.opendaylight.controller.myapp; bundle-version="0.1.0"
		Fragments            null
		Version              0.1.0
		LastModified         1415241909876
		Headers               Bnd-LastModified = 1415241641291
		 Build-Jdk = 1.7.0_65
		 Built-By = vagrant
		 Bundle-Activator = org.opendaylight.controller.myapp.Activator
		 Bundle-ManifestVersion = 2
		 Bundle-Name = myapp
		 Bundle-SymbolicName = org.opendaylight.controller.myapp
		 Bundle-Version = 0.1.0
		 Created-By = Apache Maven Bundle Plugin
		 Export-Package = org.opendaylight.controller.myapp;uses:="org.opendaylight.controller.sal.flowprogrammer,org.opendaylight.controller.sal.core,org.apache.felix.dm,org.opendaylight.controller.switchmanager,org.opendaylight.controller.sal.packet,org.slf4j,org.opendaylight.controller.sal.utils,org.opendaylight.controller.sal.action,org.opendaylight.controller.sal.match";version="0.1.0"
		 Import-Package = org.apache.felix.dm;version="[3.0,4)",org.opendaylight.controller.sal.action;version="[0.7,1)",org.opendaylight.controller.sal.core;version="[0.7,1)",org.opendaylight.controller.sal.flowprogrammer;version="[0.7,1)",org.opendaylight.controller.sal.match;version="[0.7,1)",org.opendaylight.controller.sal.packet;version="[0.7,1)",org.opendaylight.controller.sal.utils;version="[0.7,1)",org.opendaylight.controller.switchmanager;version="[0.7,1)",org.slf4j;version="[1.7,2)"
		 Manifest-Version = 1.0
		 Tool = Bnd-1.50.0


		ProtectionDomain     null
		Key                  281
		Location             file:/home/vagrant/test/myapp/target/myapp-0.1.jar
		State                2
		Bundle                 281|Installed  |    1|org.opendaylight.controller.myapp (0.1.0)
		BundleContext        null
		BundleId             281
		StartLevel           1
		SymbolicName         org.opendaylight.controller.myapp
		BundleData           org.opendaylight.controller.myapp_0.1.0
		KeyHashCode          281
		StateChanging        null
		BundleDescription    org.opendaylight.controller.myapp_0.1.0
		Framework            org.eclipse.osgi.framework.internal.core.Framework@68852efd
		ResolutionFailureException org.osgi.framework.BundleException: The bundle "org.opendaylight.controller.myapp_0.1.0 [281]" could not be resolved
		Revisions            [org.opendaylight.controller.myapp_0.1.0]

osgi > start 281

		2014-11-06 02:45:49.571 UTC [Gogo shell] INFO  o.o.controller.myapp.PacketHandler  - Set DataPacketService.
		Set DataPacketService.
		2014-11-06 02:45:49.572 UTC [Gogo shell] INFO  o.o.controller.myapp.PacketHandler  - Set FlowProgrammerService.
		Set FlowProgrammerService.
		2014-11-06 02:45:49.572 UTC [Gogo shell] INFO  o.o.controller.myapp.PacketHandler  - Set SwitchManagerService.
		Set SwitchManagerService.



新建一个窗口连接到环境服务器，

	ssh -X username@IP  //注意，ssh 中一定要加 -X 选项。

运行

$ sudo mn --controller=remote,ip=192.168.1.12 --topo single,3 --mac --arp

	*** Creating network
	*** Adding controller
	*** Adding hosts:
	h1 h2 h3
	*** Adding switches:
	s1
	*** Adding links:
	(h1, s1) (h2, s1) (h3, s1)
	*** Configuring hosts
	h1 h2 h3
	*** Starting controller
	*** Starting 1 switches
	s1
	*** Starting CLI:

mininet> xterm h1 h2 h3

这时出现三个窗口，h1 h2 h3

在h1 中运行

	$ nc -l 7777

在h2 中运行

	$ nc -l 7777

在h3 中运行

	$ arp -s 10.0.0.100 00:00:00:00:00:64
    $ nc 10.0.0.100 777
    hello
    world

这是在 h1 和 h2 窗口中分别出现 hello 和 world


测试成功。




