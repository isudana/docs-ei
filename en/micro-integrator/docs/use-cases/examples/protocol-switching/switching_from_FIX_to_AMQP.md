# Switch from FIX to AMQP

This example demonstrates how WSO2 Micro Integrator receives messages through FIX and forwards them through AMQP.

Synapse will forward the order request by binding it to a JMS message payload and sending it to the AMQP consumer. AMQP consumer will send a execution back to Banzai.

## Synapse configuration

Following are the integration artifacts (proxy service) that we can used to implement this scenario.

```xml 
<proxy name="FIXProxy" transports="fix">
    <target>
        <endpoint>
            <address uri="jms:/QpidStockQuoteService?transport.jms.ConnectionFactoryJNDIName=qpidConnectionfactory&amp;java.naming.factory.initial=org.apache.qpid.jndi.PropertiesFileInitialContextFactory&amp;java.naming.provider.url=repository/samples/resources/fix/con.properties&amp;transport.jms.ReplyDestination=replyQueue"/>
        </endpoint>
        <inSequence>
            <log level="full" />
        </inSequence>
        <outSequence>
            <property name="transport.fix.ServiceName" value="FIXProxy" scope="axis2-client" />
            <log level="full" />
            <send />
        </outSequence>
    </target>
    <parameter name="transport.fix.AcceptorConfigURL">
        file:repository/samples/resources/fix/fix-synapse.cfg
    </parameter>
    <parameter name="transport.fix.AcceptorMessageStore">file</parameter>
</proxy>
```

<!--
## Configuring Sample FIX Applications

If you use a binary distribution of Quickfix/J, the two samples and
their configuration files are all packed to a single JAR file called
`         quickfixj-examples.jar        ` . You will have to extract the
JAR file, modify the configuration files and pack them to a JAR file
again under the same name.

You can pass the new configuration file as a command line parameter too,
in that case you do not need to modify the
`         quickfixj-examples.jar        ` . You can copy the config
files from `         $ESB_HOME/repository/samples/resources/fix        `
folder to `         $QFJ_HOME/etc folder        ` . Execute the sample
applications from `         $QFJ_HOME/bin        ` ,
`         ./banzai.sh/bat ../etc/banzai.cfg executor.sh/bat ../etc/executor.cfg        `
.

Locate and edit the FIX configuration file of Executor to be as follows.
This file is usually named `         executor.cfg        ` .

```java
[default]
    FileStorePath=examples/target/data/executor
    ConnectionType=acceptor
    StartTime=00:00:00
    EndTime=00:00:00
    HeartBtInt=30
    ValidOrderTypes=1,2,F
    SenderCompID=EXEC
    TargetCompID=SYNAPSE
    UseDataDictionary=Y
    DefaultMarketPrice=12.30

    [session]
    BeginString=FIX.4.0
    SocketAcceptPort=19876
```

Locate and edit the FIX configuration file of Banzai to be as follows.
This file is usually named `         banzai.cfg        ` .

```java
[default]
    FileStorePath=examples/target/data/banzai
    ConnectionType=initiator
    SenderCompID=BANZAI
    TargetCompID=SYNAPSE
    SocketConnectHost=localhost
    StartTime=00:00:00
    EndTime=00:00:00
    HeartBtInt=30
    ReconnectInterval=5

    [session]
    BeginString=FIX.4.0
    SocketConnectPort=9876
```

The `FileStorePath` property in the above two files should point to two directories in your local file system. The launcher
scripts for the sample application can be found in the bin directory of Quickfix/J distribution.

## Configuring WSO2 Micro Integrator for FIX Samples

Create the FIX configuration files as specified below. You can find the config files from "$MI_HOME/repository/samples/resources/fix" folder.

The `FileStorePath` property in the following two files should point to two directories in your local file system. Once the samples are executed, Synapse will create FIX message stores in these two directories.

```java tab='fix-synapse.cfg'
[default]
    FileStorePath=repository/logs/fix/data
    ConnectionType=acceptor
    StartTime=00:00:00
    EndTime=00:00:00
    HeartBtInt=30
    ValidOrderTypes=1,2,F
    SenderCompID=SYNAPSE
    TargetCompID=BANZAI
    UseDataDictionary=Y
    DefaultMarketPrice=12.30

    [session]
    BeginString=FIX.4.0
    SocketAcceptPort=9876
```

```java tab='synapse-sender.cfg'
[default]
    FileStorePath=repository/logs/fix/data
    SocketConnectHost=localhost
    StartTime=00:00:00
    EndTime=00:00:00
    HeartBtInt=30
    ReconnectInterval=5
    SenderCompID=SYNAPSE
    TargetCompID=EXEC
    ConnectionType=initiator

    [session]
    BeginString=FIX.4.0
    SocketConnectPort=19876
```

## Configure ESB for AMQP Transport

The samples used in this guide assumes the existence of a local QPid
(1.0-M2 or higher) installation properly installed and started up. You
also need to copy the following client JAR files into the
`         repository/components/lib        ` directory to support AMQP.
These files are found in the `         lib        ` directory of the
QPid installation.

```java
qpid-client-1.0-incubating-M2.jar
    qpid-common-1.0-incubating-M2.jar
    geronimo-jms_1.1_spec-1.0.jar
    slf4j-api-1.4.0.jar **
    slf4j-log4j12-1.4.0.jar **
```

!!! Note
    To configure FIX (Quickfix/J 1.3) with AMQP (QPid-1.0-M2), copy the "sl4j" - libraries comes with QPid and ignore the "sl4j" - libraries that come with Quickfix/J.

To enable the AMQP over JMS transport, you need to uncomment the JMS
transport listener configuration. To enable AMQP over JMS for ESB, the
`         repository/conf/axis2.xml        ` must be updated, while to
enable JMS support for the sample Axis2 server the
`         samples/axis2Server/repository/conf/axis2.xml        ` file
must be updated.

Locate and edit the AMQP connection settings file for the message consumer, this file is usually named `direct.properties` and can be found in the `repository/samples/resources/fix` directory.

```java
java.naming.factory.initial = org.apache.qpid.jndi.PropertiesFileInitialContextFactory
# register some connection factories
# connectionfactory.[jndiname] = [ConnectionURL]
connectionfactory.qpidConnectionfactory = amqp://guest:guest@clientid/test?brokerlist='tcp://localhost:5672'
# Register an AMQP destination in JNDI
# destination.[jniName] = [BindingURL]
destination.directQueue = direct://amq.direct//QpidStockQuoteService?routingkey='QpidStockQuoteService'
destination.replyQueue = direct://amq.direct//replyQueue?routingkey='replyQueue'
```

Locate and edit the AMQP connection settings file for WSO2 ESB, this
file is usually named `         con.properties        ` and can be found
in the `         repository/samples/resources/fix        ` directory.

```java
#initial context factory
#java.naming.factory.initial =org.apache.qpid.jndi.PropertiesFileInitialContextFactory
# register some connection factories
# connectionfactory.[jndiname] = [ConnectionURL]
connectionfactory.qpidConnectionfactory=amqp://guest:guest@clientid/test?brokerlist='tcp://localhost:5672'
# Register an AMQP destination in JNDI
# destination.[jndiName] = [BindingURL]
destination.directQueue=direct://amq.direct//QpidStockQuoteService
```
## Build and run

-   You will need the sample FIX blotter that comes with Quickfix/J
    (Banzai). Configure the blotter to establish sessions with Synapse.
-   Configure the AMQP transport for WSO2 ESB.
-   Start the AMQP consumer, by switching to
    `           samples/axis2Client          ` directory and running the
    consumer using the following command. Consumer will listen to the
    queue `           QpidStockQuoteService          ` , accept the
    orders and reply to the queue `           replyQueue          ` .

    ```bash
    ant amqpconsumer \-Dpropfile=$ESB_HOME/repository/samples/resources/fix/direct.properties
    ```

-   Start Banzai.
-   Enable FIX transport in the Synapse `          axis2.xml         ` .
-   Configure Synapse for FIX samples.
-   Open up the
    `           ESB_HOME/repository/samples/synapse_sample_260.xml          `
    file and make sure that the
    `           transport.fix.AcceptorConfigURL          ` property
    points to the `           fix-synapse.cfg          ` file you
    created.

    !!! Note
        Synapse creates a new FIX session with Banzai at this point.

-   Send an order request from Banzai to Synapse. For example, Buy DELL
    1000 @ MKT.
-->