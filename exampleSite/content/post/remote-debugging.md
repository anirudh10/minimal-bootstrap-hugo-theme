+++
lastmod = 2021-04-18T13:00:00Z
publishdate = 2021-04-18T13:00:00Z
tags = ["debugging", "remote", "java"]
title = "Remote Debugging"

+++
## Use Case

Imagine you have a java prod instance & you want to debug a request flow.

## Thinking Process

There are two ways of doing it - either add debug statements(& keep restarting) or perform remote debugging.

Let's look at remote debugging.

## The How to

#### On the remote server

1. Open /etc/init.d/your-java-app and add DEBUG1=" -Xdebug -Xrunjdwp:transport=dt_socket,address=33114,server=y,suspend=n". 
2. Start it - sudo service your-java-app restart 

#### On your local machine

1. Create a tunnel between your local and the prod instance using "ssh 10.0.0.111 -L 33114:localhost:33114"
2. Now open Intellij and add a "Remote JVM Debug" configuration with 33114 port. Select appropriate JDK Version. 
3. Start it. 
4. Add a breakpoint to the request flow. 
5. Hit the remote server with a request - http://10.0.0.111:3000/endpoint?pid=781
6. See the breakpoint is being hit.

That's all.