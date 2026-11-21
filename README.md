# Practica-DevOps-GitHub-Docker-Jenkins

Pipeline DevOps con **GitHub, Docker y Jenkins** aplicado al proyecto **Glosario TI**, una aplicación web para consultar términos de Tecnologías de la Información.

## La aplicación

- **Frontend:** React + TypeScript, servido con Nginx.
- **Backend:** Node.js + Express, autenticación con JWT y roles (`usuario` / `admin`).
- **Base de datos:** MongoDB.
- **Funciones:** búsqueda de términos, filtros por categoría y sistema de favoritos.

## Estructura

```
.
├── backend/              API REST (Express + Mongoose) y pruebas
├── frontend/             Aplicación React y configuración de Nginx
├── docker-compose.yml    Frontend + backend + MongoDB
├── Jenkinsfile           Pipeline de integración continua
└── .env.example          Variables de entorno de ejemplo
```

## Ejecución con Docker

```bash
cp .env.example .env        # y ajusta JWT_SECRET
docker compose up -d --build
```

- Aplicación: http://localhost:8081
- API: http://localhost:5000

Para detenerla: `docker compose down`.

### Imágenes por separado

```bash
docker build -t glosario-backend:1.0 backend
docker build -t glosario-frontend:1.0 frontend
docker run -d -p 5000:5000 -e MONGO_URI=... -e JWT_SECRET=... glosario-backend:1.0
```

## Pruebas

```bash
cd backend
npm ci
npm test
```

El `Dockerfile` del backend incluye una etapa `test`, que Jenkins usa para validar el código antes de construir la imagen final.

## Pipeline de Jenkins

El `Jenkinsfile` define estas etapas:

1. **Obtener código:** descarga el repositorio desde GitHub.
2. **Pruebas del backend:** `docker build --target test`.
3. **Construir imágenes Docker:** backend y frontend, etiquetadas con el número de build y `latest`.
4. **Ejecutar contenedores:** `docker compose up -d --build`.
5. **Verificar aplicación:** comprueba que backend y frontend respondan.
6. **Limpieza:** `docker compose down -v` al terminar.

### Jenkins en Docker

La carpeta `jenkins/` levanta un Jenkins (LTS) en un contenedor, con el cliente de Docker y los plugins Git, GitHub, Pipeline y Docker Pipeline ya instalados. El contenedor usa el Docker del equipo mediante `/var/run/docker.sock`.

```bash
cd jenkins
docker compose up -d --build
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

1. Abre http://localhost:8080 y pega la contraseña inicial.
2. Crea el usuario administrador.
3. Crea un job de tipo **Pipeline** con "Pipeline script from SCM": SCM Git, la URL de este repositorio, rama `*/master` y script `Jenkinsfile`.
4. Pulsa **Construir ahora**.

Los puertos 8080 (Jenkins), 5000 (API) y 8081 (aplicación) deben estar libres.

### Requisitos del servidor Jenkins

- Docker y el plugin Docker Compose disponibles para el usuario `jenkins`.
- Plugins: Git, GitHub, Pipeline.
- Job de tipo **Pipeline** con "Pipeline script from SCM" apuntando a este repositorio.
- Opcional: webhook de GitHub hacia `http://<servidor-jenkins>/github-webhook/`.

## Variables de entorno

| Variable | Descripción |
|---|---|
| `JWT_SECRET` | Secreto para firmar los tokens |
| `MONGO_URI` | Conexión a MongoDB (por defecto, el contenedor `mongo`) |

Nunca subas el archivo `.env` al repositorio.
