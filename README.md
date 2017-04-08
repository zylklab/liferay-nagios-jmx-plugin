# Nagios / Icinga Plugin for Liferay Portal via JMX

## Installation

First configure your JMX credentials in Liferay Portal via $CATALINA_OPTS variable in $TOMCAT_HOME/bin/setenv.sh (and restart your Liferay service): 
```
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote=true"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=6666"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=true"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.password.file=/opt/liferay/jmxremote.password"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.access.file=/opt/liferay/jmxremote.access"
CATALINA_OPTS="$CATALINA_OPTS -Djava.rmi.server.hostname=tomcat.zylk.net"
```

You should be able to connect and check JMX objects with Jconsole, VisualVM or JMXTerm.

In Nagios/Icinga server, copy next two files to your Nagios plugin directory.
 - check_jmx (Copy to /usr/lib/nagios/plugins/ - or your plugins directory) 
 - check_jmx.jar (Copy to /usr/lib/nagios/plugins/ - or your plugins directory) 
 
Then, add Liferay command definitions in Nagios plugin configuration directory.
 - /etc/icinga/objects/liferay-commands.cfg
 
 
```
define command {
        command_name check_jmx_HeapMemoryUsage_Used
        command_line /usr/lib/nagios/plugins/check_jmx -U service:jmx:rmi:///jndi/rmi://$HOSTADDRESS$:$ARG1$/jmxrmi -O java.lang:type=Memory -A HeapMemoryUsage -K used -username "$ARG2$" -password "$ARG3$" -w "$ARG4$" -c "$ARG5$"
}
```

```
define host{
        host_name           tomcat
        alias               tomcat
        address             tomcat.zylk.net
        parents             helios
        use                 generic-host
}
```

```
define service {
        use                 generic-service
        host_name           liferay
        service_description JVM Heap Memory Used
        check_command       check_jmx_HeapMemoryUsage_Used!9999!zylk!secret!750000000!800000000
}
```

 
Finally, configure you Liferay host and services to configure. 
- /etc/icinga/objects/hosts-icinga.cfg 
- /etc/icinga/objects/services-icinga.cfg 

## JMX Objects

Some basics in Liferay Monitoring are:

* JVM Heap Used 

* Thread Pool
  * maxThreads (static): This attribute tells us how many maximum threads
are configured. By default, 200 in our Tomcat setup.
  * currentThreadCount : This attribute tells about how many threads are created (including busy threads and idle threads)
  * currentThreadsBusy : This attribute tells us how many threads are busy in serving requests.

* Database Pool: There are many attributes exposed by the database connection pool MBean. The following are some of the basics
  * maxPoolSize (static): This attribute tells us the maximum number of connections that can be created in a database connection pool. 
  * numBusyConnections : This attribute tells us how many database connections are in use by Liferay server.
  * numConnections : This attribute tells us how many connections are created in the database connection pool (busy and idle connections).

* Cache statistics: They provide information of various attributes, but here are some of the key attributes for monitoring:
  * ObjectCount : This attribute tells us how many objects there are in the cache.
  * OnDiskHits : This attribute tells us how many requests are successful in locating objects from a filesystem-based cache. 
  * InMemoryHits : This attribute tells us how many requests are successful in retrieving objects from an in-memory cache.
  * CacheHits : This attribute tells us how many requests are successful in retrieving objects from the cache. It includes both in-memory and disk-based caches.
  * CacheMisses : This attribute tells us how many requests are unsuccessful in retrieving objects from the cache.

Some of them are included in liferay-command.cfg as examples.

## Tested Setup
- Liferay 6.2
- Nagios/Icinga 3

## Authors
- [Gustavo Fernandez](http://github.com/guszylk)
- [Cesar Capillas](http://github.com/CesarCapillas)

## Links
- http://www.zylk.net/en/web-2-0/blog/-/blogs/monitorizando-liferay-portal-con-nagios (Blog Post - Spanish)
- https://exchange.nagios.org/directory/Plugins/Java-Applications-and-Servers/Syabru-Nagios-JMX-Plugin/details
- https://snippets.syabru.ch/nagios-jmx-plugin/download.html
- https://github.com/toniblyx/alfresco-nagios-and-icinga-plugin
