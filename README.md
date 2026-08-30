# Evaluación Parcial N°1 - Ingeniería DevOps

## Flujo de trabajo GitFlow

Se utilizó GitFlow para tener un manejo más controlado y estructurado de las ramas al
momento de desarrollar y subir cambios al proyecto. Como estamos trabajando con parte de
un microservicio, este podría crecer a futuro y necesitar la participación de más
desarrolladores. GitFlow nos permite separar cada funcionalidad o corrección en su propia rama,
evitando interferir con el trabajo de los demás integrantes y manteniendo una estructura más
estable y organizada al momento de integrar los cambios.


## Nomenclatura de ramas

Para mantener un orden dentro del repositorio se definió la siguiente nomenclatura para
la creación de ramas:

- `main`: rama principal del proyecto.
- `develop`: rama principal de desarrollo.
- `feature/<nombre-descriptivo>`: utilizada para nuevas funcionalidades o cambios durante
  el desarrollo. Ejemplo: `feature/configuracion-h2`.
- `hotfix/<nombre-descriptivo>`: utilizada para correcciones sobre una versión estable.
  Ejemplo: `hotfix/correccion-location-header`.

Los nombres utilizados después de `feature/` y `hotfix/` deben ser descriptivos y estar
relacionados con el cambio que se realizará en la rama.


## Convención de commits

Para mantener un historial ordenado y facilitar la identificación de los cambios realizados,
se utilizaron mensajes de commit breves y descriptivos.

Se utilizaron los siguientes prefijos cuando correspondía:

- `docs:` para cambios relacionados con documentación.
- `ci:` para cambios relacionados con integración continua.
- `hotfix:` para correcciones realizadas mediante una rama hotfix.

Algunos ejemplos de commits realizados durante el desarrollo fueron:

- `docs: se agrega documentacion de endpoint`
- `hotfix: corregir cabecera al crear ingrediente`
- `se agrega el workflow de integracion continua`

El objetivo de esta convención es poder identificar de forma rápida qué cambio se realizó,
manteniendo una mejor trazabilidad dentro del historial del repositorio.


## Flujo de merge y estrategia de revisión

Para integrar los cambios entre las diferentes ramas se utilizaron Pull Requests en GitHub.

Los cambios desarrollados en ramas `feature` fueron integrados primero a `develop`.
Posteriormente, los cambios de `develop` fueron integrados a `main` mediante un Pull Request.

Para las correcciones realizadas mediante `hotfix`, se creó una rama desde `main`.
Una vez realizada la corrección, esta fue integrada nuevamente a `main` mediante Pull Request
y posteriormente sincronizada con `develop`.

Antes de realizar cada merge se revisaron los archivos modificados y los cambios incluidos
en el Pull Request. Además, GitHub Actions permitió realizar una validación automática mediante
la ejecución de las pruebas del proyecto.


## Integración continua con GitHub Actions

Para implementar integración continua se utilizó GitHub Actions mediante el archivo
`.github/workflows/ci.yml`.

El workflow fue configurado para ejecutarse automáticamente:

- Cada vez que se realiza un `push` a la rama `develop`.
- Cada vez que se crea o actualiza un Pull Request cuyo destino es `main`.

El workflow realiza los siguientes pasos:

1. Descarga el código del repositorio.
2. Configura Java 21.
3. Entrega permisos de ejecución al Maven Wrapper.
4. Ejecuta las pruebas mediante `./mvnw clean test`.

Esto permite validar automáticamente las pruebas del proyecto antes de integrar nuevos cambios.


## Estructura del proyecto

```text
eva1-devops/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .mvn/
├── docs/
│   └── ENDPOINTS.md
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
├── Dockerfile
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md