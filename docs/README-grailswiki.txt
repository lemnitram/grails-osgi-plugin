h1. Grails OSGi plugin

h2. About

The Grails OSGi plugin adds support to package and run a Grails application as an OSGi bundle. The bundle(s) may be run in an ad hoc assembled OSGi container or deployed to an external OSGi app server such as "SpringSource DM Server":http://www.springsource.org/dmserver, "Eclipse Virgo":http://eclipse.org/virgo/, or "Apache Karaf":http://karaf.apache.org/.

See also "Grails on OSGi":http://blog.jetztgrad.net/grails-on-osgi/ for details on running Grails applications in an OSGi environment.

h2. Source code and Issues

The source code is hosted at [GitHub|http://github.com/jetztgradnet/grails-osgi-plugin], issues can be reported [here|http://github.com/jetztgradnet/grails-osgi-plugin/issues].

h2. License

The OSGi plugin is released under the [Apache License 2.0|http://apache.org/licenses/LICENSE-2.0.txt].

h2. Installation

Simply call @grails install-plugin osgi@ to install the OSGi plugin

h2. Usage

h3. Creating an OSGi bundle from the Grails application

{code}
grails bundle

grails prod bundle
{code}

h3. Running the bundle

{code}
grails run-bundle

grails prod run-bundle
{code}

The application can be accessed at [http://localhost:8080/myapp-0.1/|http://localhost:8080/myapp-0.1/].

*Note*: at first start the OSGi runtime is assembled, which may take some time, while Ivy is downloading the Internet...

*Note*: if the bundle exists, it won't be re-created automatically after changes. So for now use parameter @-forceBundle@ after changing something in the Grails app: @grails run-bundle -forceBundle@ .


h3. Accessing the bundle context

The "bundle context":http://www.osgi.org/javadoc/r4v42/org/osgi/framework/BundleContext.html is the primary interface to the OSGi framework. It can be used to find and register services and interact the environment.

The bundle context is available from Grails' parent ApplicationContext as bean @bundleContext@ . It can be injected into all Spring-managed beans, i.e. all Grails artifacts, like Controllers, Services, etc.

In order to get a reference to the BundleContext, an artifact needs to define a reference with the name @bundleContext@ .

Example: controller accessing bundle context

{code}
class OsgiTestController {
    def bundleContext
    def index = {
        def bundles = []
        if (bundleContext) {
            bundles = bundleContext.bundles
        }
        else {
            flash.message = "bundleContext NOT available"
        }
        [ bundles: bundles ]
    }
}
{code}

h2. Deployment

Automatic deployment is not yet implemented. In order to deploy the OSGi-fied Grails application, the generated bundle (i.e. the war file @target/myapp-version.war@ ) and all required bundles need to installed in your OSGi application server. A list of required bundles with download URLs can be generated with @grails list-bundles@ . OSGi application servers, such as SpringSource dmServer, Apache Karaf, or Eclipse Virgo, usually offer a pickup directory, which can be used to hot deploy bundles. Simply drop the war file and all dependencies (if not installed otherwise) into this directory and you are good to go.

Later versions of the OSGi plugin will automate this process.

h2. Roadmap

See also [here|http://github.com/jetztgradnet/grails-osgi-plugin/issues] for issues and feature requests.

* make bundle generation configurable (e.g. include/exclude dependencies, ...)
* make OSGi runtime created by @grails run-bundle@ and @grails assemble-osgi-runtime@ configurable
* support auto-reloading of changed artifacts
* create sub class of GrailsApplicationContext, which implements [ConfigurableOsgiBundleApplicationContext|http://static.springsource.org/osgi/snapshot-site/apidocs/org/springframework/osgi/context/ConfigurableOsgiBundleApplicationContext.html]
* export main Grails beans and application context as OSGi service
* export services as OSGi service (via @static expose = 'osgi'@ )
* implement deployment to [SpringSource DM Server|http://www.springsource.org/dmserver] or its successor [Eclipse Virgo|http://eclipse.org/virgo/]
* implement deployment to [Apache Karaf|http://karaf.apache.org/]
* Spring DM has been donated to Eclipse, so it should be replaced by its successor [Eclipse Gemini|http://eclipse.org/gemini/]


h2. History

h3. 0.2 (2010-07-11)

* using Spring DM instead of Pax Web as Web extender
* updated Apache Felix WebConsole
* replaced (most) dependencies from bundle by OSGi bundles from the "SpringSource Enterprise Repository":http://www.springsource.com/repository
* provide access to OSGi @BundleContext@ to artifacts (controllers, services, ...) via Spring injection as bean @bundleContext@ .

h3. 0.1 (2010-01-03)

* initial version
* uses PAX Runner to run osgi-fied Grails app
* Grails app is war'ed and extended with required OSGi bundle headers to create a single, big, monolithic bundle

h2. How it works

In order to be a valid OSGi bundle, the application is war'ed and provided with the necessary bundle manifest headers (see @scripts/_Events.groovy@ for details).
The bundle can be created using @grails bundle@ .  

The OSGi runtime is assembled in the @target/osgi@ directory and uses the [Eclipse Equinox|http://eclipse.org/equinox/] OSGi framework and [Spring DM|http://www.springsource.org/osgi] with an embedded Jetty servlet container.

In the current version, most libraries from @WEB-INF/lib@ are replaced by equivalent OSGi bundles from the [SpringSource Enterprise Bundle Repository|http://www.springsource.com/repository/app/] (EBR). Only Hibernate and Grails libs retain in the bundle. 

Grails jars already are OSGi bundles, but they cannot currently be installed cleanly in a OSGi container, as they contain circular dependencies and some other dependencies can not be satisfied (e.g. the Radeox library used by the Grails Wiki is no longer maintained, but it shouldn't be referenced from the Grails runtime anyway...). 

There are bundles for Hibernate but due to a bug (possibly in Equinox), Hibernate Cache can not be resolved by the Spring ORM module (see [SPR-7003|https://issues.springsource.org/browse/SPR-7003] for a description of this problem).

The parent ApplicationContext has been replaced in @web.xml@ by an OSGi-aware variant: [OsgiBundleXmlWebApplicationContext|http://static.springsource.org/osgi/snapshot-site/apidocs/org/springframework/osgi/web/context/support/OsgiBundleXmlWebApplicationContext.html]. It knows how to [export|http://static.springsource.org/osgi/docs/2.0.0.M1/reference/html/service-registry.html#service-registry:export] and [import|http://static.springsource.org/osgi/docs/2.0.0.M1/reference/html/service-registry.html#service-registry:refs] OSGi services as/from Spring beans through Spring XML files. These features are currently hidden, but they will be exposed in the next version of the OSGi plugin and made easier through the BeanBuilder DSL and the [OSGi namespace|http://static.springsource.org/osgi/docs/2.0.0.M1/reference/html/service-registry.html].

h2. More docs

For more docs see the [wiki|http://wiki.github.com/jetztgradnet/grails-osgi-plugin] at the GitHub site or [my blog|http://blog.jetztgrad.net/category/osgi/].

h2. Getting around the OSGi runtime

h3. Web Console

The [Felix Web Management Console|http://felix.apache.org/site/apache-felix-web-console.html] provides excellent insight into the inner workings (see Screenshots tab for some examples). It can be accessed at [http://localhost:8081/system/console/|http://localhost:8081/system/console/] with user "admin" and password "admin" (*Note*: the web console runs on a different port!). 

h3. Shell Console

The command @grails run-bundle@ drops the user in the Equinox Shell (press @RETURN@ if you don't see the @osgi>@ prompt). Running @grails run-bundle -remoteConsole [port]@ opens a console with telnet access on the specified port or on port @8023@ , if omitted. Only a single user can use the console at any time.

Typing @help@ shows the available commands:

{code}
osgi> help
---Controlling the OSGi framework---
	launch - start the OSGi Framework
	shutdown - shutdown the OSGi Framework
	close - shutdown and exit
	exit - exit immediately (System.exit)
	init - uninstall all bundles
	setprop <key>=<value> - set the OSGi property
---Controlling Bundles---
	install - install and optionally start bundle from the given URL
	uninstall - uninstall the specified bundle(s)
	start - start the specified bundle(s)
	stop - stop the specified bundle(s)
	refresh - refresh the packages of the specified bundles
	update - update the specified bundle(s)
---Displaying Status---
	status [-s [<comma separated list of bundle states>]  [<segment of bsn>]] - display installed bundles and registered services
	ss [-s [<comma separated list of bundle states>]  [<segment of bsn>]] - display installed bundles (short status)
	services [filter] - display registered service details
	packages [<pkgname>|<id>|<location>] - display imported/exported package details
	bundles [-s [<comma separated list of bundle states>]  [<segment of bsn>]] - display details for all installed bundles
	bundle (<id>|<location>) - display details for the specified bundle(s)
	headers (<id>|<location>) - print bundle headers
	log (<id>|<location>) - display log entries
---Extras---
	exec <command> - execute a command in a separate process and wait
	fork <command> - execute a command in a separate process
	gc - perform a garbage collection
	getprop  [ name ] - displays the system properties with the given name, or all of them.
---Controlling Start Level---
	sl [<id>|<location>] - display the start level for the specified bundle, or for the framework if no bundle specified
	setfwsl <start level> - set the framework start level
	setbsl <start level> (<id>|<location>) - set the start level for the bundle(s)
	setibsl <start level> - set the initial bundle start level
---Controlling the Profiling---
	profilelog - Display & flush the profile log messages
---Eclipse Runtime commands---
	diag - Displays unsatisfied constraints for the specified bundle(s).
	enableBundle - enable the specified bundle(s)
	disableBundle - disable the specified bundle(s)
	disabledBundles - list disabled bundles in the system
---Controlling the Console---
	more - More prompt for console output
{code}

The command @ss@ (for short status) shows all bundles with their respective state:

{code}
osgi> ss

Framework is launched.

id	State       Bundle
0	ACTIVE      org.eclipse.osgi_3.6.0.v20100517
1	ACTIVE      org.eclipse.osgi.util_3.2.100.v20100503
2	ACTIVE      org.eclipse.osgi.services_3.2.100.v20100503
3	ACTIVE      org.eclipse.equinox.common_3.6.0.v20100503
4	ACTIVE      org.apache.felix.configadmin_1.2.4
5	ACTIVE      org.apache.felix.fileinstall_2.0.8
6	ACTIVE      com.springsource.org.apache.log4j_1.2.15
7	ACTIVE      org.ops4j.pax.logging.pax-logging-api_1.4.0
8	ACTIVE      org.ops4j.pax.logging.pax-logging-service_1.4.0
9	ACTIVE      com.springsource.javax.annotation_1.0.0
10	ACTIVE      com.springsource.javax.el_1.0.0
11	ACTIVE      com.springsource.javax.ejb_3.0.0
12	ACTIVE      com.springsource.javax.mail_1.4.1
13	ACTIVE      com.springsource.javax.persistence_1.99.0
14	ACTIVE      com.springsource.javax.transaction_1.1.0
15	ACTIVE      com.springsource.javax.servlet_2.5.0
16	ACTIVE      com.springsource.javax.servlet.jsp_2.1.0
17	ACTIVE      com.springsource.javax.servlet.jsp.jstl_1.2.0
18	ACTIVE      com.springsource.javax.jms_1.1.0
19	ACTIVE      com.springsource.javax.xml.rpc_1.1.0
20	ACTIVE      com.springsource.org.mortbay.jetty.server_6.1.9
21	ACTIVE      com.springsource.org.mortbay.util_6.1.9
22	ACTIVE      org.springframework.osgi.jetty.start.osgi_1.0.0
23	RESOLVED    org.springframework.osgi.jetty.web.extender.fragment.osgi_1.0.1
	            Master=85
24	ACTIVE      org.apache.felix.http.jetty_2.0.4
25	ACTIVE      com.springsource.org.apache.commons.beanutils_1.8.0
26	ACTIVE      com.springsource.org.apache.commons.collections_3.2.1
27	ACTIVE      com.springsource.org.apache.commons.codec_1.3.0
28	ACTIVE      com.springsource.org.apache.commons.dbcp_1.2.2.osgi
29	ACTIVE      com.springsource.org.apache.commons.el_1.0.0
30	ACTIVE      com.springsource.org.apache.commons.digester_1.8.1
31	ACTIVE      com.springsource.org.apache.commons.fileupload_1.2.1
32	ACTIVE      com.springsource.org.apache.commons.httpclient_3.1.0
33	ACTIVE      com.springsource.org.apache.commons.io_1.4.0
34	ACTIVE      com.springsource.org.apache.commons.lang_2.4.0
35	ACTIVE      com.springsource.org.apache.commons.pool_1.5.3
36	ACTIVE      com.springsource.org.apache.commons.validator_1.3.1
37	ACTIVE      com.springsource.org.apache.oro_2.0.8
38	ACTIVE      com.springsource.org.apache.ivy_2.1.0
39	ACTIVE      com.springsource.org.apache.tools.ant_1.7.1
40	ACTIVE      com.springsource.antlr_2.7.7
41	ACTIVE      com.springsource.org.dom4j_1.6.1
42	ACTIVE      com.springsource.org.aspectj.runtime_1.6.8.RELEASE
43	ACTIVE      com.springsource.org.aspectj.weaver_1.6.8.RELEASE
44	ACTIVE      com.springsource.com.opensymphony.sitemesh_2.4.1
45	ACTIVE      com.springsource.javassist_3.9.0.GA
46	ACTIVE      com.springsource.org.objectweb.asm_1.5.3
47	ACTIVE      com.springsource.org.jboss.cache_3.2.0.GA
48	ACTIVE      com.springsource.org.jboss.util_2.2.13.GA
49	ACTIVE      com.springsource.org.jboss.logging_2.0.5.GA
50	ACTIVE      com.springsource.org.jgroups_2.5.1
51	ACTIVE      com.springsource.net.sf.ehcache_1.6.2
52	ACTIVE      com.springsource.org.hibernate.annotations.common_3.3.0.ga
53	RESOLVED    com.springsource.org.hibernate.annotations_3.4.0.GA
	            Master=54
54	ACTIVE      com.springsource.org.hibernate_3.3.2.GA
	            Fragments=53
55	ACTIVE      com.springsource.org.xmlpull_1.1.4.c
56	RESOLVED    com.springsource.org.apache.xerces_2.9.1
	            Master=60
57	ACTIVE      com.springsource.org.apache.xalan_2.7.1
58	ACTIVE      com.springsource.org.apache.xml.serializer_2.7.1
59	ACTIVE      com.springsource.org.apache.xml.resolver_1.2.0
60	ACTIVE      com.springsource.org.apache.xmlcommons_1.3.4
	            Fragments=56
61	ACTIVE      com.springsource.org.apache.xml.security_1.4.2
62	ACTIVE      org.springframework.aop_3.0.3.RELEASE
63	ACTIVE      org.springframework.asm_3.0.3.RELEASE
64	ACTIVE      org.springframework.aspects_3.0.3.RELEASE
65	ACTIVE      org.springframework.beans_3.0.3.RELEASE
66	ACTIVE      org.springframework.context_3.0.3.RELEASE
67	ACTIVE      org.springframework.context.support_3.0.3.RELEASE
68	ACTIVE      org.springframework.core_3.0.3.RELEASE
69	ACTIVE      org.springframework.expression_3.0.3.RELEASE
70	ACTIVE      org.springframework.instrument_3.0.3.RELEASE
71	ACTIVE      org.springframework.jdbc_3.0.3.RELEASE
72	ACTIVE      org.springframework.jms_3.0.3.RELEASE
73	ACTIVE      org.springframework.orm_3.0.3.RELEASE
74	ACTIVE      org.springframework.oxm_3.0.3.RELEASE
75	ACTIVE      org.springframework.transaction_3.0.3.RELEASE
76	ACTIVE      org.springframework.web_3.0.3.RELEASE
77	ACTIVE      org.springframework.web.servlet_3.0.3.RELEASE
78	ACTIVE      com.springsource.org.aopalliance_1.0.0
79	ACTIVE      com.springsource.net.sf.cglib_2.2.0
80	ACTIVE      org.springframework.osgi.core_2.0.0.M1
81	ACTIVE      org.springframework.osgi.extender_2.0.0.M1
82	ACTIVE      org.springframework.osgi.io_2.0.0.M1
83	INSTALLED   org.springframework.osgi.test_2.0.0.M1
84	ACTIVE      org.springframework.osgi.web_2.0.0.M1
85	ACTIVE      org.springframework.osgi.web.extender_2.0.0.M1
	            Fragments=23
86	INSTALLED   org.grails.osgi_1.3.2
87	INSTALLED   org.grails.crud_1.3.2
88	INSTALLED   org.grails.gorm_1.3.2
89	INSTALLED   org.grails.resources_1.3.2
90	INSTALLED   org.grails.spring_1.3.2
91	INSTALLED   org.grails.web_1.3.2
92	ACTIVE      gant_1.9.1
93	ACTIVE      groovy-all_1.7.3
94	ACTIVE      gant_1.9.2
95	ACTIVE      org.apache.felix.webconsole_2.0.2
96	ACTIVE      tripper_0.1.0

osgi>     
{code}

