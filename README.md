# LightRAG

Lokale LightRAG-Instanz mit PostgreSQL-Backend, betrieben via Docker Compose.

LightRAG ist ein Graph-basiertes RAG-System (Retrieval-Augmented Generation), das Wissensgraphen mit Vektorsuche kombiniert.

## Voraussetzungen

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Azure OpenAI Ressource (oder OpenAI / Ollama als Alternative)

## Einrichtung

1. **`.env` erstellen**

   ```powershell
   cp .env.example .env
   ```

2. **`.env` befüllen** — folgende Werte müssen gesetzt werden:

   | Variable | Beschreibung |
   |----------|-------------|
   | `LIGHTRAG_API_KEY` | Bearer-Token für API-Zugriff |
   | `AUTH_ACCOUNTS` | WebUI-Login: `benutzername:passwort` |
   | `TOKEN_SECRET` | JWT-Signierungsschlüssel (mind. 32 Zeichen, zufällig) |
   | `LLM_BINDING_HOST` | Azure OpenAI Endpoint-URL |
   | `LLM_BINDING_API_KEY` | Azure OpenAI API-Key |
   | `LLM_MODEL` | Deployment-Name des LLM |
   | `EMBEDDING_BINDING_HOST` | Azure OpenAI Endpoint-URL (Embedding) |
   | `EMBEDDING_BINDING_API_KEY` | Azure OpenAI API-Key (Embedding) |
   | `EMBEDDING_MODEL` | Deployment-Name des Embedding-Modells |
   | `POSTGRES_USER` | PostgreSQL Benutzername |
   | `POSTGRES_PASSWORD` | PostgreSQL Passwort |

3. **Container starten**

   ```powershell
   docker compose up -d
   ```

4. **WebUI öffnen:** [http://localhost:9621](http://localhost:9621)

## Architektur

```
docker-compose.yml
├── lightrag          # LightRAG Server + WebUI  (Port 9621)
│   └── ghcr.io/hkuds/lightrag:latest
└── postgres          # PostgreSQL mit pgvector + Apache AGE
    └── apache/age:latest + pgvector (inline Dockerfile)
```

### Datenpersistenz

| Pfad (Host) | Pfad (Container) | Inhalt |
|-------------|------------------|--------|
| `./data/rag_storage` | `/app/data/rag_storage` | RAG-Index, Graphdaten |
| `./data/inputs` | `/app/data/inputs` | Eingabedokumente |
| `./data/postgres` | `/var/lib/postgresql` | Datenbankdaten |

### Datenbank-Extensions

Die PostgreSQL-Instanz wird mit zwei Extensions initialisiert (`init-extensions.sql`):

- **pgvector** — Vektorsuche für Embeddings
- **Apache AGE** — Graph-Datenbank für den Wissensgraphen

## Konfiguration

### LLM-Anbieter wechseln

In der `.env` den `LLM_BINDING`-Wert anpassen:

```env
# Azure OpenAI (Standard)
LLM_BINDING=azure_openai

# OpenAI
LLM_BINDING=openai

# Ollama (lokal)
LLM_BINDING=ollama
LLM_BINDING_HOST=http://host.docker.internal:11434
```

### Authentifizierung

Die WebUI ist durch Login geschützt. Der `TOKEN_SECRET` wird für die JWT-Signierung benötigt und muss ein langer, zufälliger String sein.

```powershell
# Beispiel: zufälligen Token generieren
-join ((65..90) + (97..122) + (48..57) | Get-Random -Count 48 | ForEach-Object { [char]$_ })
```

## Nützliche Befehle

```powershell
# Starten
docker compose up -d

# Stoppen
docker compose down

# Logs anzeigen
docker compose logs -f lightrag

# Neu starten (nach .env-Änderungen)
docker compose down && docker compose up -d
```
