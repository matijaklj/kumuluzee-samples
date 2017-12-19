# Microservices simple

> Simple example of splitting an application into microservices using KumuluzEE.

This example demonstrates how to split an application into microservices using KumuluzEE and create a self-sustaining
package that runs independently for each one.

Let's create an online book catalogue that people can browse and place orders for the books they like.
While there would normally be many more functionalities, for brevity we are only going to look at the two of them;
browsing available books and placing orders.

We start by splitting the catalogue and orders functionalities into two separately configured and deployed microservices.
That way we have created microservices that are only concerned with their respected functionalities.
We’ve also reduced the interference with one another and overall form a better modular and bug free application.
Each one of them will communicate with each other through pre-defined REST interfaces.

The example demonstrates the above premise.

## Requirements

In order to run this example you will need the following:

1. Java 8 (or newer), you can use any implementation:
    * If you have installed Java, you can check the version by typing the following in a command line:
        
        ```
        java -version
        ```

2. Maven 3.2.1 (or newer):
    * If you have installed Maven, you can check the version by typing the following in a command line:
        
        ```
        mvn -version
        ```
        
3. (Optional, to get a database running) Docker 1.13.0 (or newer):
    * If you have installed Maven, you can check the version by typing the following in a command line:
    
        ```
        docker --version
        ```
    
## Prerequisites

Make sure you have PostgreSQL database running in order to start the example. If you don't have one, you can simply start
an instance using docker:

    ```bash
    docker run -e POSTGRES_PASSWORD=postgres --restart=always -p 5432:5432 -d postgres:9.6-alpine
    ```

## Usage

The example uses docker to build and run the microservices.

1. Build the sample using maven:

    ```bash
    $ cd servlet
    $ mvn clean package
    ```

2. Run each individual microservice separately (separate terminal) with a single command with the appropriate environment variables that can override the applications config located in `config.yaml``:
    * `PORT` should containt the port on which the microservice will accept connections
    * `KUMULUZEE_DATASOURCES[0].CONNECTION-URL` should contain the jdbc URL for the datasource (defaults are obtained from the `config.yaml` file)
    * `KUMULUZEE_DATASOURCES[0].USERNAME` should contain the datasource username (defaults are obtained from the `config.yaml` file)
    * `KUMULUZEE_DATASOURCES[0].PASSWORD` should contain the datasource password (defaults are obtained from the `config.yaml` file)
    * `ORDERS.CATALOGUEURL` should contain the catalogue base url for the orders service
    
    Built as exploded:
    
    ```bash
    $ PORT=3000 java -cp "catalogue/target/classes:catalogue/target/dependency/*" com.kumuluz.ee.EeApplication
    
    $ PORT=3001 java -cp "orders/target/classes:orders/target/dependency/*" com.kumuluz.ee.EeApplication
    ```
    
    Build as uber-jar:
    
    ```bash
        $ PORT=3000 java -jar target/${project.build.finalName}.jar
        
        $ PORT=3001 java -jar target/${project.build.finalName}.jar
    ```
    
The application/service can be accessed on the following URLs:
    * Catalogue service - http://localhost:3000/books
    * Orders service - http://localhost:3001/orders

To shut down the example simply stop the processes in the foreground.

##Tutorial

This example demonstrates how to split an application into microservices using KumuluzEE and create a self-sustaining
package that runs independently for each one.

We will follow these steps:
* Create maven project with three modules: catalogue, orders and shared module between the two models.
* Add Maven dependency to individual modules
* Implent microservices
* Build the microservice
* Run it

### Add Maven dependencies

To catalogue and orders add the following dependencies:

```xml
  <dependencies>
        <dependency>
            <groupId>com.kumuluz.ee.samples</groupId>
            <artifactId>microservices-simple-models</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>com.kumuluz.ee</groupId>
            <artifactId>kumuluzee-microProfile-1.0</artifactId>
        </dependency>
    </dependencies>
```

Add the `kumuluzee-maven-plugin` build plugin to package microservice as uber-jar:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.kumuluz.ee</groupId>
            <artifactId>kumuluzee-maven-plugin</artifactId>
            <version>${kumuluzee.version}</version>
            <executions>
                <execution>
                    <id>package</id>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

or exploded:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.kumuluz.ee</groupId>
            <artifactId>kumuluzee-maven-plugin</artifactId>
            <version>${kumuluzee.version}</version>
            <executions>
                <execution>
                    <id>package</id>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

To models module add the following maven dependencies:

```xml
<dependencies>
        <dependency>
            <groupId>com.kumuluz.ee</groupId>
            <artifactId>kumuluzee-jpa-eclipselink</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.0.0</version>
        </dependency>
    </dependencies>
```

### Build the microservice and run it

To build the microservice and run the example, use the commands as described in previous sections.