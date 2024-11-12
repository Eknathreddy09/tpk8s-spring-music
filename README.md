Spring Music
============

This is a sample application for using database services on [Tanzu Platform for K8s](https://docs.vmware.com/en/VMware-Tanzu-Platform/SaaS/create-manage-apps-tanzu-platform-k8s/index.html) with the [Spring Framework](http://spring.io) and [Spring Boot](http://projects.spring.io/spring-boot/).

This application has been built to store the same domain objects in one of a variety of different persistence technologies - relational, document, and key-value stores. This is not meant to represent a realistic use case for these technologies, since you would typically choose the one most applicable to the type of data you need to store, but it is useful for testing and experimenting with different types of services on Cloud Foundry.

The application use Spring Java configuration and [bean profiles](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html) to configure the application and the connection objects needed to use the persistence stores. 

## Building

This project requires Java version 17 or later to compile.

> [!NOTE]
> If you need to use an earlier Java version, check out the [`spring-boot-2` branch](https://github.com/cloudfoundry-samples/spring-music/tree/spring-boot-2), which can be built with Java 8 and later.  

To build a runnable Spring Boot jar file, run the following command:

~~~
$ ./gradlew clean assemble
~~~

## Running the application locally

One Spring bean profile should be activated to choose the database provider that the application should use. The profile is selected by setting the system property `spring.profiles.active` when starting the app.

The application can be started locally using the following command:

~~~
$ java -jar -Dspring.profiles.active=<profile> build/libs/spring-music-1.0.jar
~~~

where `<profile>` is one of the following values:

* `mysql`
* `postgres`
* `mongodb`
* `redis`

If no profile is provided, an in-memory relational database will be used. If any other profile is provided, the appropriate database server must be started separately. Spring Boot will auto-configure a connection to the database using it's auto-configuration defaults. The connection parameters can be configured by setting the appropriate [Spring Boot properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

If more than one of these profiles is provided, the application will throw an exception and fail to start.

## Running the application on Tanzu Platform

When running on TP for K8s, the application will detect the type of database service bound to the application (if any). If a service of one of the supported types (MySQL, Postgres, Oracle, MongoDB, or Redis) is bound to the app, the appropriate Spring profile will be configured to use the database service. The connection strings and credentials needed to use the service will be extracted from the TP environment.

If no bound services are found containing any of these values in the name, an in-memory relational database will be used.

If more than one service containing any of these values is bound to the application, the application will throw an exception and fail to start.

After installing the 'tanzu cli' [command-line interface for Tanzu platform](https://docs.vmware.com/en/VMware-Tanzu-CLI/1.4/tanzu-cli/index.html), targeting a app space, and logging in, the application can be built and pushed using the command:

~~~
$ tanzu deploy -y
~~~

The application will be pushed using settings in the provided `yml` file under .tanzu/config directory. 

### Creating and binding services

Using the provided yml file, the application will be created without an external database (in the `in-memory` profile). You can create and bind database services to the application using the information below.

#### System-managed services

Depending on the Cloud Foundry service provider, persistence services might be offered and managed by the platform. These steps can be used to create and bind a service that is managed by the platform:

~~~
# view the services available
$ tanzu services type list
# create a service instance
$ tanzu  services create PostgreSQLInstance/spring-music-db --parameter storageGB=1 --skip-bind-prompt
# bind the service instance to the application
$ tanzu services bind PostgreSQLInstance/spring-music-db ContainerApp/spring-music
~~~

#### Database drivers

Database drivers for MySQL, Postgres, Microsoft SQL Server, MongoDB, and Redis are included in the project.

To connect to an Oracle database, you will need to download the appropriate driver (e.g. from http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html). Then make a `libs` directory in the `spring-music` project, and move the driver, `ojdbc7.jar` or `ojdbc8.jar`, into the `libs` directory.
In `build.gradle`, uncomment the line `compile files('libs/ojdbc8.jar')` or `compile files('libs/ojdbc7.jar')` and run `./gradle assemble`.


## Alternate Java versions

By default, the application will be built and deployed using Java 17 compatibility.
If you want to use a more recent version of Java, you will need to update two things.

In `build.gradle`, change the `targetCompatibility` Java version from `JavaVersion.VERSION_17` to a different value from `JavaVersion`:

~~~
java {
  ...
  targetCompatibility = JavaVersion.VERSION_17
}
~~~
