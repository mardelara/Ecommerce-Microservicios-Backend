# Perfulandia Microservices

Proyecto de arquitectura de microservicios desarrollado con Spring Boot y Spring Cloud. Simula el backend de una tienda de perfumes con gestión de productos y ventas.

## Arquitectura

```
                        ┌──────────────────┐
                        │  Config Server   │  :8888
                        │  (microservice-  │
                        │    config)       │
                        └────────┬─────────┘
                                 │ configuración centralizada
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
   ┌──────▼──────┐      ┌────────▼───────┐    ┌────────▼───────┐
   │   Eureka    │      │    Gateway     │    │   API Docs     │
   │  (Registro) │      │  (:8080)       │    │  (Swagger)     │
   │   :8761     │      └────────┬───────┘    │   :8060        │
   └─────────────┘               │            └────────────────┘
                          ┌──────┴──────┐
                          │             │
                 ┌────────▼───┐  ┌──────▼──────┐
                 │  Producto  │  │    Venta     │
                 │   :8090    │  │    :9090     │
                 └────────────┘  └─────────────┘
```

## Módulos

| Módulo | Puerto | Descripción |
|---|---|---|
| `microservice-config` | 8888 | Servidor de configuración centralizada (Spring Cloud Config) |
| `microservice-eureka` | 8761 | Servidor de descubrimiento de servicios (Netflix Eureka) |
| `microservice-gateway` | 8080 | API Gateway (Spring Cloud Gateway) |
| `microservice-producto` | 8090 | CRUD de productos con HATEOAS |
| `microservice-venta` | 9090 | CRUD de ventas, consume msvc-producto vía Feign |
| `microservice-api-docs` | 8060 | Documentación Swagger centralizada |

## Tecnologías

- **Java 17** + **Spring Boot 3.3.12**
- **Spring Cloud 2023.0.5** (Config Server, Eureka, Gateway, OpenFeign)
- **Spring Data JPA** + **MySQL 8**
- **Spring HATEOAS**
- **SpringDoc OpenAPI** (Swagger UI)
- **Lombok**
- **JUnit 5** + **Mockito** + **DataFaker**

## Requisitos previos

- Java 17+
- Maven 3.8+
- MySQL 8 corriendo en `localhost:3306`

## Configuración

### Base de datos

Crea las bases de datos necesarias en MySQL:

```sql
CREATE DATABASE perfulandia_db;
CREATE DATABASE db_producto_test;
CREATE DATABASE db_venta_test;
```

### Variables de entorno (opcional)

El proyecto usa valores por defecto para desarrollo local. Si tu entorno difiere, puedes sobreescribir con variables de entorno:

| Variable | Valor por defecto | Descripción |
|---|---|---|
| `DB_URL` | `jdbc:mysql://localhost:3306/perfulandia_db` | URL de BD principal |
| `DB_TEST_URL` | `jdbc:mysql://localhost:3306/db_<servicio>_test` | URL de BD de tests |
| `DB_USERNAME` | `root` | Usuario MySQL |
| `DB_PASSWORD` | *(vacío)* | Contraseña MySQL |

## Cómo ejecutar

Los servicios deben iniciarse **en este orden**:

1. **Config Server** (`microservice-config`)
2. **Eureka** (`microservice-eureka`)
3. **Gateway** (`microservice-gateway`)
4. **Producto** y **Venta** (en cualquier orden)
5. **API Docs** (`microservice-api-docs`) — opcional

Desde cada carpeta de módulo:

```bash
mvn spring-boot:run
```

O desde la raíz, compilar todo:

```bash
mvn clean install -DskipTests
```

## Documentación API

Con todos los servicios corriendo, accede a Swagger UI en:

- **Centralizada:** http://localhost:8060/swagger-ui.html
- **Producto:** http://localhost:8090/swagger-ui.html
- **Venta:** http://localhost:9090/swagger-ui.html
- **Gateway:** http://localhost:8080/swagger-ui.html

## Endpoints principales

### Productos (`/api/v1/productos`)

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/v1/productos` | Listar todos los productos |
| GET | `/api/v1/productos/{id}` | Obtener producto por ID |
| POST | `/api/v1/productos` | Crear producto |
| PUT | `/api/v1/productos/{id}` | Actualizar producto |
| DELETE | `/api/v1/productos/{id}` | Eliminar producto |

### Ventas (`/api/v1/ventas`)

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/v1/ventas` | Listar todas las ventas |
| GET | `/api/v1/ventas/{id}` | Obtener venta por ID |
| POST | `/api/v1/ventas` | Crear venta |
| PUT | `/api/v1/ventas/{id}` | Actualizar venta |
| DELETE | `/api/v1/ventas/{id}` | Eliminar venta |

## Ejecutar tests

```bash
# Tests de un módulo específico
cd microservice-producto
mvn test

# Todos los módulos
mvn test --projects microservice-producto,microservice-venta
```
