
术语

JMX : Java Management Extension
MBean : managed Beans
Jenkins

###YANG Tools

YANG Tools 是用 YANG 这种模式语言(modeling language)构建的一系列库和工具，用于 java
工程和应用。比如 Opendaylight Controller 中的 MD-SAL，java 代码生成工具等。

YANG Tools is a infrastructure project aiming to develop necessary tooling and
libraries providing support of NETCONF and YANG for Java (JVM-language based)
projects and applications, such as Model Driven SAL for Controller (which uses
YANG as it's modeling language) and Netconf / OFConfig plugins.

* Yang Parser

Parse YANG from files, streams into Java Metamodel

解析 YANG 文件和流 为 Java 元模型(Metamodel)

* Yang Maven Tools

Maven plugin responsible for parsing and processing YANG files with plugable
architecture for third-party code generators

Maven 插件，负责解析和处理 YANG 文件。

* YANG REST API Generator

General-purpose implementation of REST-APIs based on YANG modules and mapping
defined in draft-biermann-netconf-yang-api

基于 YANG 模块和 draft-biermann-netconf-yang-api （定义 YANG 和 REST API
的映射关系），产生 REST-API


* Generation of DSDL (RelaxNG) based on RFC6110
* Generation of YIN representation (RFC6020)
    compile time (using custom generator for YANG Maven Tools)
    runtime (transforming org.opendaylight.yangtools.yang.model.* classes
                 to their YIN representation in XML DOM)
* Implementation of validation of instance data (XML) based on YANG to
* RelaxNG mapping - RFC6110

* Parser of YANG files
* Java meta-model for YANG
* Java binding for YANG
* Maven plugin for processing YANG files
* Infrastructure for code generators based on YANG
* Validation of instance data (XML) based on YANG to RelaxNG mapping - RFC6110
* Proof-of-concept, research and support for new YANG extensions, which are meant to be reused by other
projects.
* IDE related tools to assist in writing, using and developing YANG models
* Libraries and supporting functionality for YANG API (REST APIs defined by the YANG model).


###预备知识

* RFC6021
* YANG RFC6020
* JMX MBean:
* Xtend : source code templates.

###Opendaylight 扩展

Opendaylight introduces a layer of separation that isolates data, RPCs and
notifications, based on the revision of modules, to support multiple versions of
the clients at the same time without the risk of accidentally changing data,
which semantics changed between revisions.

Because multiple revisions of the same module can co-exist in the system, the
architecture must support components (plugins) responsible for translating data
between different module revisions.

####QNames

In context of YANG, a QName is a full name of a defined node, type, procedure or
a notification. It is used to prevent name clashes between nodes with the same local name, but
from different schemas.

QName = (XMLNamespace,Revision,LocalName)

* XMLNamespace

    namespace assigned to the YANG module with defined element, type, procedure or notification.

* Revision

    revision of the YANG module that describes the element

* LocalName

    YANG schema identifier that was defined for this node in the YANG module

####RPC

In regular Netconf/YANG use cases, RPCs are used to model functionality and APIs
provided by a Netconf server to a Netconf client.

In context of the Controller SAL, RPCs are used to model functionality provided
by Providers and consumed by Consumers.

更多参考[这里](https://wiki.opendaylight.org/view/OpenDaylight_Controller:YANG_Schema_and_Model)

###文件结构

####包名

* 包名都以组件名 org.opendaylight.yang 为前缀
* 前缀应该在组件后面，如  org.opendaylight.yang.parser
* 复杂的对象可以有一些子包，如 org.opendaylight.yang.parser.util
* 在组件名之后的部分用于区分公共的还是私有的组件。公共的接口必须放在不同的包，
实现也应该放在不同的包，比如 org.opendaylight.yang.service 包含公共服务的定义，
org.opendaylight.yang.service.impl 包含服务的实现。org.opendaylight.yang.service.mock
包含服务的 mock 实现

推荐的值如下表：

    api     : 释放给组件外面的公共接口
    spi     : 提供功能的公共接口
    impl    : 主要组件的实现,如果用于产品目的实现多于一个，不要用这个后缀，用描述实现的名称
    util    : 为了更好地实现提供的工具函数
    mock    : 用于测试目的的 mock 实现
    test    : 单元测试组件


参考[这里](https://wiki.opendaylight.org/view/YANG_Tools:Design_and_Coding_Guidelines)



####yang folder

* yang-common

Common concepts for YANG

* yang-model-api

Java Interfaces describing in-memory model of parsed

* YANG schema

* yang-model-util

Util classes and methods for parsed YANG schema code-generator folder

####code-generator folder

* binding-generator-*

APIs and Implementation of component responsible for mapping between YANG and JAVA

*binding-java-api-generator

Component responsible for writing the mapping as Java code

* binding-model-api

    APIs for modeling JAVA types which are to be genereated
* code-generator-demo

    Demo code generator application which was presented on HackFest

* maven-yang-*

Maven plugin with SPI for adding components processing YANG files in Maven build
process

* yang-model-parser-*

Implementation of YANG parser, which parses input text YANG files into the
objects defined in yang-model-api.

 - yang-binding - Java interfaces for generated code
  - yang-common - Common terms
   - yang-data-api - DOM-like APIs
    - yang-model-api - Semantic model of parsed YANG files
     - yang-maven-plugin - Maven plugin for processing YANG files
      - yang-model-parser-impl - ANTLR4 parser
       - binding-model-api - Interfaces / model of Java
        - binding-generator - Translates YANG to Java

            maven-yang-* folders SHOULD be renamed to yang-maven-*
                yang-maven-plugin-* - packages org.opendaylight.controller.
                    yang-model-parser-* folders SHOULD be moved to yang folder
                    instead of code-generator
                        yang-model-parser-* folders and Maven artefacts SHOULD
                        be renamed to yang-parser-*
                            binding-java-api-generator add functionality to test
                            generated APIs with external non-generated
                            dependencies;
                                binding-generator-impl Split GeneratedTypesTest
                                into more granular tests based on used yang
                                resources and use cases;
                                    yang-model-parser-impl Improvement of
                                    YangModelParserImpl, YangModelParserListener
                                    and BasicValidations in
                                    yang-model-parser-impl for better NPE
                                    handling


###Code Map

https://wiki.opendaylight.org/view/Yang_Tools:Code_Generation_Demo:YANG2JAVA_Mapping_%28Flow_example%29

https://wiki.opendaylight.org/view/YANG_Tools:YANG_to_Java_Mapping

###YANG Demo

####l2switch

$ git clone https://github.com/opendaylight/l2switch.git

参照

* [这里](https://wiki.opendaylight.org/view/L2_Switch:Tutorial:Yang)
* [这里](https://wiki.opendaylight.org/view/Yang_Tools:Code_Generation_Demo:YANG2JAVA_Mapping_%28Flow_example%29) 

来研究 YANG 的机制

###YANG Demo

$ cd controller/opendaylight/md-sal/samples

####toaster

* pom.xml contains configuration for project and code generation plugin
* src/main/yang/toaster.yang contains YANG model of toaster

####toaster-consumer

contains sample consumer code for MD-SAL

####toaster-provider

contains sample provider code for MD-SAL

####toaster-it

Sample PAX EXAM integration test using these dependencies.

####toaster-config

contains sample config code for MD-SAL

###RUN
mvn clean install
mvn generate-sources


###Ping Demo

https://wiki.opendaylight.org/view/Ping



##Reference

https://wiki.opendaylight.org/view/YANG_Tools:Helium:How-To%27s/Tutorials
https://wiki.opendaylight.org/view/YANG_Tools:Component_Map
[YANG2JAVA_1](https://wiki.opendaylight.org/view/YANG_Tools:YANG_to_Java_Mapping)
{YANG2JAVA_2}(https://wiki.opendaylight.org/view/Yang_Tools:Code_Generation_Demo:YANG2JAVA_Mapping_%28Flow_example%29)

[Feature-Hydrogen]https://wiki.opendaylight.org/view/YANG_Tools:Hydrogen_Release_Review
[Feature-Helium](https://wiki.opendaylight.org/view/YANG_Tools:Helium:Release_Notes)
[Develop Road](https://wiki.opendaylight.org/view/YANG_Tools:Release_Plan_2013)
[可用的模块](https://wiki.opendaylight.org/view/YANG_Tools:Available_Models)
[Schema and Model](https://wiki.opendaylight.org/view/OpenDaylight_Controller:YANG_Schema_and_Model)



##Suffix

follow must be given in build section of pom.xml

     <!-- 可选 -->
     <dependency>
       <groupId>org.opendaylight.yangtools</groupId>
       <artifactId>yang-binding</artifactId>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools</groupId>
       <artifactId>yang-common</artifactId>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools.model</groupId>
       <artifactId>ietf-yang-types</artifactId>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools.model</groupId>
       <artifactId>ietf-inet-types</artifactId>
       <version>${ietf-inet-types.version}</version>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools.model</groupId>
       <artifactId>yang-ext</artifactId>
       <version>${yang-ext.version}</version>
     </dependency>

    <!--必须-->
    <plugins>
      <plugin>
        <groupId>org.opendaylight.yangtools</groupId>
        <artifactId>yang-maven-plugin</artifactId>
        <version>${yangtools.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>generate-sources</goal>
            </goals>
            <configuration>
              <yangFilesRootDir>src/main/yang</yangFilesRootDir>
              <codeGenerators>
                <generator>
                  <codeGeneratorClass>org.opendaylight.yangtools.maven.sal.api.gen.plugin.CodeGeneratorImpl</codeGeneratorClass>
                  <outputBaseDir>${salGeneratorPath}</outputBaseDir>
                </generator>
                <generator>
                  <codeGeneratorClass>org.opendaylight.yangtools.yang.unified.doc.generator.maven.DocumentationGeneratorImpl</codeGeneratorClass>
                  <outputBaseDir>target/site/models</outputBaseDir>
                </generator>
                <generator>
                  <codeGeneratorClass>org.opendaylight.yangtools.yang.wadl.generator.maven.WadlGenerator</codeGeneratorClass>
                  <outputBaseDir>target/site/models</outputBaseDir>
                </generator>
              </codeGenerators>
              <inspectDependencies>true</inspectDependencies>
            </configuration>
          </execution>
        </executions>

		<!--可选 -->
        <dependencies>
          <dependency>
            <groupId>org.opendaylight.yangtools</groupId>
            <artifactId>maven-sal-api-gen-plugin</artifactId>
            <version>${yangtools.version}</version>
            <type>jar</type>
          </dependency>
          <dependency>
            <groupId>org.opendaylight.yangtools</groupId>
            <artifactId>yang-binding</artifactId>
            <version>${yangtools.version}</version>
            <type>jar</type>
          </dependency>
          <dependency>
            <groupId>org.opendaylight.controller</groupId>
            <artifactId>yang-jmx-generator-plugin</artifactId>
            <version>${config.version}</version>
          </dependency>
        </dependencies>
      </plugin>
    <plugins>


###YANG Demo

$ cd controller/opendaylight/md-sal/samples

####toaster

* pom.xml contains configuration for project and code generation plugin
* src/main/yang/toaster.yang contains YANG model of toaster

####toaster-consumer

contains sample consumer code for MD-SAL

####toaster-provider

contains sample provider code for MD-SAL

####toaster-it

Sample PAX EXAM integration test using these dependencies. 

####toaster-config

contains sample config code for MD-SAL

###RUN
mvn clean install
mvn generate-sources 


###Ping Demo

https://wiki.opendaylight.org/view/Ping



##Reference

https://wiki.opendaylight.org/view/YANG_Tools:Helium:How-To%27s/Tutorials
https://wiki.opendaylight.org/view/YANG_Tools:Component_Map
[YANG2JAVA_1](https://wiki.opendaylight.org/view/YANG_Tools:YANG_to_Java_Mapping)
{YANG2JAVA_2}(https://wiki.opendaylight.org/view/Yang_Tools:Code_Generation_Demo:YANG2JAVA_Mapping_%28Flow_example%29)

[Feature-Hydrogen]https://wiki.opendaylight.org/view/YANG_Tools:Hydrogen_Release_Review
[Feature-Helium](https://wiki.opendaylight.org/view/YANG_Tools:Helium:Release_Notes)
[Develop Road](https://wiki.opendaylight.org/view/YANG_Tools:Release_Plan_2013)
[Available Model](https://wiki.opendaylight.org/view/YANG_Tools:Available_Models)
[Schema and Model](https://wiki.opendaylight.org/view/OpenDaylight_Controller:YANG_Schema_and_Model)



##Suffix

follow must be given in build section of pom.xml

     <!-- 可选 -->
     <dependency>
       <groupId>org.opendaylight.yangtools</groupId>
       <artifactId>yang-binding</artifactId>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools</groupId>
       <artifactId>yang-common</artifactId>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools.model</groupId>
       <artifactId>ietf-yang-types</artifactId>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools.model</groupId>
       <artifactId>ietf-inet-types</artifactId>
       <version>${ietf-inet-types.version}</version>
     </dependency>
     <dependency>
       <groupId>org.opendaylight.yangtools.model</groupId>
       <artifactId>yang-ext</artifactId>
       <version>${yang-ext.version}</version>
     </dependency>

    <!--必须-->
    <plugins>
      <plugin>
        <groupId>org.opendaylight.yangtools</groupId>
        <artifactId>yang-maven-plugin</artifactId>
        <version>${yangtools.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>generate-sources</goal>
            </goals>
            <configuration>
              <yangFilesRootDir>src/main/yang</yangFilesRootDir>
              <codeGenerators>
                <generator>
                  <codeGeneratorClass>org.opendaylight.yangtools.maven.sal.api.gen.plugin.CodeGeneratorImpl</codeGeneratorClass>
                  <outputBaseDir>${salGeneratorPath}</outputBaseDir>
                </generator>
                <generator>
                  <codeGeneratorClass>org.opendaylight.yangtools.yang.unified.doc.generator.maven.DocumentationGeneratorImpl</codeGeneratorClass>
                  <outputBaseDir>target/site/models</outputBaseDir>
                </generator>
                <generator>
                  <codeGeneratorClass>org.opendaylight.yangtools.yang.wadl.generator.maven.WadlGenerator</codeGeneratorClass>
                  <outputBaseDir>target/site/models</outputBaseDir>
                </generator>
              </codeGenerators>
              <inspectDependencies>true</inspectDependencies>
            </configuration>
          </execution>
        </executions>

		<!--可选 -->
        <dependencies>
          <dependency>
            <groupId>org.opendaylight.yangtools</groupId>
            <artifactId>maven-sal-api-gen-plugin</artifactId>
            <version>${yangtools.version}</version>
            <type>jar</type>
          </dependency>
          <dependency>
            <groupId>org.opendaylight.yangtools</groupId>
            <artifactId>yang-binding</artifactId>
            <version>${yangtools.version}</version>
            <type>jar</type>
          </dependency>
          <dependency>
            <groupId>org.opendaylight.controller</groupId>
            <artifactId>yang-jmx-generator-plugin</artifactId>
            <version>${config.version}</version>
          </dependency>
        </dependencies>
      </plugin>
    <plugins>

