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
  <li>Oracle Version - 11g, 12c</li>
  <li>Golden Gate Version - 12.x, 19.1</li>
  <li>Java adapter for golden gate - 12.x, 19.1</li>
</ul>

The Goldengate application adapter for Java should be setup and configured. This connector has been tested with the following GG application adapter versions,
<br>
[12.2.0.1.1](https://docs.oracle.com/en/middleware/goldengate/adapter/12.2.0.1.1/index.html)
<br>
[19.1](https://docs.oracle.com/pls/topic/lookup?ctx=en/middleware/goldengate/big-data/19.1/gbdin&id=GADBD-GUID-A6C0DEC9-480F-4782-BD2A-54FEDDE2FDD9)
<br>
The installation and setup has also been tested with the built in [file handler](https://docs.oracle.com/goldengate/gg121211/gg-adapter/GADAD/flatfile_config.htm#GADAD424) and we have necessary velocity template file in the package to convert the CDC events into a XML file.

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
  <li>&nbsp;logback.xml - Logging configurations for the connector.</li>
  </ol>
  <li>ogg/extlib – Place all the libraries that are needed by the connector in this folder. This folder is provided in the classpath in
javaue.properties</li>
  <li>ogg/extlib/config – This is the main config location. Connector loads mapper config from this location. Set -Divoyant.cdc.connector.config to
this location.</li>
  <li>ogg/extlib/config/mappers – Mapping files for the connector. These Mapper file(s) defines the mapping from source (Oracle) to target
(Redis Enterprise)</li>
<details><summary>Sample mapper.xml</summary>
<p>

#### mapper configuration file.
### Sample mapper.xml under rl-connector-gg/ogg/extlib/config/mappers folder

```xml
<Schema xmlns="http://cdc.connector.ivoyant.com/Mapper/Config" name="ORCLPDB1.HR"> <!-- Schema name e.g. HR. One mapper file per schema and you can have multiple tables in the same mapper file as long as schema is same, otherwise create multiple mapper files e.g. mapper1.xml, mapper2.xml or <table_name>.xml etc. under mappers folder of your config dir.-->
    <Tables>
        <Table name="EMPLOYEES"> <!-- EMPLOYEES table under HR schema -->
            <!-- publishBefore - Global setting, that specifies if before values have to be published for all columns
 *                 - This setting could be overridden at each column level -->
            <Processors>
                <RedisProcessor id="EMPLOYEES" processorID="REDIS_HASH_PROCESSOR" publishChangedColumnsOnly="false" deleteOnKeyUpdate="true" prependTableNameToKey="true">
                    <Mapper>
                        <Column src="EMPLOYEE_ID" target="EmpNumber" type="INT"/> <!-- key column on the source emp table -->
                        <Column src="FIRST_NAME" target="FName"/>
                        <Column src="LAST_NAME" target="LName"/>
                        <Column src="JOB_ID" target="JobId"/>
                        <Column src="HIRE_DATE" target="StartDate"/>
                        <Column src="SALARY" target="Salary" type="DOUBLE"/>
                    </Mapper>
                    <RedisSink connectionId="cluster1" async="true"/>
                </RedisProcessor>
            </Processors>
        </Table>
        <Table name="EMPLOYEES_DATA">
            <Processors>
                <RedisProcessor id="EMPLOYEES_DATA" processorID="REDIS_STRIING_PROCESSOR" publishChangedColumnsOnly="false" deleteOnKeyUpdate="true" prependTableNameToKey="false">
                    <Mapper>
                        <Column src="EMPLOYEE_ID" target="EmpNumber" type="INT"/> <!-- key column on the source EMPLOYEES_DATA table -->
                        <Column src="FIRST_NAME" target="FName"/>
                        <Column src="LAST_NAME" target="LName"/>
                        <Column src="JOB_ID" target="JobId"/>
                        <Column src="HIRE_DATE" target="StartDate"/>
                        <Column src="SALARY" target="Salary" type="DOUBLE"/>
                    </Mapper>
                    <RedisSink connectionId="cluster1" async="true">
                        <Formatter>FREEMARKER_FORMATTER</Formatter>
                        <FormatterTemplate>pipe-delimited.ftl</FormatterTemplate>
                    </RedisSink>
                </RedisProcessor>
            </Processors>
        </Table>      
    </Tables>
</Schema>
      
```
If you don't need any transformation of source columns then you can simply use passThrough option and you don't need to explicitly map each source columns to Redis target data structure.
```xml
<Schema xmlns="http://cdc.connector.ivoyant.com/Mapper/Config" name="ORCLPDB1.HR"> <!-- Schema name e.g. HR. One mapper file per schema and you can have multiple tables in the same mapper file as long as schema is same, otherwise create multiple mapper files e.g. mapper1.xml, mapper2.xml or <table_name>.xml etc. under mappers folder of your config dir.-->
    <Tables>
        <Table name="EMPLOYEES"> <!-- EMPLOYEES table under HR schema -->
            <!-- publishBefore - Global setting, that specifies if before values have to be published for all columns
 *                 - This setting could be overridden at each column level -->
            <Processors>
                <RedisProcessor id="EMPLOYEES-Passthrough" processorID="REDIS_HASH_PROCESSOR" passThrough="true" publishChangedColumnsOnly="false" deleteOnKeyUpdate="true" prependTableNameToKey="true">
                    <Mapper>
                        <Column src="EMPLOYEE_ID" target="EmpNumber" type="INT"/> <!-- key column on the source emp table -->
                    </Mapper>
                    <RedisSink connectionId="cluster1" async="true"/>
                </RedisProcessor>
            </Processors>
        </Table>
    </Tables>
</Schema>
```

</p>
</details>
  <li>ogg/extlib/config/redis-connections.props – Contains redis connection details. The property key is the same that is provided as connectionId value in mapping file.</li>
</ul>

Please see [here](https://github.com/lettuce-io/lettuce-core/wiki/Redis-URI-and-connection-details#uri-syntax) for Redis URI syntax.

<br>
If you had the sample file handler working well before, this connector should work if all the configs are in the right place. Follow the steps below to start the connector
  <ul>
    <li>Start the extract by executing "start extract javaue" i.e. <i>GGSCI> START EXTRACT JAVAUE</i></li>
  </ul>
  
:information_source:
<br>You can use the command "info all" to see the _EXTRACT_ status and view its report by using "view extract javaue".
<br>Logs of the extract are also located within the /u01/app/ogg/dirrpt directory.
<br>If everything has been configured properly, you will be able to observe the data being replicated in redis.
<br>If you make any changes by replacing a jar or changing a configuration file, you must stop the extract and start it again to see the new changes.
<br>For troubleshooting look at the standard user exit log files and connector log file (dafault location is logs/cdc-1.log). Please ensure that logback jars are available in the classpath as shown in the sample `javaue.properties` under `javawriter.bootoptions`.
<br>The connector works with [EXTRACT](https://docs.oracle.com/goldengate/gg12201/gg-adapter/GADAD/GUID-4A07BBD8-5E9E-406E-9D5C-9DA3FDF1C165.htm#GADBD124) setup as shown in the sample `JAVAUE.prm` file under dirprm directory but starting with Oracle Golden Gate version 19.x, you need a [REPLICAT](https://docs.oracle.com/goldengate/gg12201/gg-adapter/GADAD/GUID-4A07BBD8-5E9E-406E-9D5C-9DA3FDF1C165.htm#GADBD117) setup. Here is a sample of REPLICAT,
<br>
```bash
REPLICAT JAVAUE
getEnv (JAVA_HOME)
getEnv (LD_LIBRARY_PATH)
getEnv (PATH)
TARGETDB LIBFILE libggjava.so SET property=dirprm/javaue.properties
STATOPTIONS RESETREPORTSTATS
REPORT AT 11:59
REPORTROLLOVER AT 00:01
REPORTCOUNT EVERY 30 MINUTES, RATE
GROUPTRANSOPS 1000
GETUPDATEBEFORES
MAP HR.*, TARGET HR.*;
```
