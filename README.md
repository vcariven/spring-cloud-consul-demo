# spring-cloud-consul-demo

> To reproduce : https://github.com/spring-cloud/spring-cloud-consul/issues/213 

* Get a local running consul agent (on default port `8500`)
* Get a local running spring boot configserver (on default port `8888`), and registering on consul
```
@SpringBootApplication
@EnableConfigServer
@EnableDiscoveryClient
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

## How to reproduce
* Launch `spring-cloud-consul-demo` (default port `8080`)
* You should see the following trace: `Registering service with consul: NewService{id='demo', name='demo',...`
* Stop it: nothing about unregistering is logged
```
inMXBeanRegistrar$SpringApplicationAdmin : Application shutdown requested.
ationConfigEmbeddedWebApplicationContext : Closing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@17f460bb: startup date [Tue Sep 13 13:10:28 CEST 2016]; parent: org.springframework.context.annotation.AnnotationConfigApplicationContext@3f6f6701
o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans
o.apache.catalina.core.StandardService   : Stopping service Tomcat
o.a.c.c.C.[Tomcat].[localhost].[/]       : Destroying Spring FrameworkServlet 'dispatcherServlet'
``` 

### Probable cause
* In `src/main/resources/bootstrap.yml`, switch `spring.cloud.config.discovery.enabled` to `false`
* Relaunch `spring-cloud-consul-demo` (config server addr/port is no more discovered through consul)
* You still see the following trace:
```
Registering service with consul: NewService{id='demo', name='demo',...
``` 
* Stop it.
* You should now see: `Deregistering service with consul: demo`
``` 
inMXBeanRegistrar$SpringApplicationAdmin : Application shutdown requested.
ationConfigEmbeddedWebApplicationContext : Closing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@aeab9a1: startup date [Tue Sep 13 13:08:25 CEST 2016]; parent: org.springframework.context.annotation.AnnotationConfigApplicationContext@3f6f6701
o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0
o.s.c.consul.discovery.ConsulLifecycle   : Deregistering service with consul: demo
o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans
o.apache.catalina.core.StandardService   : Stopping service Tomcat
o.a.c.c.C.[Tomcat].[localhost].[/]       : Destroying Spring FrameworkServlet 'dispatcherServlet'
``` 
