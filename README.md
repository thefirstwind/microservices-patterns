# microservices-patterns

# FTGO example application

This is the example code for the book [Microservice patterns](https://github.com/thefirstwind/microservices-patterns/blob/main/_attachments/Microservices_Patterns.pdf).

![](_images/687474703a2f2f6d6963726f73657276696365732e696f2f692f4d6963726f73657276696365732d5061747465726e732d436f7665722e706e67.png)

## Please note

* The code is still work in progress
* It primarily illustrates the technical aspects of the microservice architecture and so the business logic is minimal
* The documentation is sparse/non-existent and you will need to look in the book
* The application consists of many services and so requires a lot of memory. It runs well, for example, on a 16GB Macbook pro.
* The application's services and the infrastructure services, such as MySQL and Apache Kafka, are deployed using Docker containers using either Docker Compose or Kubernetes.


## Application architecture

Not surprisingly, this application has a microservice architecture.
There are the following services:

* [ftgo-consumer-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-consumer-service) - the `Consumer Service`
* [ftgo-restaurant-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-restaurant-service) - the `Restaurant Service`
* [ftgo-order-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service) - the `Order Service`
* [ftgo-kitchen-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-kitchen-service) - the `Kitchen Service`
* [ftgo-accounting-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-accounting-service) - the `Accounting Service`
* [ftgo-order-history-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-history-service) - a `Order History Service`, which is a CQRS view
* [ftgo-api-gateway](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-api-gateway) - the API gateway

## Service design

Key points:

* A service consists of a single Gradle module.
For example, `ftgo-order-service` implements the `Order Service`
* A service is a Spring Boot application
* A service has a Swagger UI `http://.../swagger-ui.html`. See `open-swagger-uis.sh`
* A service typically consists of the following packages:
  * domain - domain logic including aggregates
  * messaging - messaging adapters
  * web - Spring MVC controllers (HTTP adapters)
  * main - the main application
* The services use the following other frameworks
  * [`Eventuate Tram framework`](https://github.com/eventuate-tram/eventuate-tram-core) - implements transactional messaging
  * [`Eventuate Tram Saga framework`](https://github.com/eventuate-tram/eventuate-tram-sagas) - implements sagas
  * [`Eventuate Client framework`](https://github.com/eventuate-clients/eventuate-client-java) - implements event sourcing

## Chapter by chapter

This section maps the chapters to the code.

### Chapter 3 Inter-process communication in a microservice architecture

* The services have a REST API
* The services also communicate using the Apache Kafka message broker via the `Eventuate Tram` framework

### Chapter 4 Managing transactions with sagas

The [ftgo-order-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service) uses sagas to maintain data consistency:

* [CreateOrderSaga](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service/src/main/java/net/chrisrichardson/ftgo/orderservice/sagas/createorder/CreateOrderSaga.java)
* [CancelOrderSaga](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service/src/main/java/net/chrisrichardson/ftgo/orderservice/sagas/cancelorder/CancelOrderSaga.java)
* [ReviseOrderSaga](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service/src/main/java/net/chrisrichardson/ftgo/orderservice/sagas/reviseorder/ReviseOrderSaga.java)

The services that participate in these sagas define the following command handlers:

* `Accounting Service` [AccountingServiceCommandHandler](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-accounting-service/src/main/java/net/chrisrichardson/ftgo/accountingservice/messaging/AccountingServiceCommandHandler.java)
* `Consumer Service` [ConsumerServiceCommandHandlers](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-consumer-service/src/main/java/net/chrisrichardson/ftgo/consumerservice/domain/ConsumerServiceCommandHandlers.java)
* `Kitchen Service` [KitchenServiceCommandHandler](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-kitchen-service/src/main/java/net/chrisrichardson/ftgo/kitchenservice/messagehandlers/KitchenServiceCommandHandler.java)
* `Order Service` [OrderCommandHandlers](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service/src/main/java/net/chrisrichardson/ftgo/orderservice/service/OrderCommandHandlers.java)



### Chapter 5 Designing business logic in a microservice architecture

All the services' business logic is implemented using Domain-Driven design aggregates.

* `Accounting Service`
** [`Account`](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-accounting-service/src/main/java/net/chrisrichardson/ftgo/accountingservice/domain/Account.java) aggregate in the link:./ftgo-accounting-service[ftgo-accounting-service]
* `Consumer Service`
** [Consumer](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-consumer-service/src/main/java/net/chrisrichardson/ftgo/consumerservice/domain/Consumer.java)
* `Order Service`
** [Order](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service/src/main/java/net/chrisrichardson/ftgo/orderservice/domain/Order.java)
** [Restaurant](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-service/src/main/java/net/chrisrichardson/ftgo/orderservice/domain/Restaurant.java)
* `Kitchen Service`
** [Restaurant](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-kitchen-service/src/main/java/net/chrisrichardson/ftgo/kitchenservice/domain/Restaurant.java)
** [Ticket](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-kitchen-service/src/main/java/net/chrisrichardson/ftgo/kitchenservice/domain/Ticket.java)
* `Restaurant Service`
** [Restaurant](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-restaurant-service/src/main/java/net/chrisrichardson/ftgo/restaurantservice/domain/Restaurant.java)


### Chapter 6 Developing business logic with event sourcing

* [`Account`](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-accounting-service/src/main/java/net/chrisrichardson/ftgo/accountingservice/domain/Account.java) aggregate in the link:./ftgo-accounting-service[ftgo-accounting-service] is implemented using event sourcing

### Chapter 7 Implementing queries in a microservice architecture

* [ftgo-order-history-service](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-order-history-service) is an example of a CQRS view
* [ftgo-api-gateway](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-api-gateway) uses API composition to implement the REST endpoint for retrieving the order history

### Chapter 8 External API patterns

* [ftgo-api-gateway](https://github.com/thefirstwind/microservices-patterns/blob/main/ftgo-api-gateway) is the API gateway


## Building and running the application

### Pre-requisites

* Java 8+
* Docker and Docker Compose
* Internet access so that Gradle and Docker can download dependencies and container images

### Building

Temporary: Build the Spring Cloud Contracts using this command:

```
./gradlew buildContracts
```

Build the services using this command:

```
./gradlew assemble
```

### Running the application

Run the application using this command:

```
./gradlew :composeUp
```

Note: the ':'

This can take a while.

### Using the application

Use the services Swagger UIs to invoke the services.

* Create consumer - `http://localhost:8081/swagger-ui.html`
* Create a restaurant - `http://localhost:8084/swagger-ui.html`
* Create an order - `http://localhost:8082/swagger-ui.html`
* View the order - `http://localhost:8082/swagger-ui.html`
* View the order history -  `http://localhost:8086/swagger-ui.html`

You can also access the application via the `API Gateway` at `http://localhost:8087`.
However, currently it  doesn't have a Swagger UI so you will have to use `curl`, for example.

Note: if the containers aren't accessible via `localhost` - e.g. you are using Docker Toolbox, you will have to use `${DOCKER_HOST_IP}` as described below.

### Stopping the application

Stop the application using this command:

```
./gradlew :composeDown
```

## Deploying the application on Kubernetes

You can find Kubernetes YAML files in the following directories: `deployment/kubernetes` and `*/src/deployment/kubernetes`.
There are also some helpful shell scripts.

### Deploying services

You can run this command

```
./deployment/kubernetes/scripts/kubernetes-deploy-all.sh
```

### Undeploying the services

You can run the script to undeploy the services:

```
./deployment/kubernetes/scripts/kubernetes-delete-all.sh
```

If you want to delete the persistent volumes for Apache Kafka, Zookeeper and MySQL please run the command:

```
./deployment/kubernetes/scripts/kubernetes-delete-volumes.sh
```

## Setting environment variables to do development

You should not need to set any environment variables.
To run the application, you certainly do not.
Similarly, to do development (e.g. run tests), you typically do not need to set any environment variables.
That's because Docker containers are generally accessible (e.g. Docker for Windows/Mac) on the host via `localhost`.
However, if Docker is running elsewhere (e.g. you are using Docker Toolbox) you will need to set `DOCKER_HOST_IP`.

### Quick way

A quick way to set the environment variables is to run the script `./set-env.sh`.

### Long way

The value of `DOCKER_HOST_IP` must be meaningful to both Java services/tests running on your desktop/laptop and to Docker containers.
Please do NOT set it to the unresolvable hostname of your machine, `localhost` or `127.0.0.1` since the Docker containers will probably not work correctly.

### Verifying that DOCKER_HOST_IP is set correctly

You can verify that `DOCKER_HOST_IP` is set correctly by running this command:

```
docker run -p 8889:8888 -e DOCKER_DIAGNOSTICS_PORT=8889 -e DOCKER_HOST_IP \
     --rm eventuateio/eventuateio-docker-networking-diagnostics:0.2.0.RELEASE
```

### Setting the environment variable in your IDE

If you want to run Java services/tests within your IDE on your desktop/laptop AND  the Docker containers are not accessible via `localhost` THEN you will need to set `DOCKER_HOST_IP` within your IDE.
How to do this depends on your operating system and IDE.
For example, I find it convenient to launch my IDE from the command line and after setting this environment variable.
