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

> Este repositorio implementa únicamente el microservicio **Catálogo**.

---

## 🧰 Pre-requisitos

Antes de ejecutar el proyecto, asegúrate de tener instalado:

- Java 17
- Maven 3.9+
- Docker
- Docker Compose

Verificar instalación:

```bash
java -version
mvn -v
docker -v
docker compose version
```

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

## 🧠 ¿Por qué funciona `mysql-catalogo` como host?

Dentro de Docker Compose, los contenedores pueden comunicarse usando el nombre del servicio.

Ejemplo:

```env
CATALOGO_DB_HOST=mysql-catalogo
```

Esto funciona automáticamente porque Docker Compose crea una red interna por defecto.

---

# 🚀 Ejecución en modo desarrollo (dev)

## 🔹 1. Clonar repositorio

```bash
git clone <repo>
cd catalogo
```

---

## 🔹 2. Levantar base de datos

```bash
docker compose -f docker-compose-dev.yml up
```

---

## 🔹 3. Ejecutar aplicación

```bash
mvn spring-boot:run
```

👉 Perfil activo: `dev`

---

## 🌐 Acceso DEV

```
http://localhost:8081
```

---

# 🐳 Ejecución en modo producción (prod)

## 🔹 1. Crear archivo `.env`

```env
CATALOGO_MYSQL_ROOT_PASSWORD=una_clave_segura
CATALOGO_MYSQL_DATABASE=db_catalogo

SPRING_PROFILES_ACTIVE=prod

CATALOGO_DB_HOST=mysql-catalogo
CATALOGO_DB_PORT=3306
CATALOGO_DB_NAME=db_catalogo
CATALOGO_DB_USERNAME=root
CATALOGO_DB_PASSWORD=una_clave_segura
```

---

## 🔹 2. Levantar servicios

```bash
docker compose -f docker-compose.yml up -d
```

---

## 🌐 Acceso PROD

```
http://localhost:8082
```

---

# 📈 Escalado de la aplicación (múltiples instancias)

---

## 🔹 Paso 1. Detener entorno previo

```bash
docker compose -f docker-compose.yml down
docker rm -f catalogo1 catalogo2 catalogo3
```

---

## 🔹 Paso 2. Levantar solo MySQL

```bash
docker compose -f docker-compose.yml up -d mysql-catalogo
```

---

## 🔹 Paso 3. Construir imagen

```bash
docker build -t catalogo-service .
```

---

## 🔹 Paso 4. Ejecutar instancias

> ⚠️ Nota: En este escenario usamos la red por defecto creada por Docker Compose.

### Instancia 1

```bash
docker run -d --name catalogo1 \
  --network catalogo_default \
  --env-file .env \
  -p 8082:8082 \
  catalogo-service
```

### Instancia 2

```bash
docker run -d --name catalogo2 \
  --network catalogo_default \
  --env-file .env \
  -p 8083:8082 \
  catalogo-service
```

### Instancia 3

```bash
docker run -d --name catalogo3 \
  --network catalogo_default \
  --env-file .env \
  -p 8084:8082 \
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
- http://localhost:8084

---

## 🔹 Paso 7. Finalizar

```bash
docker rm -f catalogo1 catalogo2 catalogo3
docker compose -f docker-compose.yml down
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

# ⚠️ Problemas comunes

## ❌ No conecta a MySQL

Verificar:

```bash
docker ps
```

- mysql-catalogo está activo
- nombre correcto: mysql-catalogo

---

## ❌ Puerto ocupado

```bash
docker ps
docker stop <container>
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