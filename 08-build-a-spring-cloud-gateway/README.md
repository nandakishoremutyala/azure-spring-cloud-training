# 08 - Build a Spring Cloud Gateway

__This guide is part of the [Azure Spring Cloud training](../README.md)__

A Spring Cloud gateway allows you to selectively expose your microservices and to route traffic to them and among them. In this section, we will create a Spring Cloud Gateway that will expose the microservices we created in the preceding two sections.

---

## Create a Spring Cloud Gateway

The application that we create in this guide is [available here](gateway/).

To create our gateway, we will invoke the Spring Initalizer service from the command line:

```bash
curl https://start.spring.io/starter.tgz -d dependencies=cloud-gateway,cloud-eureka,cloud-config-client -d baseDir=gateway -d bootVersion=2.6.1 -d javaVersion=11 | tar -xzvf -
```

> We use the `Cloud Gateway`, `Eureka Discovery Client` and the `Config Client` components.

## Configure the application

Rename `src/main/resources/application.properties` to `src/main/resources/application.yml` and add the following configuration:

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods:
              - GET

```

- The `spring.cloud.gateway.discovery.locator.enabled=true` part is to configure Spring Cloud Gateway to use the Spring Cloud Service Registry to discover the available microservices.
- The `spring.cloud.gateway.globalcors.corsConfiguration` part is to allow CORS requests to our gateway. This will be helpful in the next guide, when we will add a front-end that is not hosted on Azure Spring Cloud.

## Create the application on Azure Spring Cloud

As in [02 - Build a simple Spring Boot microservice](../02-build-a-simple-spring-boot-microservice/README.md), create a specific `gateway` application in your Azure Spring Cloud instance. As this application is a gateway, we add the `--is-public true` flag so it is exposed publicly.

```bash
az spring-cloud app create -n gateway --runtime-version Java_11 --is-public true
```

## Deploy the application

You can now build your "gateway" project and send it to Azure Spring Cloud:

```bash
cd gateway
./mvnw clean package -DskipTests
az spring-cloud app deploy -n gateway --artifact-path target/demo-0.0.1-SNAPSHOT.jar
cd ..
```

## Test the project in the cloud

- Go to "Apps" in your Azure Spring Cloud instance.
  - Verify that `gateway` has a `Registration status` which says `1/1`. This shows that it is correctly registered in the Spring Cloud Service Registry.
  - Select `gateway` to have more information on the microservice.
- Copy/paste the public URL that is provided (there is a "Test endpoint" like for microservices, but the gateway is directly exposed on the Internet, so let's use the public URL). Keep this URL handy for subsequent sections.

As the gateway is connected to the Spring Cloud Service Registry, it should have automatically opened routes to the available microservices, with URL paths in the form of `/MICROSERVICE-ID/**`.

  > 🛑 **The MICROSERVICE-ID must be uppercase, all CAPS**. Replace XXXXXXXX with the name of your Azure Spring Cloud instance.

- Test the `city-service` microservice endpoint: 
  ```
  curl https://XXXXXXXX-gateway.azuremicroservices.io/CITY-SERVICE/cities
  ```
- Test the `weather-service` microservice endpoint by doing: 
  ```
  curl 'https://XXXXXXXX-gateway.azuremicroservices.io/WEATHER-SERVICE/weather/city?name=Paris%2C%20France'
  ```

If you need to check your code, the final project is available in the ["gateway" folder](gateway/).

---

⬅️ Previous guide: [07 - Build a Spring Boot microservice using MySQL](../07-build-a-spring-boot-microservice-using-mysql/README.md)

➡️ Next guide: [09 - Putting it all together, a complete microservice stack](../09-putting-it-all-together-a-complete-microservice-stack/README.md)
