[//]: # " Copyright (c) 2015, 2018 Oracle and/or its affiliates. All rights reserved. "
[//]: # " "
[//]: # " This program and the accompanying materials are made available under the "
[//]: # " terms of the Eclipse Distribution License v. 1.0, which is available at "
[//]: # " http://www.eclipse.org/org/documents/edl-v10.php. "
[//]: # " "
[//]: # " SPDX-License-Identifier: BSD-3-Clause "

Server Asynchronous Standalone Example
======================================

This example demonstrates JAX-RS 2.0 asynchronous API, both on the client and
server side.

The  goal of the sample is to demonstrate that with limited I/O processing threads
on the server the synchronous execution of a long-running task may lead to resource
starvation caused by I/O processing threads being blocked by the long-running
operation, unable to serve new incoming requests.

OTOH, when the same long-running operation is executed asynchronously, the I/O
threads do not need to block while waiting for the long-running operation to finish
and the overall throughput of the system is greatly increased.

The example consists of 2 modules:

  1. web application module "webapp" (produces server-async-standalone-webapp.war)

     - the module contains an long-running echo service deployed under application context root on path templates:

       - "long-running/sync/{echo}", served by a synchronously executed, blockingmethod.

       - "long-running/async/{echo}", served by an asynchronously executed, non-blocking method.

  2. client module "client" (produces client.jar)

     - the module contains the web application client developed using JAX-RS 2.0
       client API & leveraging the asynchronous client execution for spawning
       multiple simultaneous client requests with subsequent asynchronous response
       processing.

To build the example, change to the example root directory and invoke maven build:

>   mvn clean package

This command builds the web application WAR as well as client JAR artifacts. Once
built, the web application may be deployed to the application server. E.g. to deploy
the application to a running GlassFish instance, you may do (while still being in
the root example directory):

>   $AS_HOME/bin/as-admin deploy ./webapp/target/server-async-standalone-webapp.war

Once deployed, the client can be run as follows:

>   cd client
>   mvn exec:java

This will start the main client GUI window. In the top panel, the GUI lets you to customize
few variables such as number of requests to be sent, base web application URL or sync/async
endpoint to be invoked. Once the client is set up, the "Run" button runs the requests. Each
request is represented as a colored box in the lower window panel. The color indicates the
request status:

* yellow - running
* green  - successfully completed
* red    - request execution failed with an error

Also, the tooltip of each box provides additional status information. After the execution is
finished, short statistics about the run are displayed in the middle window panel.

Executing command-line client:
------------------------------
Optionally, a command line Main.java client can be executed (by changing the mainClass of
exec-maven-plugin in pom.xml to org.glassfish.jersey.examples.server.async.Main).
Following properties may be customized in the command-line client runtime:

  1. req=<n>

     : number of requests to be sent by the client, defaults to 10

  2. mode=(sync|async)

     : execution mode defines which version of the service should be used, defaults to "async"

  3. uri=<base_webapp_uri>

     : base URI of the deployed web application, defaults to "http://localhost:8080/server-async-standalone-webapp"

These properties may be passed throug maven to the client runtime as command-line arguments
using the -Dexec.args="<arguments>" flag, e.g.:

>   mvn exec:java -Dexec.args="req=20 mode=sync uri=http://foo.com/bar"

When the example client is run, successfully executed requests are output as an
asterisk '*', failed requests as an exclamation mark '!'. Any errors that occured
during the client execution are reported at the end of the run including the
statistics about the processing time and success ratio.

Try running the client using many requests and compare the results when running
against sync and async version of the resource, e.g.:

>   mvn exec:java -Dexec.args="req=100 mode=sync"
>   mvn exec:java -Dexec.args="req=100 mode=async"

You should see a significant speed improvement in the latter run.
