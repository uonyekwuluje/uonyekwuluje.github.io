---
layout: post
title:  "Setting up Java Development Environment"
categories: Java
---

Before you start developing code in Java, you need to setup your development environment with the necessary development tools
required to build, compile, test and execute your java code.

# **Base Requirements**
* **Operating System**: This is the core and starting block. If you are reading this post, you already have one. May be Windows,
  Linux or Mac OS
* **Java Development Kit(JDK)**: This is the foundation and building block of other tools required to write your java code and
  also it's ecosystem. You can download your JDK [Here](https://www.oracle.com/java/technologies/javase-downloads.html)
* **Build/project management tools**: While not compulsory, this will make your life so much easier and stress free.
  * **Maven**: Maven helps and simplifies the build process. You can download Maven [Here](http://maven.apache.org/download.cgi).
  * **Gradle**: Gradle is another tool available for the build process. You can download Maven [Here](https://gradle.org/).
  Personally, I prefer Maven. It's XML based and works great. Just my personal preference.
* **Integrated Development Environment (IDE)**: A IDE helps with code entry and also glues your other tools under the hood. A lot
  of them come with debuggers also. You have lots of options here. IntelliJ Idea from Jetbrains, Eclipse, NetBeans etc. Try them
  out and stick with one you are comfortable with.
* **Git**: Git is used for version control. This is essential and I recommend using this as much as possible.

# **Setup**
This setup is geared towards linux systems and assumes a basic level of systems administration.
* Download JDK and Maven and untar them in your ```opt``` directory. 
* Setup your systems profile and path. For the purpose of this post, my ```/etc/profile``` setup looks like this
```
export JAVA_HOME="/opt/jdk-15"
export M2_HOME="/opt/apache-maven"
export MAVEN_HOME="/opt/apache-maven"
export PATH=$JAVA_HOME/bin:$PATH
export PATH=${M2_HOME}/bin:${PATH}
```
When this is completed, test your setup. ```source /etc/profile```. Check your java version with this command ```java -version```. 
You should see this
```
openjdk version "15" 2020-09-15
OpenJDK Runtime Environment (build 15+36-1562)
OpenJDK 64-Bit Server VM (build 15+36-1562, mixed mode, sharing)
```
Also test Maven. ```mvn --version```. You should see this:
```
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /opt/apache-maven
Java version: 15, vendor: Oracle Corporation, runtime: /opt/jdk-15
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.19.0-11-amd64", arch: "amd64", family: "unix"
```
If you can see the above, you are ready to begin your Java Development Journey


# **HelloWorld**
Any programming tutorial will not be complete without the famous "HelloWorld". We will generate one with some added
features and functionality.
* Create Project using MAVEN
```
mvn archetype:generate -DgroupId=com.javaconcepts.app -DartifactId=javaconcepts -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
cd into the project directory and update your pom file with the code below:
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.javaconcepts.app</groupId>
  <artifactId>javaconcepts</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <name>javaconcepts</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.javaconcepts.app.App</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </plugin>
    </plugins>
  </build>

</project>
```
* Update the base app ```src/main/java/com/javaconcepts/app/App.java```
```
package com.javaconcepts.app;

public class App 
{
    public static void main(String[] args)
    {
        App2 newapp = new App2();
        System.out.println("Hello World!");
        System.out.println("Welcome to my first App");
        newapp.displayName();        
    }
}
```
and create a second one ```src/main/java/com/javaconcepts/app/App2.java ```.
```
package com.javaconcepts.app;

public class App2 
{
    public void displayName()
    {
        System.out.println("Welcome to my second class");
    }
}
```
* Compile with this command ```mvn clean install assembly:single```. If everything works well, execute with this command ```java -jar target/javaconcepts-1.0-SNAPSHOT-jar-with-dependencies.jar```. If all goes well, you should see this
```
Hello World!
Welcome to my first App
Welcome to my second class
```

