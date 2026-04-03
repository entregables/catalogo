# Catálogo

Este proyecto es un microservicio de catálogo desarrollado en Java con Spring Boot. Proporciona una API REST para la gestión de categorías y otros recursos relacionados.

## Estructura del proyecto

- **src/main/java/com/upeu/catalogo/**: Código fuente principal.
- **src/main/resources/**: Archivos de configuración y recursos.
- **src/test/java/com/upeu/catalogo/**: Pruebas unitarias y de integración.
- **Dockerfile** y **docker-compose.yml**: Archivos para contenerización y orquestación.

## Requisitos

- Java 17+
- Maven 3.8+
- Docker (opcional, para despliegue en contenedores)


## Ejecución en modo desarrollo (dev)

### Usando Maven y base de datos MySQL en docker
1. Clona el repositorio.
2. Levanta la base de datos MySQL para desarrollo:
   ```sh
   docker compose -f docker-compose-dev.yml up 
   ```
3. Ejecuta la aplicación fuera del IDE, perfil por defecto es `dev`:
   ```sh
   mvn spring-boot:run
   ```
4. Accede a la API en: [http://localhost:8081](http://localhost:8081)

## Ejecución en modo producción (prod)

### Usando Docker Compose
1. Configura las variables de entorno necesarias en un archivo `.env` (ver ejemplo abajo).
2. Levanta todos los servicios (app y base de datos):
   ```sh
   docker compose -f docker-compose.yml up -d
   ```
3. Accede a la API en: [http://localhost:8082](http://localhost:8082)

#### Ejemplo de archivo `.env` para producción
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


## Escalado de la aplicación (varias instancias)

Para escalar la aplicación y que todas las instancias consuman la misma base de datos, sigue estos pasos:

### 1. Construye la imagen de la app (si no lo hiciste antes):
```sh
docker build -t catalogo-service .
```

### 2. Ejecuta múltiples instancias de la app (no probado)
Puedes lanzar varias instancias manualmente cambiando el puerto de mapeo. Ejemplo usando los valores del archivo `.env`:
```powershell
# Instancia 1 (puerto 8082)
docker run -d --name catalogo1 `
   -p 8082:8082 `
   -e SPRING_PROFILES_ACTIVE=prod `
   -e CATALOGO_DB_HOST=mysql-catalogo `
   -e CATALOGO_DB_PORT=3306 `
   -e CATALOGO_DB_NAME=db_catalogo `
   -e CATALOGO_DB_USERNAME=root `
   -e CATALOGO_DB_PASSWORD=root `
   catalogo-service

# Instancia 2 (puerto 8083)
docker run -d --name catalogo2 `
   -p 8083:8082 `
   -e SPRING_PROFILES_ACTIVE=prod `
   -e CATALOGO_DB_HOST=mysql-catalogo `
   -e CATALOGO_DB_PORT=3306 `
   -e CATALOGO_DB_NAME=db_catalogo `
   -e CATALOGO_DB_USERNAME=root `
   -e CATALOGO_DB_PASSWORD=root `
   catalogo-service
```

# Instancia 2 (puerto 8083)
docker run -d --name catalogo2b `
  --network catalogo-net `
  -p 8083:8082 `
  -e SPRING_PROFILES_ACTIVE=prod `
  -e CATALOGO_DB_HOST=mysql-catalogo `
  -e CATALOGO_DB_PORT=3306 `
  -e CATALOGO_DB_NAME=db_catalogo `
  -e CATALOGO_DB_USERNAME=root `
  -e CATALOGO_DB_PASSWORD=root `
  catalogo-service
```

O usando Docker Compose, puedes escalar con:
```sh
docker compose -f docker-compose.yml up --scale catalogo=3 -d
```
Esto levantará 3 instancias de la app usando la misma base de datos.

### 3. Configuración necesaria para compartir la base de datos
- Todas las instancias deben apuntar al mismo host, puerto, nombre de base de datos, usuario y contraseña (variables de entorno).
- La base de datos debe estar accesible en red para todas las instancias.
- No escales la base de datos MySQL con múltiples contenedores independientes; usa una sola instancia o un clúster gestionado.
- Si usas balanceador de carga, enruta el tráfico a las distintas instancias de la app.

**Importante:**
- No necesitas cambiar nada en el código para escalar la app, solo asegúrate de que las variables de entorno de conexión a la base de datos sean iguales en todas las instancias.
- Si usas Docker Compose, revisa que el servicio `mysql-catalogo` esté definido como único y el servicio `catalogo` sea el que escales.

## Documentación OpenAPI

La documentación de la API está disponible en `http://localhost:8081/swagger-ui/index.html` cuando la aplicación está en ejecución.

## Autor

- Angel Sullon Macalupu - 2026
