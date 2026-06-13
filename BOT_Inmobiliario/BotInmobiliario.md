### TL;DR
- **Producto:** Ecosistema B2B/B2C (Casado Props), un agente autónomo de misión crítica que trasciende el rol de chatbot para convertirse en un **Co-Worker Inmobiliario**. Ejecuta acciones mutacionales (crear tareas, tickets, registrar pagos) heredando el RBAC del usuario.
- **Arquitectura & Infra:** Topología Docker Compose orquestada detrás de Traefik (Zero-Trust ingress). Stack monolítico altamente optimizado con PostgreSQL 16 + pgvector, Redis 7 (control de estado) y n8n + Next.js (Cognición y UI/API).
- **Seguridad & DevSecOps:** Aprovisionamiento inmutable vía `deploy.sh` con segregación UID/GID. WAF Semántico multimodal impulsado por Gemini 2.5 Flash, capaz de transcribir audios on-the-fly y mitigar ataques de *Prompt Injection* antes de tocar el agente principal.
- **Gobierno Agéntico:** Framework ReAct estricto. El bot reutiliza 60+ API Routes REST con validación Zod, forzando un paradigma de "Propuesta → Confirmación UI → Ejecución" para mutaciones, garantizando trazabilidad total en el CRM.

---

# 🏢 Bot_Inmobiliaria (Casado Props): Ecosistema Autónomo Co-Worker

El proyecto **Casado Props** redefine la automatización inmobiliaria mediante la implementación de un agente cognitivo (Co-Worker) integrado nativamente en un dashboard Next.js. Su valor radica en la capacidad de ejecutar flujos de trabajo operativos (consultas de catálogo, gestión de tareas, CRM y mantenimiento) interpretando lenguaje natural o **notas de voz**, aplicando invariablemente las políticas de seguridad y los permisos del usuario que emite la solicitud.

A continuación, se detalla la ingeniería L2/L3 de su infraestructura, stack tecnológico y gobierno agéntico.

---

## 🏗️ 1. Arquitectura y Topología de Infraestructura (L2/L3)

El ecosistema fue diseñado bajo el paradigma de **Infraestructura Inmutable** y **Zero-Trust Network Architecture**. Se prescindió de túneles externos (ej. Cloudflare) en favor de **Traefik Reverse Proxy** para manejar el ingress L7 de manera nativa, facilitando la gestión de dominios y la generación automática de certificados Let's Encrypt.

### Stack de Aplicación y Topología Docker

Todos los servicios operan confinados en una red bridge (`casadoprop-net`), limitando la exposición superficial únicamente a Traefik.

1. **`casadoprop-db` (PostgreSQL 16 + `pgvector`):** Base de datos consolidada. Soporta tanto el estado transaccional de Next.js como la memoria de LangChain para n8n.
2. **`casadoprop-redis` (Redis 7 Alpine):** Gestor de estado volátil. Se utiliza para colas asíncronas y para el control de concurrencia estricto.
3. **`casadoprop-n8n`:** Motor cognitivo principal y orquestador de flujos. Restringido detrás de Traefik.
4. **`casadoprop-app` (Next.js + Prisma):** Interfaz de usuario y Backend API. Expone más de 60 endpoints REST validados con Zod que el bot consume como herramientas (Tools).

**Fragmento representativo de segmentación y límites (docker-compose.yml):**
```yaml
  n8n:
    image: n8nio/n8n:latest
    container_name: casadoprop-n8n
    environment:
      - WEBHOOK_URL=http://n8n:5678
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - DB_TYPE=postgresdb
      - EXECUTIONS_MODE=regular
    networks:
      - casadoprop-net
    deploy:
      resources:
        limits:
          cpus: '0.8'
          memory: 1.5G
        reservations:
          memory: 512M
```
*Justificación:* El uso de `reservations` y `limits` asegura que el entorno cognitivo (n8n) no asfixie la base de datos o el frontend en picos de alta demanda computacional (ej. procesamiento masivo de webhooks).

---

## 🛡️ 2. DevSecOps y Aprovisionamiento Determinístico

### Orquestador Inmutable: `deploy.sh` (L2)
Se diseñó un operador Bash para estandarizar el despliegue. Este script valida pre-requisitos y asegura la correcta segregación de privilegios en los volúmenes, previniendo errores de permisos en contenedores no-root.

```bash
# PREPARACIÓN DE ENTORNO Y PERMISOS
mkdir -p postgres_data redis_data n8n_data

# Segregación estricta de UID/GID para evitar exploits de escalada
chown -R 1000:1000 n8n_data
chown -R 999:999 postgres_data redis_data

chmod 755 deploy.sh init-multiple-dbs.sh
chmod 644 init-tables.sql

# DESPLIEGUE ATÓMICO
docker compose up -d --build
docker image prune -f
```

### High Availability y Resiliencia en BD: `init-multiple-dbs.sh` (L3)
Para garantizar la creación limpia y aislada de bases de datos (`n8n_db` y `casadoprop`) dentro de un único clúster Postgres, se implementó un script que respeta la directiva `ON_ERROR_STOP=1`.

**El problema resuelto:** Ejecutar `\c` (connect) en un bloque SQL rompe el contexto de la directiva de error. 
**La solución:** Se utilizó el metacomando `\gexec` y bloques `psql` separados para garantizar la **idempotencia total**.

```bash
# PASO 1: Crear DB usando metacomando \gexec para forzar evaluación dinámica
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" -d postgres <<EOSQL
SELECT 'CREATE DATABASE $db'
WHERE NOT EXISTS (SELECT FROM pg_database WHERE datname = '$db')
\gexec
GRANT ALL PRIVILEGES ON DATABASE $db TO $POSTGRES_USER;
EOSQL

# PASO 2: Inyectar extensiones en la DB recién creada
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" -d "$db" <<EOSQL
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS pg_trgm;
EOSQL
```
Esta arquitectura SQL previene corrupciones de inicialización, una de las causas más comunes de *downtime* en despliegues automatizados.

---

## 🧠 3. Gobierno Agéntico y Framework Cognitivo (L3)

El paradigma más valioso del ecosistema es la evolución de un bot de "Lectura" a un **Co-Worker Ejecutor**. 

### Políticas de Ejecución y RBAC Inherente
El agente n8n no posee acceso de superusuario a la base de datos. En su lugar, el frontend de Next.js (`/api/bot/chat`) intercepta la sesión JWT del usuario humano, extrae el `userId` y `role`, y los envía al webhook de n8n. 

Cuando el Agente decide usar una herramienta (ej. `crear_tarea`), hace un HTTP Request de vuelta a `/api/bot/actions/tasks` pasando esos mismos identificadores.
**Seguridad:** Las políticas de acceso, validación de Zod, y registro de auditoría (`activity_logs`) de la app principal evalúan al bot *exactamente igual* que si el usuario hubiera hecho clic en la interfaz. Un usuario con rol `RESIDENT` no puede crear registros administrativos mediante *Prompt Injection*.

### WAF Semántico Multimodal (Gateway Router)
El flujo `01_CasadoProp_Router_Gateway` actúa como el primer vector de defensa y normalización.

1. **Validación Criptográfica:** El webhook rechaza payloads sin el header `x-bot-token` correcto (Token SHA-256).
2. **Procesamiento de Audio On-The-Fly:** Si el payload incluye audio (`hasAudio == true`), un nodo de código extrae el buffer binario y lanza un request directo a la API de Gemini para transcribir el audio sin intermediarios, optimizando la latencia.
   ```javascript
   // Extracción nativa de audio en memoria (n8n)
   const buffer = await this.helpers.getBinaryDataBuffer(0, binaryKey);
   const base64Audio = buffer.toString('base64');
   // ... payload construction para Gemini
   ```
3. **Clasificador de Intenciones (LLM Firewall):** Utilizando Gemini 2.5 Flash (`temperature: 0.1`) forzado a responder en JSON estructurado, se clasifica el prompt sanitizado en tres únicas vías de ejecución:
   - `APP`: Intentos legítimos de operación. Pasa al Executor.
   - `MALICIOSO`: Intentos de inyección de prompt o exfiltración. Descartado con HTTP 200 silencioso.
   - `INCIERTO`: Peticiones ambiguas. Retorna solicitud de clarificación.

### El Agente Ejecutor (ReAct Paradigm)
El flujo `02_CasadoProp_Agent_Executor` alberga el LangChain Agent. Su *System Prompt* es estricto y define su comportamiento:

```text
Eres el Co-Worker de Casado Prop Gestión...
CONTEXTO DEL USUARIO ACTUAL:
- Nombre: {{ $json.body.userName }}
- Rol: {{ $json.body.userRole }}

RESTRICCIONES:
- NO inventar datos. Si no tenés la información, indicalo explícitamente.
- NO ejecutar acciones destructivas sin confirmación previa.
- Responder SIEMPRE en español argentino.
```
El nodo utiliza **Postgres Chat Memory** conectándose directamente a la tabla `chat_messages` (donde los mensajes se almacenan como `JSONB`). El agente tiene a su disposición sub-workflows (Tools) como `Consultar Propiedades`, los cuales abstraen la complejidad REST hacia la API de Next.js.

---

## 🚨 4. Manejo Global de Errores (InfoSec & Observabilidad)

El diseño abraza el principio *Fail-Safe*. El flujo `06_GLOBAL_Error_Handler` captura asíncronamente cualquier colapso (timeout, error de API, fallo de DB) en los workflows.

1. **Persistencia SQL:** Loguea el fallo en `bot_error_logs` (PostgreSQL) guardando el `execution_id`, `node_name` y el stacktrace completo (`error_details` JSONB).
2. **Alertas Out-Of-Band (OOB):** Dispara una alerta crítica vía Gmail OAuth2 al administrador de sistemas, evitando exponer detalles técnicos al usuario final.

```json
{
  "subject": "=ALERTA CRÍTICA Fallo en {{ $('Error Trigger').item.json.workflow.name }}",
  "message": "=Hubo un error en la ejecución del BOT... Mensaje:\"{{ $('Error Trigger').item.json.execution.error.message }}\""
}
```

---

## 🎯 Síntesis de Valor Profesional (Impacto CV)
La construcción de este artefacto demuestra **Seniority** en la intersección de **Desarrollo Full-Stack, Ingeniería de Datos y Seguridad (DevSecOps)**:
- **Zero-Trust y RBAC Inherente:** El bot no evade las políticas de la aplicación, las respeta por diseño arquitectónico.
- **Cognición Segura:** Creación de WAFs semánticos y procesado multimodal en milisegundos.
- **Idempotencia de Infraestructura:** Manejo profundo de Linux (UID/GID namespaces) y PostgreSQL avanzado (`\gexec`, `vector`).
- **Product Management:** Cambio de paradigma técnico, pasando de herramientas conversacionales a "Agentes Ejecutores" operacionales de alto valor para negocios B2B.
