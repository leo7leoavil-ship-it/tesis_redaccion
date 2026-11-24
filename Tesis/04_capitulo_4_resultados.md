# CAPÍTULO IV. RESULTADOS

> **Estado del documento:**  
> - **Sección `4.1 Implementación del sistema`:** Redactada al 100 % con base en la especificación real de `tech-tutor-engine` v1.0.0 (`Documentos_tecnicos/`).  
> - **Sección `4.2 Resultados de pruebas`:** Incluye los resultados verificados de la suite automatizada (`72` pruebas en `pytest`) y **tablas estructuradas con marcadores `[PENDIENTE DE MEDICIÓN REAL]`** listas para vaciar los resultados de `tests/benchmark_thesis.py` y de las pruebas piloto con usuarios en los endpoints.

---

## 4.1 Implementación del sistema (`tech-tutor-engine` v1.0.0)

Se implementó exitosamente el motor de inferencia y evaluación técnica **`tech-tutor-engine` v1.0.0** bajo una arquitectura desacoplada *Model-as-a-Service* (MaaS) construida sobre `FastAPI` y `Uvicorn` (`app/main.py`). La solución organiza la lógica de negocio en capas separadas de configuración y seguridad (`app/core/`), contratos de datos (`app/schemas/`), enrutamiento v1 (`app/api/v1/`) y servicios de inteligencia artificial (`app/services/`).

### 4.1.1 Orquestación del flujo multimodal asíncrono

El orquestador central (`app/services/evaluation_pipeline.py`) coordina la ejecución secuencial y asíncrona de los tres subsistemas de IA sin bloquear el *event loop* de FastAPI, delegando las operaciones intensivas de CPU (`ASR` y `RAG`) mediante `asyncio.to_thread` y aplicando tiempos límite explícitos por etapa: **`90.0 s`** para ASR, **`15.0 s`** para recuperación RAG y **`30.0 s`** para inferencia LLM (`HTTP 504 Gateway Timeout` ante excedencia). Durante el arranque (*lifespan* en `app/main.py`), el sistema precarga los modelos en memoria y verifica la disponibilidad de la base vectorial mediante `ensure_corpus_available()`.

### 4.1.2 Subsistema de Reconocimiento Automático de Voz (ASR)

Implementado en `app/services/asr_service.py` bajo un patrón *Singleton* con importación perezosa de `faster_whisper`:

* **Modelo y decodificación en memoria:** Utiliza `faster-whisper` (`CTranslate2`) con pesos `large-v3-turbo` en CPU cuantizado a `int8` (`cpu_threads=4`, `num_workers=1`) y decodificación *greedy* (`ASR_BEAM_SIZE = 1`), optimizada para reducir la latencia en CPU respecto a búsquedas por haz más amplias. Todo el procesamiento ocurre en memoria mediante `io.BytesIO` (aplicando el parche `_patch_pyav_metadata_errors()` para compatibilidad con `PyAV 19+`) o mediante conversión directa cero-copia `pcm16_to_float32()` para tramas WebSocket `pcm16` a `16 kHz`.
* **Sesgo léxico y filtrado:** Activa `vad_filter=True` (`min_silence_duration_ms=500`) junto con `ASR_INITIAL_PROMPT` cargado con vocabulario técnico de ingeniería (*PyTorch, Kubernetes, Docker, FastAPI, horizontal sharding, microservices, latency, system design*).
* **Protección de recursos y cancelación cooperativa:** Valida límites estrictos de **`5 MB`** (`5,242,880` bytes, rechazo `HTTP 413`) y **`60.0 s`** (`AudioDurationExceededError` → `HTTP 422`) antes de iterar los segmentos de CTranslate2. Controla la concurrencia en CPU con `asyncio.Semaphore(ASR_MAX_CONCURRENCY=1)` y espera máxima de `10.0 s` (`HTTP 429`), e implementa cancelación cooperativa mediante `threading.Event` (`cancel_event`) ante desconexiones o *timeouts*.
* **Métricas acústicas (`ASRMetrics`):** Reporta `transcription`, `detected_language`, `speech_duration_seconds`, `confidence_score`, `word_count`, `words_per_minute`, `avg_logprob`, `no_speech_prob` y, cuando se incluye `expected_transcript`, `wer_score` y `cer_score` calculados con `jiwer`.

### 4.1.3 Subsistema de Recuperación Semántica Aumentada (RAG)

Implementado en `app/services/rag_service.py` y aprovisionado por `app/services/corpus_sync.py`:

* **Base vectorial y embeddings:** Opera sobre `ChromaDB` (`PersistentClient` en `data/technical_corpus_db/chroma.sqlite3`, `~1.08 GB` en disco) con la colección `tech_tutor_corpus`, integrada por **`213,439` fragmentos técnicos** indexados en espacio de distancia coseno (`hnsw:space: cosine`) mediante `SentenceTransformer` (`all-MiniLM-L6-v2`, `384` dimensiones).
* **Fusión semántica y sanitización:** Construye la consulta mediante `build_search_query()` con el formato `"{escenario_efectivo}: {texto}"` (empleando directamente el texto validado de `custom_scenario` cuando el escenario es personalizado) y limpia etiquetas residuales (`@article@`, `@video@`, `@feed@`) mediante `sanitize_chunk_text()` para optimizar el contexto enviado al LLM.

### 4.1.4 Núcleo evaluativo (`LLM-as-a-Judge`) y determinismo léxico

Implementado en `app/services/llm_service.py` sobre el cliente asíncrono `AsyncOpenAI` conectado a **Groq Cloud** (`https://api.groq.com/openai/v1`):

* **Inferencia estructurada:** Emplea el modelo `openai/gpt-oss-120b` con `temperature = 0.1`, `reasoning_effort = "low"` y `LLM_STRUCTURED_OUTPUT = true` (esquema JSON estricto con degradación automática a `json_object`), evaluando al candidato bajo rúbricas **MCER / CEFR (`A2`–`C2`)**. Emite `overall_score`, `grammar_accuracy_score`, `technical_precision_score`, `detected_technical_terms` y hasta 6 elementos en `feedback_details` (`vocabulary_enhancement`, `grammar_correction`, `architectural_alignment`) anclados al contexto RAG.
* **Cálculo determinista en Python:** Para evitar alucinaciones aritméticas, el LLM extrae únicamente la lista literal `detected_technical_terms`, mientras que `compute_lexical_density()` calcula la densidad léxica de forma determinista:
  $$\text{lexical\_density\_ratio} = \min\left(\frac{|\text{unique\_terms}|}{\max(|\{w \in \text{tokens} : |w| > 1\}|, 1)},\; 1.0\right)$$
* **Política estricta de fallos:** Ante errores de red, *timeouts* (`504`) o respuestas inválidas (`LLMServiceError` → `502`), el sistema nunca fabrica puntajes simulados.

### 4.1.5 Contratos de interfaz, seguridad y consola demostrativa

La especificación OpenAPI (`v1.0.0`) y los esquemas `Pydantic v2` (`app/schemas/`) exponen los siguientes puntos de acceso protegidos mediante autenticación *fail-closed* (`hmac.compare_digest` contra `API_AUTH_TOKEN`):

| Método | Ruta | Autenticación | Propósito |
| :--- | :--- | :--- | :--- |
| `GET` | `/` · `/docs` · `/redoc` · `/openapi.json` | Pública | Descubrimiento y documentación API interactiva |
| `GET` | `/ui` | Pública (shell) | Consola web gráfica demostrativa (micrófono, archivo, WS y texto) |
| `GET` | `/health` · `/api/v1/health` | Token | Sonda de estado (`chroma_ready`, `asr_ready`, `llm_configured`) |
| `POST` | `/api/v1/query` | Token | Búsqueda vectorial directa (`top_k` ∈ `[1, 20]`) |
| `POST` | `/api/v1/evaluate/text` | Token | Evaluación de texto (`RAG → LLM`) |
| `POST` | `/api/v1/evaluate/voice` | Token | Evaluación multimodal de audio (`ASR → RAG → LLM`) |
| `WS` | `/api/v1/ws/practice` | Token (`?token=`) | Streaming de audio *full-duplex* con eventos de etapa y cancelación `reset` |

Todos los endpoints de evaluación comparten el catálogo canónico `TargetScenario` (`9` escenarios predefinidos de ingeniería + `custom` validado de `3` a `120` caracteres).

### 4.1.6 Contenerización y rutas de despliegue

El sistema se empaquetó mediante un `Dockerfile` *multi-stage* CPU con `ffmpeg`, PyTorch CPU, pre-descarga de `large-v3-turbo` y `all-MiniLM-L6-v2` y ejecución como usuario no privilegiado (`uid 1000`, puerto `7860`), requiriendo en operación `3.5–4.5 GB` de RAM. Se validaron documentalmente dos rutas operativas: despliegue local con túnel HTTPS/WSS (`cloudflared`) para pruebas externas y demostración, y despliegue continuo 24/7 en contenedor Docker.

---

## 4.2 Resultados de pruebas

### 4.2.1 Verificación funcional y de contratos (`pytest`)

La ejecución de la suite automatizada (`pytest -v`) en entorno controlado reportó el cumplimiento íntegro de las **72 pruebas unitarias y de integración** en menos de `2 segundos` (utilizando dobles de prueba sin acceso a red ni carga de pesos), verificando:
* Rechazo *fail-closed* (`HTTP 401` y cierre WebSocket `1008`) ante tokens ausentes o inválidos.
* Validación estricta de los `9` escenarios predefinidos y el modo `custom` (rechazo `422` ante combinaciones inválidas).
* Corte temprano de audios `> 5 MB` (`413`) y `> 60 s` (`422`), saturación de cola ASR (`429`) y *timeouts* por etapa (`504`).
* Sanitización RAG, ordenamiento coseno y cálculo determinista de densidad léxica.

### 4.2.2 Resultados de *benchmark* controlado (`tests/benchmark_thesis.py`)

El `23 de noviembre de 2026` se ejecutó el subcomando `rag` de `tests/benchmark_thesis.py` (`python tests/benchmark_thesis.py rag --queries benchmark_data/queries.json --ks 3 5`) sobre la colección productiva `tech_tutor_corpus` (`213,439` vectores en `ChromaDB`), evaluando `20` consultas técnicas representativas de ingeniería de software, infraestructura, bases de datos, sistemas distribuidos y entrevistas técnicas. Los subcomandos `wer` y `latency` (extremo a extremo con audio) permanecen pendientes de ejecución al requerir un conjunto de archivos de audio de referencia.

**Tabla 4.1. Resultados de exactitud fonética ASR (`benchmark_thesis.py wer`)**

| Muestra / Condición | Audios evaluados ($N$) | WER promedio (%) | CER promedio (%) | Meta tesis | Estado |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Audios técnicos controlados | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | $< 12\%$ | `[PENDIENTE]` |

**Tabla 4.2. Calidad de recuperación semántica RAG (`benchmark_thesis.py rag`, $N = 20$ consultas)**

| Métrica | Valor obtenido | Objetivo tesis | Estado |
| :--- | :---: | :---: | :---: |
| **Consultas evaluadas** | `20` | — | — |
| **`Recall@3`** | **`90.00%` (`0.90`)** | $\ge 85\%$ | **APROBADO (PASS)** |
| **`Recall@5`** | **`90.00%` (`0.90`)** | Referencia | — |
| **`NDCG@3`** | **`0.7917`** | — | — |
| **`NDCG@5`** | **`0.8316`** | — | — |

El subsistema RAG superó la meta establecida ($\text{Recall@3} \ge 85\%$) alcanzando **`90.00%` (`18/20` aciertos en Top-3)** y un ordenamiento sólido (`NDCG@5 = 0.8316`). Adicionalmente, en las únicas dos consultas marcadas como no coincidentes por el criterio estricto de repositorio (`system-design-primer`, consultas 1 y 2 sobre *sharding* y *teorema CAP*), la inspección de los fragmentos recuperados confirmó que el motor devolvió documentación temáticamente equivalente proveniente de `developer-roadmap` (`system-design/content/sharding` y `cap-theorem`), `mongo` (`README_sharding_catalog.md`) y `coding-interview-university`, evidenciando pertinencia semántica efectiva en la totalidad de las consultas.

**Tabla 4.3. Perfilado de latencia por etapa (`benchmark_thesis.py`)**

| Etapa del pipeline | Media (s / ms) | Mediana P50 | P95 | Mín / Máx | Perfil de hardware |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **RAG (`search_latency_ms`, $N=20$)** | **`146.59 ms`** | **`140.18 ms`** | **`162.62 ms`** | `123.86` / `284.90 ms` | CPU local (`ChromaDB` + `MiniLM`) |
| ASR (`faster-whisper int8`) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[CPU / RAM]` |
| LLM (`Groq gpt-oss-120b`) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | API Cloud |
| **Total extremo a extremo** | **`[PENDIENTE]`** | **`[PENDIENTE]`** | **`[PENDIENTE]`** | **`[PENDIENTE]`** | — |

### 4.2.3 Resultados del estudio piloto en endpoints con usuarios reales

`[PENDIENTE DE MEDICIÓN REAL: Aplicar el protocolo piloto (N = 10–15 participantes) contra POST /api/v1/evaluate/voice y POST /api/v1/evaluate/text y vaciar las métricas agregadas en la Tabla 4.4.]`

**Tabla 4.4. Métricas agregadas de interacciones reales en endpoints (estudio piloto)**

| Indicador del motor | $N$ interacciones | Media | Mínimo | Máximo | Desv. estándar |
| :--- | :---: | :---: | :---: | :---: | :---: |
| `wer_score` (lectura guiada) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |
| `confidence_score` (ASR) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |
| `words_per_minute` (WPM) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |
| `overall_score` (0–100) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |
| `grammar_accuracy_score` (0–100) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |
| `technical_precision_score` (0–100) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |
| `lexical_density_ratio` (0–1) | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` | `[PENDIENTE]` |

---

## 4.3 Evaluación del cumplimiento de objetivos y contrastación de hipótesis

`[PENDIENTE DE CIERRE TRAS MEDICIÓN REAL: Una vez completadas las Tablas 4.1–4.4, redactar la contrastación final de cada objetivo específico (OE1–OE4) y la validación cuantitativa de la hipótesis (WER < 12 % y Recall@3 ≥ 85 %).]`

De manera preliminar, en el plano de ingeniería de software, los cuatro objetivos específicos cuentan con **implementación operativa completa** y **verificación funcional aprobada (`72/72` pruebas en `pytest`)**, quedando pendiente únicamente el registro de las mediciones empíricas finales sobre carga real.
