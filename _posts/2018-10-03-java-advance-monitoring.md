---
layout: post
title: "Advance monitoring and performance tuning in Java"
author: pratyush
categories: [ Tuning, Tutorial ]
image: assets/images/java-advance-monitoring.webp
description: "Monitoring and performance tuning in Java"
hidden: true
---

Java advance monitoring and performance tuning settings to be applied in Java Application Server startup files. Works for both Tomcat and Wildfly.

### 1. Java OPTS for JMX
Java Management Extensions (JMX) is a Java technology that supplies tools for managing and monitoring applications, system objects etc. Those resources are represented by objects called MBeans (also called Managed Bean).

```java
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=6000 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.rmi.port=6001 
-Djava.rmi.server.hostname=192.168.1.1
```
Once the settings are in place, we can use JConsole tool to monitor the server.

![Remote Connection](https://docs.oracle.com/javase/7/docs/technotes/guides/management/figures/connectrem.gif)

Overview Tab to get metrics like Heap, Running Threads and Classes loaded

![Overview Tab](https://docs.oracle.com/javase/7/docs/technotes/guides/management/figures/overviewtab.gif)

> Link to Original Article [Oracle Java 7 Docs](https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html)

### 2. Java OPTS for GC Log
Garbage Collection GC tuning is the process of adjusting the startup parameters of your JVM-based application to match the desired results. It’s crucial for the garbage collector to work as efficiently as possible. The GC process can be heavy. Settings for capturing Garbage collection logs is mentioned below.

```java
-verbose:gc -Xloggc:/home/centos/gc.log.'date +%Y%m%d%H%M%S' -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime
```

Once the log is captured, we can use [GCEasy.io](https://gceasy.io/) to analyse the output.

![Consecutive GC](https://gceasy.io/assets/gc-recommendations/Consecutive%20full%20GC.png)

> Link to Original Article [GCEasy.io](https://gceasy.io/gc-recommendations/consecutive-full-gc-solution.jsp)

### 3. Java OPTS for Heap Dump
Modern applications waste memory because of several reasons: Duplicate String, Duplicate Objects, Inefficient Collections etc. Heap dump can be used to detects those inefficiencies.

```java
-d64 -server -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heap -Djava.util.Arrays.useLegacyMergeSort=true
```

Once the log is captured, we can use [Heaphero.io](https://heaphero.io/) to analyse the output.

![Heap Statistics](https://blogheaphero.files.wordpress.com/2018/04/heap-statistics.png)

![Memory by class](https://blogheaphero.files.wordpress.com/2018/04/whats-in-your-memory.png)

> Link to Original Article [Heaphero.io](https://blog.heaphero.io/2018/04/13/heaphero-user-manual-2/)

### 4. Thread dump
A thread dump is a snapshot of the state of all the threads of a Java process. The state of each thread is presented with a stack trace, showing the content of a thread's stack. A thread dump is useful for diagnosing problems as it displays the thread's activity. Thread dumps are written in plain text, so we can save their contents to a file and look at them later in a text editor.

#### Thread dump using `jstack`
```
jstack <java-process-id> > /tmp/threaddump.txt
```
Once the thread dump is captured in a file, we can use [FastThread.io](https://fastthread.io/) to analyse the output.

![Thread Count](https://fastthread.io/assets/images/ft-screenshots/thread-count-summary.PNG)

![Stack Trace](https://fastthread.io/assets/images/ft-screenshots/repeating-stack-traces.PNG)

![Transitive graph](https://fastthread.io/assets/images/ft-screenshots/transitive-graph.PNG)

![Waiting Threads](https://fastthread.io/assets/images/ft-screenshots/waiting-threads.PNG)

> Link to Original Article [FastThread.io](https://fastthread.io/how-to-analyze-thread-dumps-faq.jsp)