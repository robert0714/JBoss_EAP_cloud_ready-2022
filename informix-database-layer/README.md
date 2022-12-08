# informix-database-layer

This project is needed to create a custom user-developer layer that will be used to configure JBoss EAP through the Galleon framework. The project [wildfly-datasources-galleon-pack](https://github.com/jbossas/eap-datasources-galleon-pack) includes several layers to add the most common jdbc datasources (postgresql, mysql and oracle) inside the OpenShift Container Platform (OCP) but I would like to demonstrate how to customize a JBoss EAP subsystem and also show you how use a different database that is not included in the eap-datasources-galleon-pack.

# Reference data
* all env variable
   * https://github.com/wildfly-extras/wildfly-datasources-galleon-pack/blob/main/testsuite/bootablejar/pom.xml
* mysql sample   
   * https://github.com/wildfly-extras/wildfly-datasources-galleon-pack/blob/main/doc/mysql/README.md
   * https://github.com/wildfly-extras/wildfly-datasources-galleon-pack/blob/main/common/src/main/resources/layers/standalone/mysql-datasource/layer-spec.xml
   * https://github.com/wildfly-extras/wildfly-datasources-galleon-pack/blob/main/common/src/main/resources/layers/standalone/postgresql-default-datasource/layer-spec.xml
   * https://github.com/wildfly-extras/wildfly-datasources-galleon-pack/blob/main/common/src/main/resources/layers/standalone/postgresql-driver/layer-spec.xml



## Using Elastic APM
reference: https://www.elastic.co/guide/en/observability/current/ingest-traces.html

```shell
> java  -javaagent:elastic-apm-agent-1.28.4.jar \
-Delastic.apm.service_name=my-application \
-Delastic.apm.server_urls=http://192.168.18.30:8200 \
-Delastic.apm.environment=production \
-Delastic.apm.service_name=interview-cga101g1 \
-Delastic.apm.trace_methods_duration_threshold=100ms \
-Delastic.apm.stack_trace_limit=80 \
-Delastic.apm.application_packages=com.bidapplylist,com.core,com.manager,com.member,com.member \
-Delastic.apm.trace_methods=com.bidapplylist.*,com.core.*,com.manager.*,com.member.*,com.member.* \
-jar weather-app-eap-cloud-ready/target/weather-app-cloud-ready-1.0-bootable.jar \
-Dorg.wildfly.datasources.infomix.datasource=TestDB \
-Dorg.wildfly.datasources.infomix.database=cga101g1 \
-Dorg.wildfly.datasources.infomix.password=tibame \
-Dorg.wildfly.datasources.infomix.user-name=tibame \
-Dorg.wildfly.datasources.infomix.host=192.168.18.30 \
-Dhibernate.connection.url=jdbc:mysql://192.168.18.30:3306/cga101g1 \
-DREDIS_HOST=localhost \
-Djboss.bind.address=0.0.0.0
```

## Testsuite Example
* https://github.com/wildfly-extras/wildfly-datasources-galleon-pack/blob/main/testsuite/bootablejar/src/test/java/org/wildfly/datasources/test/SimpleTestCase.java

Galleon Layers
=========

* `informix-default-datasource`: Provision the `INFORMIXDS` non xa datasource and configure it as the default one. Depends on `informix-datasource` layer.
* `informix-datasource`: Provision the `INFORMIXDS` non xa datasource. Depends on `informix-driver` layer.
* `informix-driver`: Provision the `informix` driver. This layer installs the JBoss Modules module `com.infomix.jdbc`.

Configuration
========

The following set of environment variables and corresponding Java system properties can be used to configure the datasource.

Required configuration
==============

* `INFORMIX_DATABASE` (or `OPENSHIFT_INFORMIX_DB_NAME`)

  * Description: Defines the database name to be used in the datasource’s `connection-url` property.
  * No default value.
  * Required: True if `INFORMIX_URL` is not set.
  * System Property: `org.wildfly.datasources.infomix.database`

* `INFORMIX_PASSWORD` (or `OPENSHIFT_INFORMIX_DB_PASSWORD`)

  * Description: Defines the password for the datasource.
  * No default value.
  * Required: True
  * System Property: `org.wildfly.datasources.infomix.password`

* `INFORMIX_URL`

  * Description: Defines the connection URL for the datasource. 
  * Default Value: `jdbc:informix-sqli://${INFORMIX_HOST}:${INFORMIX_PORT}/${INFORMIX_DATABASE}`
  * Required: True if `INFORMIX_PORT/HOST/DATABASE` are not set.
  * System Property: `org.wildfly.datasources.infomix.connection-url`

* `INFORMIX_USER` (or `OPENSHIFT_INFORMIX_DB_USERNAME`)

  * Description: Defines the username for the datasource. 
  * No default value.
  * Required: True
  * System Property: `org.wildfly.datasources.infomix.user-name`

Optional configuration
==============

* `INFORMIX_BACKGROUND_VALIDATION`

  * Description: When set to true database connections are validated periodically in a background thread prior to use. Defaults to false, meaning the `validate-on-match` method is enabled by default instead.  
  * Default Value: `false`
  * System Property: `org.wildfly.datasources.infomix.background-validation`

* `INFORMIX_BACKGROUND_VALIDATION_MILLIS`

  * Description: Specifies frequency of the validation, in milliseconds, when the `background-validation` database connection validation mechanism is enabled.    
  * Default Value: `10000`
  * System Property: `org.wildfly.datasources.infomix.background-validation-millis`


* `INFORMIX_DATASOURCE` (or `OPENSHIFT_INFORMIX_DATASOURCE`)

  * Description: Datasource name used in the `jndi-name`.
  * Default Value: `MySQLDS`
  * System Property: `org.wildfly.datasources.infomix.datasource`

* `INFORMIX_ENABLED`

  * Description: Set to false to disable the datasource.
  * Default Value: `true`
  * System Property: `org.wildfly.datasources.infomix.enabled`

* `INFORMIX_EXCEPTION_SORTER`

  * Description: Specifies the exception sorter class that is used to properly detect and clean up after fatal database connection exceptions. Valid value: `org.jboss.jca.adapters.jdbc.extensions.informix.InformixExceptionSorter`
  * Default Value: `org.jboss.jca.adapters.jdbc.extensions.informix.InformixExceptionSorter`
  * System Property: `org.wildfly.datasources.infomix.exception-sorter-class-name`

* `INFORMIX_FLUSH_STRATEGY`

  * Description: Specifies how the datasource should be flushed in case of an error.    
  * Default Value: `FailingConnectionOnly`
  * System Property: `org.wildfly.datasources.infomix.flush-strategy`

* `INFORMIX_HOST` (or `INFORMIX_SERVICE_HOST` or `OPENSHIFT_INFORMIX_DB_HOST`)

  * Description: Defines the database server’s host name or IP address to be used in the datasource’s `connection-url` property.
  * Default Value: `localhost`
  * Required: True if `INFORMIX_URL` is not set.
  * System Property: `org.wildfly.datasources.infomix.host`

* `INFORMIX_IDLE_TIMEOUT_MINUTES`

  * Description: Specifies the maximum time, in minutes, a connection may be idle before being closed.
  * Default Value:`30`
  * System Property: `org.wildfly.datasources.infomix.idle-timeout-minutes`

* `INFORMIX_JNDI`

  * Description: Defines the JNDI name for the datasource.
  * Default Value:` java:jboss/datasources/${INFORMIX_DATASOURCE}`
  * System Property: `org.wildfly.datasources.infomix.jndi-name`

* `INFORMIX_JTA`

  * Description: Defines Java Transaction API (JTA) option for the non-XA datasource.
  * Default Value: `true`
  * System Property: `org.wildfly.datasources.infomix.jta`

* `INFORMIX_MAX_POOL_SIZE`

  * Description: Defines the maximum pool size option for the datasource.
  * Default Value: `20`
  * System Property: `org.wildfly.datasources.infomix.max-pool-size`

* `INFORMIX_MIN_POOL_SIZE`

  * Description: Defines the minimum pool size option for the datasource.
  * Default Value: `0`
  * System Property: `org.wildfly.datasources.infomix.min-pool-size`

* `INFORMIX_PORT` (or `INFORMIX_SERVICE_PORT` or `OPENSHIFT_INFORMIX_DB_PORT`)

  * Description: Defines the database server’s port to be used in the datasource’s `connection-url` property. 
  * Default Value: `5432`
  * Required: True if `INFORMIX_URL` is not set.
  * System Property: `org.wildfly.datasources.infomix.port`

* `INFORMIX_STALE_CONNECTION_CHECKER`

  * Description: Specifies a connection checker class that is used to check stale connections. Valid value: No checker provided by JBoss, only custom stale checker can be provided.
  * Default Value: `org.jboss.jca.adapters.jdbc.extensions.novendor.NullStaleConnectionChecker`
  * System Property: `org.wildfly.datasources.infomix.stale-connection-checker-class-name`

* `INFORMIX_TX_ISOLATION`

  * Description: Defines the `java.sql.Connection` transaction isolation level for the datasource.    
  * Default Value: `TRANSACTION_READ_COMMITTED`
  * System Property: `org.wildfly.datasources.infomix.transaction-isolation`

* `INFORMIX_VALIDATE_ON_MATCH`

  * Description: Indicates whether or not connection level validation should be done when a connection factory attempts to match a managed connection for a given set. This is typically exclusive to the use of background validation.
  * Default Value: `true`
  * System Property: `org.wildfly.datasources.infomix.validate-on-match`

