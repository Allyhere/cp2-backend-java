# cp2-backend-java

This is a minimal CRUD service exposing a couple of endpoints over REST,
with a front-end based on Angular so you can play with it from your browser.

While the code is surprisingly simple, under the hood this is using:
 - RESTEasy Reactive to expose the REST endpoints
 - Hibernate Reactive to perform the CRUD operations on the database
 - A PostgreSQL database; see below to run one via Docker
 - ArC, the CDI inspired dependency injection tool with zero overhead

## Requirements

To compile and run this demo you will need:

- JDK 17+
- GraalVM

In addition, you will need either a PostgreSQL database, or Docker to run one.

### Configuring GraalVM and JDK 17+

Make sure that both the `GRAALVM_HOME` and `JAVA_HOME` environment variables have
been set, and that a JDK 17+ `java` command is on the path.

See the [Building a Native Executable guide](https://quarkus.io/guides/building-native-image)
for help setting up your environment.

## Building the demo

Launch the Maven build on the checked out sources of this demo:

> ./mvnw package

## Running the demo

### Live coding with Quarkus

The Maven Quarkus plugin provides a development mode that supports
live coding. To try this out:

> ./mvnw quarkus:dev

In this mode you can make changes to the code and have the changes immediately applied, by just refreshing your browser.

**Note:** By default, if no database is configured, Quarkus starts a Docker container with a Postgres database (Dev Services). However, since we have explicitly configured the external database in `src/main/resources/application.properties`, Dev Services will be disabled and the application will connect to the specified external instance.

To access the external database from the terminal (if running via Docker):

```sh
docker exec -it postgres-app psql -U appuser -d appdb
```

    Hot reload works even when modifying your JPA entities.
    Try it! Even the database schema will be updated on the fly.

### Run Quarkus in JVM mode

When you're done iterating in developer mode, you can run the application as a
conventional jar file.

First compile it:

> ./mvnw package

Next, make sure you have your external PostgreSQL database running (e.g., in a VM) with the following configuration:

- **Host**: `127.0.0.1`
- **Port**: `5432`
- **Database**: `appdb`
- **User**: `appuser`
- **Password**: `apppassword`

If you want to run a local PostgreSQL instance with these credentials using Docker, you can use:

> docker run -it --rm=true --name postgres-app -e POSTGRES_USER=appuser -e POSTGRES_PASSWORD=apppassword -e POSTGRES_DB=appdb -p 5432:5432 postgres:15

Connection properties are defined in `src/main/resources/application.properties`.

Then run the application:

> java -jar ./target/quarkus-app/quarkus-run.jar

    Have a look at how fast it boots.
    Or measure total native memory consumption...

### Run Quarkus as a native application

You can also create a native executable from this application without making any
source code changes. A native executable removes the dependency on the JVM:
everything needed to run the application on the target platform is included in
the executable, allowing the application to run with minimal resource overhead.

Compiling a native executable takes a bit longer, as GraalVM performs additional
steps to remove unnecessary codepaths. Use the  `native` profile to compile a
native executable:

> ./mvnw package -Dnative

After getting a cup of coffee, you'll be able to run this binary directly:

> ./target/hibernate-reactive-quickstart-1.0.0-SNAPSHOT-runner

    Please brace yourself: don't choke on that fresh cup of coffee you just got.
    
    Now observe the time it took to boot, and remember: that time was mostly spent to generate the tables in your database and import the initial data.
    
    Next, maybe you're ready to measure how much memory this service is consuming.

N.B. This implies all dependencies have been compiled to native;
that's a whole lot of stuff: from the bytecode enhancements that Hibernate ORM
applies to your entities, to the lower level essential components such as the PostgreSQL JDBC driver, the Undertow webserver.

## See the demo in your browser

Navigate to:

<http://localhost:8080/index.html>

## Testing the Endpoints

You can test the REST endpoints using `curl`:

### List all fruits
```bash
curl -v http://localhost:8080/fruits
```

### Create a new fruit
```bash
curl -v -X POST -H "Content-Type: application/json" -d '{"name":"Orange"}' http://localhost:8080/fruits
```

### Update a fruit
```bash
curl -v -X PUT -H "Content-Type: application/json" -d '{"name":"Granny Smith"}' http://localhost:8080/fruits/2
```

### Delete a fruit
```bash
curl -v -X DELETE http://localhost:8080/fruits/1
```

