


========================================
Maven 
========================================

Introduction
========================================
Maven is a wildly used project builder tool in programming.


How We Use Maven
----------------------------------------
we can test, install, deploy the project by maven.
When we use maven, we must config the settings.xml in maven's configuration.We can use it as its defualt settings, it will download the dependencies from maven website.
But if we use nexus or other maven server, we must add repository{id, url, release, snapshot} into the settings file.
After we build a maven project, we must config the pom.xml.
In the pom file, we add project information like {parent{groupeId, artifactId, version, relativePath}, artifactId, description, properties, dependencies{dependency{groupId, artifactId, version}}}.

When we change pom file(we change dependencies),we have to maven update project to download the denpendencies.
Then we make test of maven, make install to build the project and create jar or war package.
And we maven deploy to upload the builded package to nexus. 

Best Practices
----------------------------------------
The best teacher is practice. So I try to build a maven project.

1) Create a maven project in eclipse. Before this, you need to install the maven plugin in eclipse.
2) Enter the group id, artifact id of the project, and click finish.
3) We can see the pom file has been created in the project directory.

::

 <groupId>com.neunn.neunnmanager-sdk-java</groupId>
 <artifactId>NeunnManager-sdk-java</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <packaging>jar</packaging>
 <name>NeunnManager-sdk-java</name>
 <url>http://maven.apache.org</ur

4) Add some properties in the pom file, so we can manage it more conveniently.

::

 <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <jersey-version>1.17</jersey-version>
 </properties>

5) Add dependencies.

::
    
 <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${spring.version}</version>
 </dependency>
 <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
 </dependency>

6) Add repositories.

::

 <repositories>
    <repository>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <id>public</id>
        <name>Public Repositories</name>
        <url>http://192.168.250.3:8686/nexus/content/groups/public/</url>
    </repository>
 </repositories>

7) Add deployment.(maven deploy)

::

 <distributionManagement>
    <repository>
        <id>releases</id>
        <name>Local Nexus Repository</name>
        <url>http://192.168.250.3:8686/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
        <id>Snapshots</id>
        <name>Local Nexus Repository</name>
        <url>http://192.168.250.3:8686/nexus/content/repositories/snapshots</url>
    </snapshotRepository>
 </distributionManagement>

8) Add modules to manage other projects.

::

 <modules>    
    <module>../com.dugeng.project1</module> 
    <module>../com.dugeng.project2</module> 
 </modules>
