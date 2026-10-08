# Evaluación Parcial N.º 2 – Ingeniería DevOps

## 1. Descripción del proyecto

Esta evaluación amplía el microservicio de gestión de ingredientes desarrollado con Java 21, Spring Boot, Maven y H2.

El objetivo es implementar un pipeline CI/CD mediante GitHub Actions que automatice la compilación, las pruebas, el análisis de calidad y seguridad, la construcción de la imagen Docker y el despliegue en un entorno simulado.


## 2. Contenedorización con Docker 

Se implementó un `Dockerfile` multietapa para contenerizar el microservicio.

La primera etapa utiliza Maven y Java 21 para compilar la aplicación y generar el archivo JAR.

La segunda etapa utiliza Eclipse Temurin JRE 21 para ejecutar el microservicio, evitando incluir las herramientas de compilación en la imagen final.

El servicio utiliza el puerto `7073`.

Construcción de la imagen:

```bash
docker build -t msvc-ingrediente .
```

Esta implementación permite ejecutar la aplicación en un entorno reproducible y facilita su despliegue mediante Docker Compose.

## 3. Pruebas automatizadas con JUnit y cobertura JaCoCo

Se implementaron pruebas automatizadas utilizando JUnit, integradas con Maven y GitHub Actions.

### Justificación de JUnit frente a Pytest

Se eligió JUnit porque el microservicio está desarrollado en Java 21 y utiliza Spring Boot.

JUnit permite probar directamente las clases Java y se integra con Maven para ejecutar las pruebas durante el pipeline.

Pytest es una herramienta orientada principalmente a proyectos Python. Aunque puede utilizarse para pruebas externas de una API, no resulta la alternativa más directa para las pruebas unitarias de este microservicio.

### Resultados de las pruebas

En la última ejecución local se obtuvieron:

| Indicador | Resultado |
|---|---:|
| Pruebas ejecutadas | 25 |
| Fallidas | 0 |
| Errores | 0 |
| Omitidas | 0 |
| Compilación | BUILD SUCCESS |

Comando utilizado:

```bash
./mvnw clean verify
```

### Cobertura con JaCoCo

Se incorporó JaCoCo `0.8.13` para medir la cobertura de las pruebas automatizadas.

Los reportes se generan en:

- `target/site/jacoco/index.html`
- `target/site/jacoco/jacoco.xml`

Durante el desarrollo se observó aproximadamente un 49 % de cobertura de instrucciones en el reporte local y un 46,6 % de cobertura global en SonarQube Cloud.

El reporte XML se integra con SonarQube para complementar el análisis de calidad.

## 4. Seguridad y calidad con Dependabot y SonarQube 

### Dependabot

Se configuró Dependabot mediante:

`.github/dependabot.yml`

Esta herramienta permite identificar actualizaciones disponibles de las dependencias del proyecto y generar Pull Requests automáticos para su revisión.

Durante el desarrollo se observaron Pull Requests generados por Dependabot.

### SonarQube Cloud

Se integró SonarQube Cloud para analizar la calidad y seguridad del código.

El análisis permite identificar problemas relacionados con:

- Confiabilidad.
- Mantenibilidad.
- Seguridad.
- Cobertura.
- Duplicación de código.

### Justificación de SonarQube frente a Snyk

Se eligió SonarQube porque permite analizar la calidad del código Java, identificar problemas de mantenibilidad y confiabilidad, incorporar la cobertura de JaCoCo y evaluar los cambios mediante un Quality Gate.

Snyk también es una alternativa válida y destaca por el análisis de vulnerabilidades en dependencias, código y contenedores.

Para este proyecto se priorizó SonarQube por su integración con Maven, JaCoCo y GitHub Actions, mientras que Dependabot complementa el seguimiento de dependencias.

### Quality Gate y control de fallos

El análisis se ejecuta desde GitHub Actions utilizando el secreto `SONAR_TOKEN`.

En la rama `main`, el pipeline espera el resultado del Quality Gate mediante:

`-Dsonar.qualitygate.wait=true`

Si el Quality Gate falla, el paso de análisis puede detener el pipeline antes de construir y desplegar el contenedor.

En los Pull Requests se utiliza además la verificación independiente de SonarQube Cloud.

Durante el desarrollo se comprobó que una versión inicial del Pull Request #26 no aprobó el Quality Gate. Después de corregir el problema, el análisis fue aprobado.

En el Pull Request #27 se obtuvo:

| Indicador | Resultado |
|---|---|
| Quality Gate | Passed |
| Nuevos issues | 0 |
| Security Hotspots | 0 |
| Cobertura del código nuevo | 100 % |
| Duplicación del código nuevo | 0 % |

Estas verificaciones permiten detectar problemas y corregirlos antes de integrar cambios.

## 5. Pipeline CI/CD y despliegue automatizado 

El pipeline está configurado en:

`.github/workflows/ci.yml`

### Eventos de ejecución

Se ejecuta automáticamente:

- Al realizar un push a `develop`.
- Al realizar un push a `main`.
- Al crear o actualizar un Pull Request hacia `main`.

### Etapas del pipeline

1. Descargar el código mediante `actions/checkout@v4`.
2. Configurar Java 21 mediante `actions/setup-java@v4`.
3. Asignar permisos al Maven Wrapper.
4. Compilar y ejecutar las pruebas JUnit.
5. Generar la cobertura con JaCoCo.
6. Ejecutar el análisis de SonarQube Cloud.
7. Construir la imagen Docker.
8. Desplegar el microservicio con Docker Compose.
9. Verificar la respuesta HTTP del microservicio.

### Construcción de imagen

```bash
docker build -t msvc-ingrediente .
```

### Despliegue automático

```bash
docker compose up -d --build
```

### Verificación del despliegue

El pipeline consulta el endpoint:

`http://localhost:7073/api/ingredientes`

Se realizan hasta diez intentos, con una espera de cinco segundos entre ellos.

Si el microservicio no responde correctamente, el pipeline finaliza con error y muestra los logs del contenedor.

El despliegue se realiza dentro del runner temporal de GitHub Actions, por lo que corresponde a un entorno simulado y no a un servidor de producción permanente.

### Trazabilidad y calidad

La trazabilidad se mantiene mediante Git, commits, ramas, Pull Requests y ejecuciones de GitHub Actions.

Cada cambio puede relacionarse con las pruebas ejecutadas, el análisis de SonarQube y el resultado del pipeline.

Los Pull Requests #26 y #27 permiten comprobar cómo se identificaron, corrigieron y validaron problemas de calidad del código.

## 6. Orquestación con Docker Compose 

Se implementó Docker Compose mediante el archivo:

`docker-compose.yml`

La configuración permite construir y ejecutar el microservicio, definir límites de recursos y verificar su estado.

### Justificación de Docker Compose frente a Kubernetes

Se eligió Docker Compose porque el proyecto utiliza un único microservicio y requiere desplegarlo en un entorno simulado.

Docker Compose permite definir y ejecutar el servicio mediante un archivo de configuración, sin necesidad de implementar un clúster.

Kubernetes ofrece funcionalidades más avanzadas de orquestación, escalamiento y recuperación de contenedores, pero requiere una infraestructura y configuración más complejas.

Para el alcance de esta evaluación, Docker Compose permite cumplir el objetivo de ejecutar y verificar el microservicio de manera automatizada.

### Configuración implementada

| Parámetro | Valor |
|---|---|
| Puerto | 7073 |
| Memoria máxima | 512 MB |
| Límite de CPU | 0.50 |
| Intervalo de healthcheck | 30 segundos |
| Timeout | 10 segundos |
| Reintentos | 3 |
| Período inicial | 20 segundos |

### Comandos

Iniciar el servicio:

```bash
docker compose up -d --build
```

Verificar el estado:

```bash
docker compose ps
```

Detener el servicio:

```bash
docker compose down
```

Durante la verificación local, el contenedor alcanzó el estado `healthy`, confirmando que el endpoint utilizado por el healthcheck respondía correctamente.

## 7. Evidencias y trazabilidad de la evaluación

| Indicador | Implementación | Evidencia |
|---|---|---|
| IE1 – Contenedores | Dockerfile multietapa y construcción de imagen | `Dockerfile` y GitHub Actions |
| IE2 – Pruebas automatizadas | JUnit y JaCoCo | 25 pruebas exitosas y reporte de cobertura |
| IE3 – Seguridad y calidad | Dependabot, SonarQube y Quality Gate | Configuración, análisis y PR #26 y #27 |
| IE4 – Pipeline y despliegue | GitHub Actions y Docker Compose | `ci.yml` y ejecuciones exitosas |
| IE5 – Orquestación | Docker Compose con límites y healthcheck | `docker-compose.yml` y estado `healthy` |
