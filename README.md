# Config Server

Servidor de configuración centralizada para la arquitectura de microservicios bancarios. Sirve los archivos de configuración (`.yaml`) de `eureka-server`, `product-service` y `customer-service` desde un repositorio Git remoto.

## Stack

- Spring Boot
- Spring Cloud Config Server

## Repositorio de configuración

Los archivos `.yaml` de cada microservicio (`eureka-server.yaml`, `product-service.yaml`, `customer-service.yaml`) viven en un repositorio Git separado: [UTN-Config-Repo](https://github.com/tu-usuario/UTN-Config-Repo).

## Configuración

`src/main/resources/application.yaml`:

```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/tu-usuario/UTN-Config-Repo
          default-label: main
```

> La URL del repositorio de configuración está fija para este proyecto. En un despliegue con múltiples entornos, conviene externalizarla con una variable de entorno.

## Cómo correrlo

```bash
mvn spring-boot:run
```

El servidor levanta en el puerto **8888**.

## Verificación

Con el servidor corriendo, cada configuración se puede consultar en:

```
http://localhost:8888/eureka-server/default
http://localhost:8888/product-service/default
http://localhost:8888/customer-service/default
```

Debería devolver un JSON con toda la configuración correspondiente a ese servicio.

## Orden de arranque

Este servicio debe levantarse **primero**, ya que `eureka-server`, `product-service` y `customer-service` dependen de él para obtener su puerto y demás configuración.

## Notas

- El repositorio de configuración es leído directamente desde GitHub en cada solicitud (o según la política de caché del Config Server) — no requiere una copia local del filesystem.
- Al no usar `optional:` en `spring.config.import` de los clientes, cada microservicio requiere que el Config Server esté disponible al arrancar; si no lo está, no toma su configuración y puede caer al puerto por defecto (8080), lo que refuerza la importancia de respetar el orden de arranque documentado.