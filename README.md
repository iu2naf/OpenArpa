# Guida al Deploy in Produzione — IU2NAF Meteo Live Lombardia (OpenArpa)

Questo pacchetto contiene tutto il necessario per eseguire l'applicazione in ambiente di produzione **senza codice sorgente o compilatori**, utilizzando esclusivamente i container Docker pre-costruiti.

---

## Struttura del Pacchetto di Produzione

```
OpenArpa/
├── docker-compose.yml   # Definizione dello stack di microservizi (PostGIS, Backend, Frontend, Nginx)
├── nginx.conf           # Configurazione Reverse Proxy ottimizzata
├── .env.example         # Modello per le variabili d'ambiente
└── README.md            # Questa guida
```

---

## Prerequisiti

* **Docker Engine** (v24.0+) & **Docker Compose** (v2.20+) installati sull'host di produzione / LXC / VM.
* Le immagini Docker del backend e frontend caricate nel registry locale o sul server (`openarpa-backend:latest` e `openarpa-frontend:latest`).

---

## Istruzioni di Deploy Rapido (3 Passaggi)

### 1. Clona i file sul server
Clona l'intera cartella `OpenArpa/` nella directory di destinazione sul server (es. `/opt/openarpa` o `~/openarpa`).

### 2. Configura le variabili d'ambiente
Crea il file `.env` a partire dal modello:
```bash
cp .env.example .env
```
Modifica `.env` con un editor di testo (`nano .env`):
* Imposta una **password sicura** per PostgreSQL (`POSTGRES_PASSWORD`).
* Se le immagini provengono da un registry remoto (es. Gitea Packages), specifica i tag completi:
  ```env
  BACKEND_IMAGE=gitea.tuoserver.local/utente/openarpa-backend:latest
  FRONTEND_IMAGE=gitea.tuoserver.local/utente/openarpa-frontend:latest
  ```
* Configura la porta host desiderata (`HOST_PORT=8085`).
* (Opzionale) Inserisci token e chat ID per il bot Telegram di allerta meteo.

### 3. Avvia l'applicazione
```bash
docker compose pull   # Se utilizzi un remote registry
docker compose up -d
```

L'applicazione sarà immediatamente raggiungibile su:
👉 `http://<IP_SERVER>:8085`

Documentazione API Swagger:
👉 `http://<IP_SERVER>:8085/docs`

---

## Comandi Utili per la Gestione

### Verificare lo stato dei container
```bash
docker compose ps
```

### Visualizzare i log in tempo reale
```bash
docker compose logs -f
# Oppure per un singolo servizio:
docker compose logs -f backend
```

### Riavviare lo stack
```bash
docker compose restart
```

### Aggiornare a una nuova versione
```bash
docker compose pull
docker compose up -d --remove-orphans
```

### Backup del Database PostGIS
```bash
docker compose exec -T postgres pg_dump -U postgres openarpa > backup_openarpa_$(date +%Y%m%d_%H%M%S).sql
```

### Ripristino del Database da Backup
```bash
cat backup_openarpa_*.sql | docker compose exec -T postgres psql -U postgres openarpa
```
