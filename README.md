# rl-connector-gg
### Redis Enterprise Oracle Golden Gate Connector

## Overview
Redis Enterprise Change Data Capture (CDC) Connector for Oracle Golden Gate is a custom user exit, that makes it easier to configure and move changes from Oracle to Redis Enterprise Database or other Sinks e.g. [JMS](https://www.oracle.com/java/technologies/java-message-service.html). Below are few important features of the connector,

## Architecture

![Oracle Golden Gate Connector Redis high-level Architecture](/docs/images/oracle-gg-connector-redis.png)
<b>Oracle Golden Gate Connector Redis high-level Architecture Diagram</b>

<p></p>

![Oracle Golden Gate Connector Redis Architecture](/docs/images/oracle-gg-connector-redis-arch.png)
<b>Oracle Golden Gate Connector Redis Architecture Diagram</b>

#### Configuration driven approach
<ul>
  <li>Ability to filter tables and columns.</li>
  <li>Ability to ignore inserts, updates and deletes.</li>
  <li>Ability to filter inserts, updates and deletes.</li>
  <li>Ability to manage discrete source and target data structures by</li>
  <ol>
  <li>&nbsp;&nbsp;Transforming data during the movement of data from source schema to target schema.</li>
  <li>&nbsp;&nbsp;Handling updates to source tables by simulting corresponding deletes and inserts to target tables.</li>
  </ol>
</ul>

## Prerequisites
<ul>
  <li>Java Version - 1.8</li>
  <li>Golden Gate Version - 12.x</li>
  <li>Java adapter for golden gate - 12.x</li>
</ul>

The Goldengate application adapter for Java should be setup and configured. This connector has been tested with GG application adapter version
[12.2.0.1.1](https://docs.oracle.com/en/middleware/goldengate/adapter/12.2.0.1.1/index.html)
The installation has been tested with the built in file handler.

## Install and Setup
Download the [latest release](https://github.com/RedisLabs-Field-Engineering/rl-connector-gg/releases) e.g. ```wget https://github.com/RedisLabs-Field-Engineering/rl-connector-gg/releases/download/v1.0/rl-connector-gg-1.0.tar.gz``` and untar (tar -xvf rl-connector-gg-1.0.tar.gz) the rl-connector-gg-1.0.tar.gz archive.

All the contents would be extracted under rl-connector-gg

Contents of rl-connector-gg
<p align="left"><img src="https://github.com/RedisLabs-Field-Engineering/RedisCDC/blob/master/docs/images/rl-connector-gg-dir.png" alt="rl-connector-gg" height="450px"></p>

<ul>
  <li>ogg/dirdat – Data folder for Golden Gate. The location of this folder is provided in parameter file while configuring the exit. This location
can be anywhere that is suitable for your setup.</li>
  <li>ogg/dirprm – Parameter file for GoldenGate setup and connection</li>
  <ol>
  <li>&nbsp;JAVAUE.PRM - In this setup our UserExit is called JAVAUE and the parameter file iis called JAVAUE.PRM</li>
  <li>&nbsp;javaue.properties - This property file sets connector system properties and classpath. Please pay attention to the below two lines and make sure the handler (in this example it is redis) is added to the list of handlers in javaue.properties.
    <br><b>&nbsp;&nbsp;gg.handler.redis.type=</b>com.ivoyant.cdc.connector.ogg.GGExit</br>
    <br><b>&nbsp;&nbsp;javawriter.bootoptions=</b>-Xmx1024m -Xms512m -Dlogback.configurationFile=dirprm/logback.xml -Divoyant.cdc.connector.config=/u01/app/ogg/extlib/config -Djava.class.path=ggadapter/12.2.0.1/ggjava/ggjava.jar:ggadapter/12.2.0.1/ggjava/resources/lib/optional/logback/logback-classic.jar:ggadapter/12.2.0.1/ggjava/resources/lib/optional/logback/logback-core.jar:/dirprm:.</br></li>
  </ol>
  <li>ogg/extlib – Place all the libraries that are needed by the connector in this folder. This folder is provided in the classpath in
javaue.properties</li>
  <li>ogg/extlib/config – This is the main config location. Connector loads mapper config from this location. Set -Divoyant.cdc.connector.config to
this location.</li>
  <li>ogg/extlib/config/mappers – Mapping files for the connector. These Mapper file tell which columns should be mapped from source (Oracle) to target
(Redis Enterprise)</li>
  <li>ogg/extlib/config/redis-connections.props – Contains redis connection details. The property key is the same that is provided as connectionId value in mapping file.</li>
</ul>

<p>
  If you had the sample file handler working well before, this connector should work if all the configs are in the right place. Follow the steps below to start the connector
  <ul>
    <li>Start the extract by executing "start extract javaue" i.e. <i>GGSCI> START EXTRACT JAVAUE</i></li>
    <br>You can use the command "info all" to see the extracts status and you can view its reports by using "view extract javaue".
    <br>Logs of the extract are also located within the /u01/app/ogg/dirrpt directory within the container.
    <br>If everything has been configured properly, you will be able to observe the data being replicated in redis.
    <br>If you make any changes by replacing a jar or changing a configuration file, you must stop the extract and start it again to see the new changes.
    <br>For troubleshooting look at the standard userexit log files.
  </ul>
</p>
