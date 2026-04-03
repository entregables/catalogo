# 📦 Microservicio Catálogo

Este proyecto implementa el **Microservicio Catálogo**, responsable de gestionar entidades del dominio dentro de una arquitectura de microservicios en evolución.

---

## 🧱 Estado del proyecto

Actualmente incluye:

- API REST funcional
- Persistencia con MySQL
- Configuración por perfiles (`dev`, `prod`)
- Contenerización con Docker
- Preparado para integración con:
  - Config Server
  - Eureka
  - API Gateway

---

## 🏗️ Arquitectura (visión)

```text
Client → Gateway → Microservicios → Eureka → Config Server
```

Este repositorio implementa únicamente el microservicio **Catálogo**.

Ubicación recomendada para clases/equipos:

- Cada microservicio (`catalogo`, `producto`, `[otro-ms]`) vive en su propio repositorio Git.
- Clonar cada repositorio de microservicio dentro de la carpeta `services` para trabajo integrado local.
- Mantener la infraestructura en un único repositorio `infra` (Config Server, Registry, Gateway, etc.).
- Estructura sugerida:

```text
ProyectosMS2026/
  infra/
    config-server/
    registry-server/
    gateway/
  services/
    catalogo/
    producto/
    [otro-ms]/
```

---

## ⚙️ Stack tecnológico base 2026


- Java 17
- Spring Boot 3.5.x
- Maven 3.9+
- MySQL 8 (dentro de docker)
- Docker
- Docker Compose

Antes de ejecutar el proyecto, asegúrate de tener instalado:

```bash
java -version
mvn -v
docker -v
docker compose version
```
## Dependencias

- Spring Web
- Spring Data JPA
- Validation
- Lombok
- MySQL Driver
- Flyway
- Spring Boot Actuator
- Spring Boot DevTools
- SpringDoc OpenAPI Web (Swagger)


---

## 🔌 Puertos utilizados

| Servicio            | Puerto expuesto |
|--------------------|--------|
| Aplicación (dev)   | 8081   |
| Aplicación (prod)  | 8082   |
| MySQL (dev)        | 3307   |
| MySQL (prod)       | 3308   |

---

## 🔄 Diferencia entre DEV y PROD

| Modo | Ejecución           | Base de datos | Puerto |
|------|--------------------|--------------|--------|
| DEV  | Maven              | Docker       | 8081   |
| PROD | Docker Compose     | Docker       | 8082   |

---

## Base de datos y migraciones

Convención actual:

- Los cambios de esquema deben quedar en SQL versionado
- Flyway ejecuta automáticamente scripts en `src/main/resources/db/migration` cuando arranca `prod`
- Ejemplo actual: `V1__create_categoria_table.sql`
- En `prod`, Hibernate no crea tablas; solo valida el esquema existente

Flujo recomendado del equipo:

1. Diseñar o ajustar la tabla en SQL
2. Probar el cambio en `dev`
3. Crear una nueva versión SQL si corresponde (`V2`, `V3`, etc.)
4. Aplicar el cambio en `prod`
5. Arrancar la app en `prod` y validar

No modificar scripts ya ejecutados; crear siempre una nueva versión.

# 🚀 Ejecución en modo desarrollo (dev)

## 🔹 1. Clonar repositorio - rama inicial

Ejemplo:

```bash
git clone git clone --branch vs01-arquitectura-base https://github.com/261dist/catalogo.git

cd catalogo
```

---

## 🔹 2. Levantar base de datos

```bash
docker compose -f docker-compose-dev.yml up
```
**Si no tienes Docker, otra opción es con Laragon, XAMPP, o MySQL local**

- Asegúrate de que MySQL esté corriendo en tu máquina local.
- Verifica la configuración en `src/main/resources/application-dev.yml` (host, puerto, usuario, contraseña).
- Confirma que la BD `db_catalogo` existe.


---

## 🔹 3. Ejecutar aplicación

```bash
mvn spring-boot:run
```

👉 Perfil activo: `dev`

---

## 🌐 Acceso DEV
Swagger:
```
http://localhost:8081/swagger-ui.html
```
Health: 
```
http://localhost:8081/actuator/health
```

---

# 🐳 Ejecución en modo producción (prod)

## 🔹 1. Crear archivo `.env`

```env
CATALOGO_MYSQL_ROOT_PASSWORD=root
CATALOGO_MYSQL_DATABASE=db_catalogo

SPRING_PROFILES_ACTIVE=prod

CATALOGO_DB_HOST=mysql-catalogo
CATALOGO_DB_PORT=3306
CATALOGO_DB_NAME=db_catalogo
CATALOGO_DB_USERNAME=root
CATALOGO_DB_PASSWORD=root
```

---

## 🔹 2. Levantar servicios

```bash
docker compose -f docker-compose.yml up -d
```

---

## 🌐 Acceso PROD
API Categorias:
```
http://localhost:8082/api/v1/categorias
```
Health:
```
http://localhost:8082/actuator/health
```
Swagger: deshabilitado

---

# 📈 Escalado de la aplicación (múltiples instancias)

---

## 🔹 Paso 1. Detener entorno previo

Antes de iniciar el escalado, detener los servicios que estuvieran levantados previamente:

```bash
docker compose -f docker-compose.yml down
```
Si existieran contenedores manuales anteriores, también eliminarlos:

```bash
docker rm -f catalogo1 catalogo2 catalogo3
```

---

## 🔹 Paso 2. Levantar solo MySQL

Levantar únicamente la base de datos del entorno de producción:

```bash
docker compose -f docker-compose.yml up -d mysql-catalogo
```

---

## 🔹 Paso 3. Construir imagen

Generar la imagen Docker de la aplicación:

```bash
docker build -t catalogo-service .
```

---

## 🔹 Paso 4. Ejecutar instancias

> ⚠️ Nota: En este escenario usamos la red por defecto creada por Docker Compose.

### Instancia 1

```bash
docker run --name catalogo1 \
  --network catalogo-net \
  --env-file .env \
  -p 8082:8082 \
  catalogo-service
```

### Instancia 2

```bash
docker run -d --name catalogo2 \
  --network catalogo-net \
  --env-file .env \
  -p 8083:8082 \
  catalogo-service
```

---

## 🔹 Paso 5. Verificar

```bash
docker ps
```

---

## 🔹 Paso 6. Probar

- http://localhost:8082
- http://localhost:8083

---

## 🔹 Paso 7. Finalizar

🧠 Tip pro 

stop → apaga contenedor (como apagar servidor)

rm → elimina contenedor (como borrar VM)

rmi → elimina imagen (como borrar ISO/base)
```bash

docker stop catalogo1
docker rm catalogo1
docker rmi catalogo-service   # opcional
```
```bash
docker rm -f catalogo1 catalogo2 catalogo3
docker compose -f docker-compose.yml down
```
Para ver todos los servicios (incluidos apagados):
```bash
docker ps -a
```
Detener todos los contenedores
```bash
docker stop $(docker ps -q)
```
Eliminar todos los contenedores
```bash
docker rm $(docker ps -aq)
```
Eliminar todas las imágenes
```bash
docker rmi $(docker images -q)
```
🚀 Eliminar contenedores detenidos, imágenes no usadas, redes, cache
```bash
docker system prune -a
```
💣 Forma EXTREMA (incluye volúmenes)
```bash
docker system prune -a --volumes
```
---

## 🔹 Ejecución sin `.env` (opcional)

```bash
docker run -d --name catalogo1 \
  --network catalogo_default \
  -p 8082:8082 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e CATALOGO_DB_HOST=mysql-catalogo \
  -e CATALOGO_DB_PORT=3306 \
  -e CATALOGO_DB_NAME=db_catalogo \
  -e CATALOGO_DB_USERNAME=root \
  -e CATALOGO_DB_PASSWORD=root \
  catalogo-service
```

---

# 🔗 Integración futura

## Config Server

```properties
SPRING_CONFIG_IMPORT=optional:configserver:http://config-server:7071
```

## Eureka

```properties
EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://registry-server:8761/eureka
```

## Gateway

```yaml
uri: lb://catalogo
```

---

# ⚠️ Alcance actual

Este proyecto NO incluye aún:

- API Gateway
- Eureka
- Config Server
- Balanceador

---

# 🧠 Concepto clave

Este proyecto es un microservicio que:

- es independiente
- puede escalar
- se integrará progresivamente

---

# 📌 Nota final

Este repositorio forma parte de una arquitectura de microservicios en evolución.

# 🔧 ANEXO PR (lujo de trabajo con Git)

Este flujo permite trabajar con ramas, enviar cambios y versionar el proyecto de forma ordenada.

---

## 🔹 1. Actualizar repositorio

```bash
git branch
git pull origin main
```

---

## 🔹 2. Crear rama de trabajo y hacer el trabajo

```bash
git checkout -b tarea/avance
```
###  ATENCIÓN: NO codees sin hacer los 2 pasos anteriores



## 🔹 3. Realizar cambios

```bash
git add .
git commit -m "feat: avance"
git push -u origin tarea/avance
```

---

## 🔹 4. Volver a main y limpiar rama

```bash
git checkout main
git pull origin main

git branch -d tarea/avance
git push origin --delete tarea/avance
```

---

## 🔹 5. Crear tag (versión estable)

```bash
git tag -a vs01 -m "versión estable"
git push origin vs01
```

---

## 🔹 6. Eliminar tag (si es necesario)

```bash
git tag -d vs01
git push origin --delete vs01
```

---

## 📚 Documentación adicional

Ver documentación operativa en:

👉 https://upeuoficial.github.io/carrera-sistemas-docs-operativos/