# Config Server

Servidor de configuración centralizada para la arquitectura de microservicios bancarios. Sirve los archivos de configuración (`.yaml`) de `product-service` y `customer-service` desde una carpeta local (`config-repo/`), usando el perfil `native`.

## Stack

- Spring Boot
- Spring Cloud Config Server

## Estructura del proyecto

```
config-server/
├── src/
│   └── main/
│       ├── java/.../ConfigServerApplication.java
│       └── resources/
│           └── application.yml
├── config-repo/
│   ├── product-service.yaml
│   └── customer-service.yaml
├── pom.xml
```

## Configuración

`src/main/resources/application.yml`:

```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: file:./config-repo
```

## Cómo correrlo

```bash
mvn spring-boot:run
```

El servidor levanta en el puerto **8888**.

## Verificación

Con el servidor corriendo, cada configuración se puede consultar en:

```
http://localhost:8888/product-service/default
http://localhost:8888/customer-service/default
```

Debería devolver un JSON con toda la configuración correspondiente a ese servicio.

## Cómo agregar la configuración de un nuevo microservicio

1. Crear un archivo `<nombre-del-servicio>.yaml` dentro de `config-repo/`.
2. El nombre del archivo debe coincidir exactamente con el `spring.application.name` que ese microservicio define en su `application.yml` local.
3. El microservicio debe tener la dependencia `spring-cloud-starter-config` y la propiedad `spring.config.import: optional:configserver:http://localhost:8888` en su configuración local.

## Notas

- El modo `native` lee la configuración desde el filesystem local, sin necesidad de un repositorio Git — ideal para desarrollo.
- El prefijo `optional:` en `spring.config.import` (del lado de los microservicios clientes) permite que arranquen igual aunque el Config Server no esté disponible, útil durante el desarrollo.