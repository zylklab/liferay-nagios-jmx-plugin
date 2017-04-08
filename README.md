# Nagios / Icinga Plugin for Liferay Portal via JMX

## Installation

First check your JMX credentials in Liferay.

Copy next two files to your Nagios plugin directory.
 - check_jmx (Copy to /usr/lib/nagios/plugins/ - or your plugins directory) 
 - check_jmx.jar (Copy to /usr/lib/nagios/plugins/ - or your plugins directory) 
 
Then, add Liferay command definitions in Nagios plugin configuration directory.
 - /etc/icinga/objects/liferay-commands.cfg
 
The command defines the access to JMX objects, that can be obtained via Jconsole, VisualVM or JMXterm
 
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

TODO 

## Tested Setup

- Liferay 6.2
- Nagios/Icinga 3

## Links

- http://www.zylk.net/en/web-2-0/blog/-/blogs/monitorizando-liferay-portal-con-nagios (Blog Post - Spanish)
- https://exchange.nagios.org/directory/Plugins/Java-Applications-and-Servers/Syabru-Nagios-JMX-Plugin/details
- https://github.com/toniblyx/alfresco-nagios-and-icinga-plugin
