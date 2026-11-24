# Guía de Despliegue a Nivel Productivo · Tech Tutor Engine

Esta guía describe cómo llevar **Tech Tutor Engine** (`FastAPI + faster-whisper int8 + ChromaDB 213k + Groq gpt-oss-120b`) desde un entorno local hasta una operación estable a nivel productivo, cubriendo dos rutas según el presupuesto y la etapa del proyecto:

- **Ruta A · Desarrollo, Pruebas Externas y Defensa de Tesis (Costo $0):** Servidor local + **Cloudflare Tunnel (`cloudflared`)** con HTTPS/WSS y subdominio temporal o fijo.
- **Ruta B · Producción Continua 24/7 (Contenedor Docker):** Despliegue del `Dockerfile` en un VPS con **4–8 GB de RAM** (Oracle Cloud Free Tier, DigitalOcean/Azure con créditos estudiantiles o Hugging Face Spaces Docker).

---

## 1. Requisitos de Recursos (Por qué no sirven planes gratuitos de 512 MB)

En ejecución con todos los modelos precargados en memoria, el servicio consume aproximadamente **3.5 – 4.5 GB de RAM** y **~4.5 GB de disco**:

| Componente | Consumo Estimado en RAM | Notas Operativas |
|---|---|---|
| `faster-whisper` (`large-v3-turbo`, CPU `int8`, `beam_size=1`) | 1.5 – 2.5 GB | Un solo worker en memoria (`ASR_MAX_CONCURRENCY=1`) |
| `SentenceTransformer` (`all-MiniLM-L6-v2` + PyTorch CPU) | 0.5 – 1.0 GB | Embeddings de 384 dimensiones |
| `ChromaDB` (`tech_tutor_corpus`, 213,439 vectores) | 0.8 – 1.5 GB | `~1.08 GB` en disco (`chroma.sqlite3` + índice HNSW) |
| FastAPI + Uvicorn + Buffers `BytesIO` (máx. 5 MB/petición) | 0.3 – 0.5 GB | Procesamiento de audio 100 % en memoria |

> **Recomendación mínima para producción 24/7:** 4 vCPU y 8 GB de RAM (mínimo estricto: 2 vCPU y 4 GB de RAM con swap habilitado).

---

## 2. Fase Previa · Seguridad de Credenciales y Validación Local

1. **Rotar y generar un `API_AUTH_TOKEN` fuerte** (no reutilizar tokens de pruebas anteriores):
   ```powershell
   python -c "import secrets; print(secrets.token_urlsafe(32))"
   ```
2. **Verificar `.env`:** asegúrese de que `.env` **nunca** esté versionado en Git (`git ls-files .env` no debe devolver nada) y que contenga:
   ```ini
   GROQ_API_KEY=<tu_clave_activa_de_groq>
   API_AUTH_TOKEN=<token_generado_en_el_paso_1>
   ENVIRONMENT=production
   ALLOWED_ORIGINS=["*"]
   MAX_AUDIO_BYTES=5242880
   MAX_AUDIO_SECONDS=60.0
   ASR_BEAM_SIZE=1
   ASR_MAX_CONCURRENCY=1
   ASR_QUEUE_TIMEOUT_SECONDS=10.0
   ASR_TIMEOUT_SECONDS=90.0
   RAG_TIMEOUT_SECONDS=15.0
   LLM_TIMEOUT_SECONDS=30.0
   ```
3. **Ejecutar la suite de pruebas local** (72 pruebas, < 2 s, sin red ni descarga de modelos):
   ```powershell
   pytest -v
   ```
4. **Verificar el corpus local (`213,439` vectores):**
   ```powershell
   python -c "import chromadb; c=chromadb.PersistentClient(path='data/technical_corpus_db').get_collection('tech_tutor_corpus'); print(c.count())"
   ```

---

## 3. Ruta A · Despliegue Rápido con Cloudflare Tunnel (Costo $0)

Ideal para desarrollo, integración con clientes externos (Postman/Frontend) y demostración en vivo durante la defensa de tesis aprovechando el CPU y la RAM de tu equipo sin tiempos de arranque en frío descargando 1 GB de base vectorial.

### A.1. Instalar `cloudflared` (una sola vez en Windows)
```powershell
winget install --id Cloudflare.cloudflared --accept-package-agreements --accept-source-agreements
```

### A.2. Arrancar el servidor FastAPI (Terminal 1)
```powershell
python -u -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```
Espere hasta ver `All AI pipelines preloaded in memory. System ready for inference.` y `Uvicorn running on http://127.0.0.1:8000`.

### A.3. Abrir el túnel HTTPS/WSS (Terminal 2)
- **Túnel rápido temporal (sin cuenta):**
  ```powershell
  & "C:\Program Files (x86)\cloudflared\cloudflared.exe" tunnel --url http://127.0.0.1:8000
  ```
  Cloudflare imprimirá una URL pública `https://<subdominio>.trycloudflare.com` que soporta tanto REST (`https://`) como WebSockets (`wss://`).
- **Túnel persistente con dominio propio (opcional, gratuito con cuenta Cloudflare Zero Trust):**
  Cree un *Named Tunnel* en el panel de Cloudflare Zero Trust apuntando al servicio `http://127.0.0.1:8000` para conservar siempre la misma URL entre reinicios.

---

## 4. Ruta B · Producción Continua 24/7 con Docker (VPS o Cloud)

### B.1. Preparar la base vectorial (`data/technical_corpus_db/`)
La carpeta `data/technical_corpus_db/` (~1.08 GB) no se sube a Git. Para producción tiene dos estrategias:
1. **Volumen persistente en VPS (Recomendada):** Copie `data/technical_corpus_db/` una sola vez al servidor (`scp`/`rsync`) y móntela como volumen en Docker (`-v /srv/tech_tutor/technical_corpus_db:/app/data/technical_corpus_db`). Arranca en segundos sin descargar nada.
2. **Descarga automática desde Hugging Face Datasets (para contenedores efímeros):**
   ```powershell
   hf auth login
   hf repo create <TU_USUARIO_HF>/tech-tutor-corpus --repo-type dataset --private
   hf upload <TU_USUARIO_HF>/tech-tutor-corpus "data/technical_corpus_db" . --repo-type dataset
   ```
   Configure `CORPUS_AUTO_DOWNLOAD=true`, `CORPUS_DATASET_REPO=<TU_USUARIO_HF>/tech-tutor-corpus` y `HF_TOKEN=<token_lectura>` en el contenedor.

### B.2. Construir y ejecutar el contenedor Docker
El `Dockerfile` multi-stage incluye `ffmpeg`, PyTorch CPU, pre-descarga de `large-v3-turbo` y `all-MiniLM-L6-v2` (Zero Cold-Start) y ejecuta Uvicorn como usuario no privilegiado (`uid 1000`) en el puerto `7860`:

```bash
docker build -t tech-tutor-engine:1.0.0 .

docker run -d \
  --name tech-tutor-engine \
  --restart unless-stopped \
  -p 7860:7860 \
  --env-file .env \
  -e PORT=7860 \
  -v /srv/tech_tutor/technical_corpus_db:/app/data/technical_corpus_db \
  tech-tutor-engine:1.0.0
```

> **Importante:** Mantenga **un único worker de Uvicorn** (`CMD ["uvicorn", "app.main:app", ...]`). Los modelos de Whisper y MiniLM residen en memoria y se comparten de forma segura mediante el semáforo `ASR_MAX_CONCURRENCY=1`.

### B.3. Proxy inverso HTTPS en VPS (Nginx / Caddy)
Si despliega en un VPS Linux con dominio propio, coloque **Caddy** o **Nginx** delante del puerto `7860` con:
- Certificado TLS automático (Let's Encrypt).
- Soporte de `Upgrade` / `Connection` para WebSockets en `/api/v1/ws/practice`.
- `proxy_read_timeout 120s;` y `client_max_body_size 6M;` (acorde al límite `MAX_AUDIO_BYTES=5MB` y `ASR_TIMEOUT_SECONDS=90s`).

---

## 5. Verificación Post-Despliegue

Sustituya `<URL>` por su dominio HTTPS y `<TOKEN>` por su `API_AUTH_TOKEN`:

| # | Verificación | Ruta / Comando | Resultado Esperado |
|---|---|---|---|
| 1 | Portal de Documentación | Abrir `<URL>/docs` | Guía corporativa estilo Oracle con ejemplos Postman/cURL |
| 2 | Referencia OpenAPI | Abrir `<URL>/redoc` | Especificación ReDoc completa sin secciones vacías |
| 3 | Consola Interactiva | Abrir `<URL>/ui` | Consola responsiva (Claro/Oscuro) con verificador `/health` |
| 4 | Seguridad Fail-Closed | `curl -i <URL>/health` | `401 Unauthorized` |
| 5 | Salud de Subsistemas | `curl <URL>/health -H "Authorization: Bearer <TOKEN>"` | `"status":"ok"`, `chroma_ready`, `asr_ready`, `llm_configured` en `true` (`213439 vectors`) |
| 6 | Evaluación de Texto + Custom | `POST <URL>/api/v1/evaluate/text` con `target_scenario="custom"` y `custom_scenario` | `200 OK` con `cefr_level`, `lexical_density_ratio` y `feedback_details` |
| 7 | Evaluación de Voz | Grabar/subir audio en `/ui` o `POST <URL>/api/v1/evaluate/voice` | `200 OK` con `asr_metrics` y `evaluation_core` |
| 8 | Streaming WebSocket | Conectar en pestaña WebSocket de `/ui` (`wss://<host>/api/v1/ws/practice?token=<TOKEN>`) | Secuencia `ready → transcribing → retrieving → evaluating → completed` y prueba de `reset` |

---

## 6. Solución de Problemas Frecuentes

| Síntoma | Causa Probable | Solución |
|---|---|---|
| `401 Unauthorized` o WS cierra con `1008` | `API_AUTH_TOKEN` ausente, con espacios o distinto al de `.env` | Verifique la variable en el servidor y reinicie el proceso |
| `413 Content Too Large` | Audio supera **5 MB** (`MAX_AUDIO_BYTES`) | Use formato comprimido (`.webm`/`.mp3`) o grabe < 60 s |
| `422 Unprocessable Entity` | Audio > **60 s**, sin voz detectable o `custom_scenario` inválido/presente en escenario predefinido | Verifique duración y las reglas de `target_scenario` / `custom_scenario` |
| `429 Too Many Requests` | Otro audio ocupa el semáforo ASR en CPU por más de **10 s** | Espere unos segundos y reintente |
| `504 Gateway Timeout` | Una etapa superó su timeout (`90 s` ASR / `15 s` RAG / `30 s` LLM) | Revise carga de CPU o conectividad hacia `api.groq.com` |
| Colección ChromaDB con solo 5 documentos | No se encontró `chroma.sqlite3` en `data/technical_corpus_db/` | Monte el volumen real o verifique `HF_TOKEN` y `CORPUS_DATASET_REPO` |
| `502 Bad Gateway` en evaluación | `GROQ_API_KEY` inválida o límite de cuota alcanzado en Groq Cloud | Revise la consola de Groq (`console.groq.com`) |

---

## 7. Checklist de Salida a Producción

- [ ] `API_AUTH_TOKEN` único generado con `secrets.token_urlsafe(32)` y `.env` excluido de Git
- [ ] `pytest -v` pasando **72/72** pruebas en verde
- [ ] Base vectorial confirmada con **213,439 vectores** en `tech_tutor_corpus`
- [ ] `/health` devolviendo `"status": "ok"` con los tres subsistemas en `true`
- [ ] Pruebas manuales de voz (REST + WebSocket), texto, `custom_scenario` y RAG superadas en `/ui`
- [ ] Benchmarks de tesis (`tests/benchmark_thesis.py`) archivados para la defensa
