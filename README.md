# AppDynamics Apache - Monitoring Extension

This extension works only with the standalone machine agent.

##Use Case

The Apache HTTP Server is a widely-used web server supported by the Apache Software Foundation. The Apache HTTP Server monitoring extension captures metrics from an Apache web server and displays them in the AppDynamics Metric Browser.

This extension is capable of fetching metrics from most of the apache based HTTP servers. Tested the extension with IBM HTTP server 7.0 and Oracle HTTP Server 12.1.2. 

Metrics include:

   * Availability: Percentage of time that the server has been up; graphs server up/down status over time.
   * Health Check: Most recent and most severe logs, errrors, etc.
   * Activity: Throughput over the last minute including requests, bytes, etc.
   * Resource Utilization: Resources (threads, CPU, memory) currently in use and still available.
   * Efficiency: Server optimizations to maximize throughput such as caches, etc.

In addition, it lists:

   * Top Requests: Most requests by quantity and by volume, measured by number of bytes transferred.
   * Top Activity: Current activity such as responding, cleaning up, logging, etc.

##Prerequisite

Please enable mod_status on the HTTP server to get stats.


##Installation
1. Install Apache mod_status on your Apache instance. For more information, see [Apache Module mod_status](http://httpd.apache.org/docs/2.0/mod/mod_status.html).

2. Use `curl` to verify that the URL works: http://your-apache-server:90/server-status?auto

   ```
   > curl http://localhost:90/server-status?auto
    Total kBytes: 3
    Total Accesses: 3
    Total kBytes: 3
    Uptime: 119503
    ReqPerSec: 2.5104e-5
    BytesPerSec: .0257065
    BytesPerReq: 1024
    BusyWorkers: 1
    IdleWorkers: 7
    Scoreboard: __W___……………….
   ```

3. Run `maven clean install`. Deploy the ApacheMonitor.zip file found in 'target' into the \<machine agent home\>/monitors directory.

   ```
> cd <machine agent home>/monitors/
> unzip ApacheMonitor.zip
   ```
4. Set up monitor.xml with the correct host and port:
   -   protocol=http/https
   -   host=your-apache-server
   -   port=90
   -   proxy-host=proxy host address if any
   -   proxy-port=proxy port if any

   Note: If you want to monitor more than one server, see [Monitoring multiple Apache servers](#Monitoring multiple Apache servers).  

​5. Restart the Machine Agent.

Output from this monitoring extension includes:

-   Counter Description CPU Load: The percentage of the CPU used by Apache.

-   Requests/Sec: The number of HTTP requests the web server is processing per second.

-   Bytes/Sec: The amount of data the web server is transferring per second.

-   Bytes/Requests Average: The average number of bytes being transferred per HTTP request.

-   Busy Workers: The number of Apache processes actively processing an HTTP request.

-   Idle Workers: The number of idle Apache processes waiting for an HTTP request.

### Monitoring multiple Apache servers
Currently, the Apache HTTP Server monitoring extension only supports a single Apache server.
However, you can "fool" the system into monitoring multiple servers as follows.

​1. For each server you want to monitor, copy the monitors/ApacheMonitor directory into another directory, such as monitors/ApacheStatusMonitor2.

​2. Edit the monitor.xml file in that directory, changing the `<name>`, protocol, host, port and metric-prefix values accordingly:


   ```  
<monitor>
	<name>ApacheStatusMonitor2</name>
	<type>managed</type>
	<description>Monitors general status of Apache Server 
	</description>
    <monitor-configuration>
    </monitor-configuration>
	<monitor-run-task>
	      <execution-style>periodic</execution-style>
	      <execution-frequency-in-seconds>60</execution-frequency-in-seconds>
	      <name>Apache Status Monitor Run Task</name>
    	  <display-name>Apache Status Monitor Task</display-name>
    	  <description>Apache Status Monitor Task</description>
    	  <type>java</type>
    	  <execution-timeout-in-secs>120</execution-timeout-in-secs>
    	  <task-arguments>
    	                    <argument name="protocol" is-required="true" default-value="http"/>
	        	    <argument name="host" is-required="true" default-value="localhost"/>
		            <argument name="port" is-required="true" default-value="8092"/>
		            <argument name="proxy-host" is-required="false" default-value="localhost"/>
		            <argument name="proxy-port" is-required="false" default-value="8888"/>
		            <argument name="custom-url-path" is-required="false" default-value="/server-status?auto"/>
		            <argument name="metric-prefix" is-required="false" default-value="Custom Metrics|WebServer|Apache2|Status|"/>
    	  </task-arguments>
    	  <java-task>
          	<classpath>ApacheMonitor.jar</classpath>
          	<impl-class>com.appdynamics.monitors.apache.ApacheStatusMonitor
          	</impl-class>
          </java-task>
	</monitor-run-task>
</monitor>
   ```
​
<br/>
   
​4. When you have finished adding all the servers you want to monitor, restart the machine agent.



##Directory Structure

<table><tbody>
<tr>
<th align = 'left'> Directory/File </th>
<th align = 'left'> Description </th>
</tr>
<tr>
<td class='confluenceTd'> src/main/resources/config </td>
<td class='confluenceTd'> Contains the monitor.xml </td>
</tr>
<tr>
<td class='confluenceTd'> src/main/java </td>
<td class='confluenceTd'> Contains source code to the Apache monitoring extension </td>
</tr>
<tr>
<td class='confluenceTd'> target </td>
<td class='confluenceTd'> Only obtained when using maven. Run 'maven clean install' to get the distributable .zip file. </td>
</tr>
<tr>
<td class='confluenceTd'> pom.xml </td>
<td class='confluenceTd'> maven build script to package the project (required only if changing Java code) </td>
</tr>
</tbody>
</table>


##Metrics

###Availability

-   Uptime (1 or 0)

###Resource Utilization

-   Counter Description CPU Load (N/A on Windows) -- The percentage of the CPU used by Apache.
-   Processes
    - Busy Workers -- The number of Apache processes actively processing an HTTP request.
    - Idle Workers -- The number of idle Apache processes waiting for an HTTP request.
-   Memory

###Activity

-   Accesses -- Total number of accesses per Minute
-   Total Traffic (kb)
-   Requests per second -- The number of HTTP requests the web server is processing per second.
-   Bytes per second -- The amount of data the web server is transferring per second.
-   Bytes per request -- The average number of bytes being transferred per HTTP request.
-   Activity Types
    -   Starting up
    -   Reading Request
    -   Sending Reply
    -   Keepalive
-   DNS Lookup
-   Closing Connection
-   Logging
-   Gracefully Finishing
-   Cleaning up of working

##Load balancing metrics

In addition to the above specified metrics, this extension can also show metrics from mod_jk status. To do this we have to configure mod_jk in the apache HTTP server.
More info on mod_jk at http://tomcat.apache.org/connectors-doc/

###example configuration

####in httpd.conf
   ```   
	LoadModule    jk_module  modules/mod_jk.so
	JkWorkersFile conf/workers.properties
	JkShmFile     /var/log/httpd/mod_jk.shm
	JkLogFile     /var/log/httpd/mod_jk.log
	JkLogLevel    info
	JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "
	JkMount  /examples/* loadbalancer
	
	<Location /status>
	    JkMount statusmanager
	    Order deny,allow
	    Allow from localhost
	</Location>

   ```
####worker.properties file	
   ```
	worker.list=worker1,worker2,loadbalancer,statusmanager
	
	#worker1
	worker.worker1.type=ajp13
	worker.worker1.host={host1}
	worker.worker1.port={host1 ajp post}
	worker.worker1.lbfactor=100   
	
	#worker2
	worker.worker2.type=ajp13
	worker.worker2.host={host2}
	worker.worker2.port={host2 ajp port}
	worker.worker2.lbfactor=100   
	
	#Load balancer
	worker.loadbalancer.type=lb
	worker.loadbalancer.balance_workers=worker1,worker2
	worker.loadbalancer.sticky_session=1
	
	#status manager
	worker.statusmanager.type=status
	
   ```
####monitor.xml
   ```
       <argument name="jk-status-path" is-required="false" default-value="/status"/>
       <argument name="jk-worker-stats" is-required="false" default-value="connection_pool_timeout,ping_timeout,connect_timeout,prepost_timeout,reply_timeout,retries,connection_ping_interval,recovery_options,max_packet_size,activation,lbfactor,distance,lbmult,lbvalue,elected,sessions,errors,client_errors,reply_timeouts,transferred,read,busy,max_busy,connected,time_to_recover_min,time_to_recover_max,used,map_count,last_reset_ago"/>
        
   ```
   `jk-status-path` is the url path defined in the Location section of httpd.conf.<br/>
   `jk-worker-stats` list of worker stats.

##Contributing

Always feel free to fork and contribute any changes directly via [GitHub](https://github.com/Appdynamics/apache-monitoring-extension).

##Community

Find out more in the [AppSphere](http://appsphere.appdynamics.com/t5/Extensions/Apache-Monitoring-Extension/idi-p/753) community.

##Support

For any questions or feature request, please contact [AppDynamics Center of Excellence](mailto:help@appdynamics.com).
