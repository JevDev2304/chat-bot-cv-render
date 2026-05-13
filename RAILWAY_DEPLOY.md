# Despliegue en Railway con Docker

Guía paso a paso para desplegar el backend del chatbot CV en [Railway](https://railway.app) usando Docker.

---

## Prerrequisitos

- Cuenta en [Railway](https://railway.app) (puedes registrarte con GitHub)
- Repositorio subido a GitHub
- API Keys de Gemini y Pinecone a la mano
- Docker instalado localmente (opcional, para pruebas locales)

---

## Paso 1 — Crear el Dockerfile

En la raíz del proyecto, crea un archivo llamado `Dockerfile` con el siguiente contenido:

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src
RUN chmod +x gradlew && ./gradlew bootJar -x test --no-daemon

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

> Este Dockerfile usa una construcción en dos etapas (*multi-stage build*): la primera compila el proyecto con Gradle y la segunda genera una imagen más liviana solo con el JAR final.

---

## Paso 2 — Crear el proyecto en Railway

1. Inicia sesión en [railway.app](https://railway.app)
2. Haz clic en **New Project**
3. Selecciona **Deploy from GitHub repo**
4. Autoriza Railway para acceder a tu cuenta de GitHub
5. Busca y selecciona el repositorio `chat-bot-cv-back-end`

Railway detectará el `Dockerfile` automáticamente y lo usará para construir la imagen.

---

## Paso 3 — Configurar las variables de entorno

1. Ve a la pestaña **Variables** del servicio
2. Agrega las siguientes variables:

| Variable | Valor |
|---|---|
| `GEMINI_API_KEY` | Tu API Key de Google Gemini |
| `GEMINI_API_URL` | `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-lite:generateContent` |
| `PINECONE_API_KEY` | Tu API Key de Pinecone |
| `PINECONE_URL` | La URL de tu índice en Pinecone |

> **Nota:** Railway inyecta la variable `PORT` automáticamente, no es necesario agregarla.

---

## Paso 4 — Desplegar

1. Haz clic en **Deploy**
2. Ve a la pestaña **Deployments** para ver los logs en tiempo real
3. Espera a que el estado cambie a **Active** ✓

---

## Paso 5 — Obtener la URL pública

1. Ve a la pestaña **Settings** del servicio
2. En la sección **Networking**, haz clic en **Generate Domain**
3. Railway te asignará una URL del tipo:
   ```
   https://chat-bot-cv-back-end-production.up.railway.app
   ```

---

## Paso 6 — Verificar el despliegue

Prueba el endpoint de salud con curl o desde el navegador:

```bash
curl https://<tu-url>.up.railway.app/actuator/health
```

Respuesta esperada:

```json
{ "status": "UP" }
```

---

## Solución de problemas comunes

| Problema | Solución |
|---|---|
| `Port binding failed` | Railway maneja el puerto con `$PORT`, verificar que `application.yml` use `${PORT:8081}` |
| `Build failed` | Verificar que el proyecto compile localmente con `./gradlew bootJar -x test` |
| `Application failed to start` | Verificar que todas las variables de entorno estén configuradas |
| `401 Unauthorized` en Gemini/Pinecone | Revisar que las API Keys sean correctas y tengan permisos |
