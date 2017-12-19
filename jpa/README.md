# KumuluzEE JPA and CDI sample

> Develop JPA entities and use CDI within a REST service and pack it as a KumuluzEE microservice.

The objective of this sample is to demonstrate how to develop JPA entities and use CDI. The tutorial guides you through the development of JPA entities and CDI within a REST service and shows how to pack it as a KumuluzEE microservice. You will add KumuluzEE dependencies into pom.xml. To develop the JPA entities, you will create @Entity classes. You will implement a CustomerService CDI class. Finally, you will call this class from the REST service class. Required knowledge: basic familiarity with JPA, CDI and basic concepts of REST and JSON.

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
3. Git:
    * If you have installed Git, you can check the version by typing the following in a command line:
    
        ```
        git --version
        ```
    

## Prerequisites

In order to run this sample you will have to setup a local PostgreSQL database:
- __database host__: localhost:5432
- __database name__: customers
- __user__: postgres
- __password__: postgres

The required tables will be created automatically upon running the sample.

## Usage

The example uses maven to build and run the microservices.

1. Build the sample using maven:

    ```bash
    $ cd jpa
    $ mvn clean package
    ```

2. Run the sample:
* Uber-jar:

    ```bash
    $ java -jar target/${project.build.finalName}.jar
    ```
    
    in Windows environemnt use the command
    ```batch
    java -jar target/${project.build.finalName}.jar
    ```

* Exploded:

    ```bash
    $ java -cp target/classes:target/dependency/* com.kumuluz.ee.EeApplication
    ```
    
    in Windows environment use the command
    ```batch
    java -cp target/classes;target/dependency/* com.kumuluz.ee.EeApplication
    ```
    
    
The application/service can be accessed on the following URL:
* JAX-RS REST resource page - http://localhost:8080/v1/customers

To shut down the example simply stop the processes in the foreground.

## Tutorial
This tutorial will guide you through the steps required to create a simple REST microservice which uses JPA 2.1 and pack it as a KumuluzEE microservice. We will extend the existing [KumuluzEE JAX-RS REST sample](https://github.com/kumuluz/kumuluzee-samples/tree/master/jax-rs),
 with access to the database using JPA 2.1. Therefore, first complete the existing JAX-RS sample tutorial, or clone the JAX-RS sample code. We will use PostgreSQL in this tutorial.

We will follow these steps:
* Complete the tutorial for [KumuluzEE JAX-RS REST sample](https://github.com/kumuluz/kumuluzee-samples/tree/master/jax-rs) or clone the existing sample
* Ensure access to PostgreSQL database.
* Add Maven dependencies
* Implement the persistence using standard JPA
* Build the microservice
* Run it

### Add Maven dependencies

Since your existing starting point is the existing KumuluzEE JAX-RS REST sample, you should already have the dependencies for `kumuluzee-bom`, `kumuluzee-core`, `kumuluzee-servlet-jetty` and `kumuluzee-jax-rs-jersey` configured in `pom.xml`.

Add the `kumuluzee-cdi-weld`, `kumuluzee-jpa-eclipselink` and `postgresql` dependencies:
```xml
<dependency>
    <groupId>com.kumuluz.ee</groupId>
    <artifactId>kumuluzee-cdi-weld</artifactId>
</dependency>
<dependency>
    <groupId>com.kumuluz.ee</groupId>
    <artifactId>kumuluzee-jpa-eclipselink</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.0.0</version>
</dependency>
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

### Implement database access layer

Enhance existing `Customer` class with JPA annotations:
* Add `@Entity` and `@Table` to make it persistable. 
* Add `Customer.findCustomers` named query for retrieving all customers from database
* Mark attribute `id` as a primary key.
* Change the default database column mapping.

```java
@Entity
@Table(name = "customer")
@NamedQueries({
        @NamedQuery(
                name = "Customer.findCustomers",
                query = "SELECT c " +
                        "FROM Customer c"
        )
})
public class Customer implements Serializable {

    @Id
    private String id;
    @Column(name = "first_name")
    private String firstName;
    @Column(name = "last_name")
    private String lastName;

    // TODO: implement get and set methods
}
```

Implement `CustomerService` class. Implement it as a `@RequestScoped` CDI bean, and inject `EntityManager`. Use `EntityManager` to implement the following methods:

```java
public Customer getCustomer(String customerId);
public List<Customer> getCustomers(); 
public void saveCustomer(Customer customer);
public void deleteCustomer(String customerId); 
```
Sample implementation of `CustomerService` class:

```java
@RequestScoped
public class CustomerService {

    @PersistenceContext
    private EntityManager em;

    public Customer getCustomer(String customerId) {
        return em.find(Customer.class, customerId);
    }

    public List<Customer> getCustomers() {
        List<Customer> customers = em
                .createNamedQuery("Customer.findCustomers", Customer.class)
                .getResultList();

        return customers;
    }

    @Transactional
    public void saveCustomer(Customer customer) {
        if (customer != null) {
            em.persist(customer);
        }

    }

    @Transactional(Transactional.TxType.REQUIRED)
    public void deleteCustomer(String customerId) {
        Customer customer = em.find(Customer.class, customerId);
        if (customer != null) {
            em.remove(customer);
        }
    }
}
```

### Implement REST Service

Make `CustomerResource` class a CDI bean by adding `@RequestScoped` annotation. Inject created `CustomerService` to `CustomerResource` using `@Inject` annotation. Replace the invocation of static `Database` class with invocation of injected `CustomerService` implementation. Sample implementation:

```java
@RequestScoped
@Consumes(MediaType.APPLICATION_JSON)
@Produces(MediaType.APPLICATION_JSON)
@Path("customers")
public class CustomerResource {

    @Inject
    private CustomerService customerBean;

    @GET
    public Response getAllCustomers() {
        List<Customer> customers = customerBean.getCustomers();
        return Response.ok(customers).build();
    }

    @GET
    @Path("{customerId}")
    public Response getCustomer(@PathParam("customerId") String customerId) {
        Customer customer = customerBean.getCustomer(customerId);
        return customer != null
                ? Response.ok(customer).build()
                : Response.status(Response.Status.NOT_FOUND).build();
    }

    @POST
    public Response addNewCustomer(Customer customer) {
        customerBean.saveCustomer(customer);
        return Response.noContent().build();
    }

    @DELETE
    @Path("{customerId}")
    public Response deleteCustomer(@PathParam("customerId") String customerId) {
        customerBean.deleteCustomer(customerId);
        return Response.noContent().build();
    }
}
```

### Configure CDI

Create the directory `resources/META-INF`. In this directory create the file `beans.xml` with the following content to enable CDI:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/beans_1_2.xsd"
       bean-discovery-mode="annotated">
</beans>
```

### Configure database and persistence

Make sure, that you have database server prepared, as described in Prerequisites section.

In the directory `resources/META-INF` create the file `persistence.xml`: 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
             http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
             version="2.1">
    <persistence-unit name="kumuluzee-samples-jpa" transaction-type="JTA">

        <jta-data-source>jdbc/CustomersDS</jta-data-source>

        <class>com.kumuluz.ee.samples.jpa.Customer</class>

        <properties>
            <property name="javax.persistence.schema-generation.database.action" value="create"/>
            <property name="javax.persistence.schema-generation.create-source" value="metadata"/>
            <property name="javax.persistence.schema-generation.drop-source" value="metadata"/>
        </properties>
    </persistence-unit>
</persistence>
```

Modify the element `class` in the above example, to reflect the package and class name of entity `Customer` in your source code structure.

In the directory `resources` add the file `config.yaml` with the following database connectivity properties:

```yaml
kumuluzee:
  datasources:
    - jndi-name: jdbc/CustomersDS
      connection-url: jdbc:postgresql://localhost:5432/customers
      username: postgres
      password: postgres
      max-pool-size: 20
```

### Build the microservice and run it

To build the microservice and run the example, use the commands as described in previous sections.
