---
layout: post
title:  "Setting up Java Development Environment"
categories: Java
---

Before you start developing code in Java, you need to setup your development environment with the necessary development tools
required to build, compile, test and execute your java code.

# Base Requirements
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

# Setup
This setup is geared towards linux systems and assumes a basic level of systems administration.
* Download JDK and Maven and untar them in your ```opt``` directory. 
* Setup your systems profile and path. For the purpose of this post, my ```/etc/profile``` setu looks like this
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
