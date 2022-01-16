---
layout: post
author: Rizwan Minhas
title: Multiple Java setup on macOS
excerpt: A simple way to manage multiple Java versions on macOS
---

There are different ways to setup multiple Java versions on a macOS e.g. you could use [sdkman](https://sdkman.io/) or [jEnv](https://www.jenv.be/) but in this post I will show you how to do it without using any of these tools.

1. Open a terminal and type ```/usr/libexe/java_home -V```. 

    This will show you all the currently installed JVMs on your machine. In my case I have 3 JVMs installed: 
    [![multiple installed java versions](../../../assets/2022-01/multiple_java_versions.png)](../../../assets/2022-01/multiple_java_versions.png)

2. If you want to install more JVMs then install from [https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/). **Note:** You will have to create an account to install older versions like JDK 8 or 11.

3. Open your your `~/.bash_profile` or `~/.zshrc` and enter the following (_ofcourse for these may differ based on what java versions you have decided to use_):

    ```sh
    alias java17="export JAVA_HOME=$(/usr/libexec/java_home -v17)"
    alias java11="export JAVA_HOME=$(/usr/libexec/java_home -v11)"
    alias java8="export JAVA_HOME=$(/usr/libexec/java_home -v1.8.0)"
    ```    

4. Now restart the terminal or source the config file and type `java17` to set `JAVA_HOME` to version 17 or type `java8` to set it to version 8 etc.