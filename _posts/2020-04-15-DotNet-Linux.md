---
layout: post
title:  ".NET Linux Setup"
categories: .Net, Windows, Programming
---

This post focuses on a base install of .Net in linux. I found myself dealing with lots of build related issues with windows.
Initially, i did not want to deal with it but as I helped our development team with their pipeline setup, i learnt so much
and I have to admit; Windows has come a long way and the experience is much better now.
The fact that I can install .Net in linux and use it for my application development makes this even more appealing to me. This
post accomplishes the following:

* Install .Net Core SDK in Debian 
* Testing out a basic C# program 
 

#### **Systems Requirements and Dependencies**
Register the Microsoft key and set up apt sources with the commands below:
```
wget -O- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
sudo mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
wget https://packages.microsoft.com/config/debian/10/prod.list
sudo mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
sudo chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
sudo chown root:root /etc/apt/sources.list.d/microsoft-prod.list
sudo apt update
sudo apt install apt-transport-https
```

#### **Install .NET Core and Runtime**
Install .Net Core SDK, ASP.NET Core Runtime and .NET Core Runtime
```
sudo apt install dotnet-sdk-3.1 aspnetcore-runtime-3.1 dotnet-runtime-3.1
```

When the installation is complete, type ```dotnet``` in your terminal. If all went well, you should see this:
```
Usage: dotnet [options]
Usage: dotnet [path-to-application]

Options:
  -h|--help         Display help.
  --info            Display .NET Core information.
  --list-sdks       Display the installed SDKs.
  --list-runtimes   Display the installed runtimes.

path-to-application:
  The path to an application .dll file to execute.
```



#### **Hello World C#**
With our requirements in place, its time to test it with the classic HelloWorld App

```
dotnet new console -o helloWorld
cd helloWorld
```

This creates a new application with base scaffolding and should look like this:
```
helloWorld/
├── helloWorld.csproj
├── obj
│   ├── helloWorld.csproj.nuget.dgspec.json
│   ├── helloWorld.csproj.nuget.g.props
│   ├── helloWorld.csproj.nuget.g.targets
│   ├── project.assets.json
│   └── project.nuget.cache
└── Program.cs
```

To run Program.cs, type the command ```dotnet run```. You should see this:
```
Hello World!
```
and that is it. You can now continue with your application development tasks. Have fun


#### **Reference Links**

* .Net Core and SDK Setup.  [.Net Core, SDK, ASP](https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-debian10)
* C# HelloWorld [C# Intro](https://dotnet.microsoft.com/learn/dotnet/hello-world-tutorial/intro)
