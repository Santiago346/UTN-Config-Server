# Sistema Bancario — Arquitectura de Microservicios

Sistema de gestión de clientes y productos bancarios, construido con Spring Boot bajo una arquitectura de microservicios con descubrimiento de servicios y configuración centralizada.

## Repositorios del proyecto

| Componente | Repositorio | Responsabilidad |
|---|---|---|
| Eureka Server | [UTN-Eureka-Server](https://github.com/Santiago346/UTN-Eureka-Server) | Registro y descubrimiento de servicios |
| Config Server | [UTN-Config-Server](https://github.com/Santiago346/UTN-Config-Server) | Configuración centralizada de los microservicios |
| Product Service | [UTN-Product-Service](https://github.com/Santiago346/UTN-Product-Service) | Gestión de productos bancarios (cuentas, seguros, plazos fijos) |
| Customer Service | [UTN-Customer-Service](https://github.com/Santiago346/UTN-Customer-Service) | Gestión de clientes y consulta de sus productos asociados |

## Diagrama de arquitectura

```
              ┌────────────────────────┐
              │    Config Server        │
              │    (puerto 8888)        │
              │  Configuración           │
              │  centralizada (Git)      │
              └───────────┬─────────────┘
                          │
              sirve configuración (puerto,
              datasource, URL de Eureka)
                          │
                         ▼
                         ┌─────────────────────┐
                         │   Eureka Server      │
                         │   (puerto 8761)      │
                         │  Service Discovery    │
                         └──────────┬───────────┘
                                    │
                    registro / descubrimiento
                                    │
              ┌─────────────────────┴─────────────────────┐
              │                                            │
   ┌──────────▼──────────┐                       ┌─────────▼─────────┐
   │  Product Service     │                       │ Customer Service   │
   │  (puerto 8082)       │◄──────────────────────┤ (puerto 8081)      │
   │  /productos           │      Feign Client      │  /clientes          │
   └───────────────────────┘                       └─────────────────────┘
```

**Nota:** el puerto y la configuración de Eureka Server también vienen del Config Server, así que este debe levantarse primero. `product-service` y `customer-service` dependen de que **ambos** (Config Server y Eureka Server) ya estén arriba antes de arrancar.

## Tabla de puertos

| Servicio | Puerto |
|---|---|
| Config Server | 8888 |
| Eureka Server | 8761 |
| Customer Service | 8081 |
| Product Service | 8082 |

## Qué hace cada componente

- **Eureka Server**: mantiene el registro en vivo de qué servicios están corriendo y dónde, permitiendo que `customer-service` encuentre a `product-service` por nombre (sin URLs fijas).
- **Config Server**: centraliza la configuración de `product-service` y `customer-service` (puerto, datasource H2, URL de Eureka), leyéndola desde un repositorio Git remoto.
- **Product Service**: expone el CRUD de productos bancarios (alta, consulta, actualización) y permite filtrar productos por cliente.
- **Customer Service**: expone el CRUD de clientes (alta, consulta, actualización, baja) y se comunica con `product-service` vía Feign para traer los productos asociados a un cliente.

## Requisitos previos

- Java 17
- Maven (o usar el wrapper `mvnw`/`mvnw.cmd` incluido en cada repositorio, no requiere tener Maven instalado globalmente)
- Git

## Orden de arranque

Como el puerto y la configuración de cada servicio (incluido Eureka Server) vienen del Config Server, el orden correcto es:

1. **Config Server**
2. **Eureka Server**
3. **Product Service**
4. **Customer Service**

```bash
# Terminal 1
cd UTN-Config-Server && ./mvnw spring-boot:run

# Terminal 2 (una vez que Config Server esté arriba)
cd UTN-Eureka-Server && ./mvnw spring-boot:run

# Terminal 3
cd UTN-Product-Service && ./mvnw spring-boot:run

# Terminal 4
cd UTN-Customer-Service && ./mvnw spring-boot:run
```

## Verificación rápida

- Dashboard de Eureka: `http://localhost:8761` — deberían aparecer `product-service` y `customer-service` como `UP`.
- Configuración servida por el Config Server:
  - `http://localhost:8888/product-service/default`
  - `http://localhost:8888/customer-service/default`
- Endpoint de integración entre servicios (Feign + Eureka + Config Server funcionando juntos):
  - `http://localhost:8081/clientes/{id}/productos`
