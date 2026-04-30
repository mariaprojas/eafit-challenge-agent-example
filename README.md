# Asistente de Pre-Auditoría (Colombia) - EAFIT Challenge

Un agente de IA especializado en pre-auditoría, cumplimiento normativo y gestión de riesgos en Colombia, desplegado como un Servicio Verificable (Verifiable Service) en el ecosistema [Verana](https://verana.io). Este proyecto es parte del [EAFIT Challenge](https://github.com/verana-labs/eafit-challenge).

## Arquitectura

Este agente está construido con el contenedor [hologram-generic-ai-agent-vs](https://github.com/2060-io/hologram-generic-ai-agent-vs), que proporciona un chatbot de IA listo para usar con mensajería DIDComm, autenticación de credenciales verificables, capacidades RAG (Retrieval-Augmented Generation) y herramientas MCP.

El agente funciona como un **servicio hijo** de la organización del EAFIT Challenge. Sus funciones principales son:

1. Recibir una **Credencial de Servicio** de la organización (prueba de que es un servicio legítimo).
2. Utilizar la **Definición de Credencial (CredDef) de Avatar** desde `avatar.eafit.testnet.verana.network` para autenticar a los usuarios mediante solicitudes de prueba de AnonCreds.
3. Proveer capacidades de análisis RAG sobre normativas locales e ISOs, y herramientas GitHub MCP a usuarios autenticados vía chat DIDComm cifrado.

## Características Principales

- **Especialista en Auditoría**: Configurado para aplicar normativas locales (Colombia) y estándares internacionales (ISO 9001, ISO 14001, ISO 19011).
- **RAG (Retrieval-Augmented Generation)**: Capacidad de leer y analizar documentos normativos, manuales de procedimiento e índices maestros proporcionados en el directorio `/docs`.
- **LLM**: Utiliza modelos de Anthropic (Claude) por defecto para el razonamiento y generación de respuestas, e integra modelos de Voyage AI para los embeddings del sistema RAG.

## Estructura del Repositorio

```
├── config.env              # Configuración del servicio (puertos, credenciales, LLMs, etc.)
├── deployment.yaml         # Valores del chart de Helm para despliegue en K8s
├── agent-pack.yaml         # Configuración del chatbot (prompts, RAG, flujos, MCP)
├── common/
│   └── common.sh           # Scripts auxiliares compartidos
├── docker/
│   ├── docker-compose.yml  # Stack de desarrollo local
│   └── .env                # Variables de entorno para Docker Compose
├── scripts/
│   ├── setup.sh            # Configuración local (Agente VS + ngrok + Credencial de Servicio)
│   └── start.sh            # Iniciar el stack de Docker Compose
├── docs/                   # Documentación base para el sistema RAG (Normas ISO, Manuales)
└── .github/
    └── workflows/
        └── deploy.yml      # Flujo de GitHub Actions para despliegue en K8s
```

## Desarrollo Local

### Requisitos Previos

- Docker y Docker Compose
- [ngrok](https://ngrok.com/) (autenticado)
- `curl`, `jq`
- API Keys correspondientes: `ANTHROPIC_API_KEY` (para el LLM) y `OPENAI_API_KEY` (para embeddings de Voyage AI o similar, según configuración).

El script de configuración se conecta a la organización EAFIT desplegada en `admin.organization.eafit.testnet.verana.network` para obtener la Credencial de Servicio.

### Inicio Rápido

```bash
# 1. Configurar el Agente VS (despliega el contenedor, obtiene la Credencial de Servicio)
source config.env
./scripts/setup.sh

# 2. Configurar variables de entorno y levantar el stack completo (chatbot + redis + postgres)
export NGROK_DOMAIN=<tu-dominio-ngrok>
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=pa-... # Utilizado en este caso para la API compatible de Voyage AI
./scripts/start.sh
```

## Despliegue en Kubernetes (GitHub Actions)

El flujo `.github/workflows/deploy.yml` despliega el agente en el clúster K8s compartido del EAFIT Challenge.

### Secretos de GitHub Requeridos

| Secreto | Descripción |
| ------ | ----------- |
| `OVH_KUBECONFIG` | Kubeconfig para el clúster K8s |
| `K8S_NAMESPACE` | Espacio de nombres objetivo (nombre de tu equipo) |
| `EXAMPLE_AGENT_ANTHROPIC_API_KEY` | API Key de Anthropic para el chatbot |
| `EXAMPLE_AGENT_OPENAI_API_KEY` | API Key para embeddings (Voyage AI/OpenAI) |
| `EXAMPLE_AGENT_POSTGRES_PASSWORD` | Contraseña de PostgreSQL |
| `EXAMPLE_AGENT_MCP_CONFIG_ENCRYPTION_KEY` | Llave de cifrado para configs de usuarios MCP (generar con `openssl rand -hex 32`) |
| `EXAMPLE_AGENT_WALLET_KEY` | Llave de cifrado de la billetera del Agente VS (generar con `openssl rand -base64 32`) |
| `EXAMPLE_AGENT_VSAGENT_DB_PASSWORD` | Contraseña de la BD interna del Agente VS |

### Despliegue

Ejecuta el flujo desde la pestaña de GitHub Actions con el paso `all` para desplegar y obtener credenciales.

El agente estará disponible en la URL configurada en `AGENT_PUBLIC_URL` (ver abajo).

## Configuración

Ajustes clave en `config.env`:

- **`AGENT_PUBLIC_URL`** — URL pública del agente desplegado. Convención: `https://<agentname>.agents.<team_name>.teams.eafit.testnet.verana.network`
- **`CREDENTIAL_DEFINITION_ID`** — CredDef de AnonCreds del servicio Avatar de EAFIT (fijado).
- **`ORG_VS_PUBLIC_URL`** — URL pública del agente de la organización EAFIT.
- **`SERVICE_NAME`** — Nombre a mostrar en la Credencial de Servicio (ej. "Asistente de Pre-Auditoría Colombia").
- **`LLM_PROVIDER`** y **`ANTHROPIC_MODEL`** — Proveedor y modelo a usar para el chatbot.

## Licencia

Apache-2.0
