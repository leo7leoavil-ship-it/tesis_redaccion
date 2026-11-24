# CAPÍTULO III. METODOLOGÍA

> **Estado del documento:** Borrador completo alineado al pivote **Motor MaaS (`tech-tutor-engine` v1.0.0)**, a la validación por endpoints y al plan de pruebas en dos niveles (controladas + piloto con usuarios).

---

## 3.1 Enfoque y tipo de investigación

La presente investigación adopta un **enfoque cuantitativo** de tipo **aplicado y tecnológico**, orientado al diseño, construcción y validación empírica de un motor de inferencia y evaluación técnica desacoplado (*Model-as-a-Service*, MaaS). El carácter cuantitativo se fundamenta en la recolección y análisis objetivo de métricas numéricas generadas por el propio sistema y sus protocolos de evaluación: Tasa de Error de Palabra ($\text{WER}$), Tasa de Error de Caracteres ($\text{CER}$), precisión de recuperación semántica ($\text{Recall@k}$), latencia por etapa (ms/s), densidad léxica técnica y puntajes estructurados bajo rúbricas del Marco Común Europeo de Referencia para las Lenguas (MCER / CEFR).

En cuanto a su alcance, el estudio es de carácter **descriptivo-experimental en ingeniería de software e inteligencia artificial**: por un lado, describe y caracteriza el desempeño de un pipeline multimodal asíncrono (`ASR → RAG → LLM-as-a-Judge`) bajo restricciones reales de hardware (CPU `int8`); por otro, evalúa experimentalmente el comportamiento de los endpoints expuestos ante entradas controladas y ante muestras reales de habla y texto técnico producidas por usuarios del dominio objetivo.

---

## 3.2 Diseño metodológico

Para garantizar el rigor técnico y la validez científica sin depender del desarrollo de interfaces frontend complejas, el diseño metodológico se estructura en **dos fases complementarias**:

1. **Fase 1 — Desarrollo iterativo y validación técnica controlada (completada en su componente de ingeniería):** comprende la construcción modular del motor `tech-tutor-engine` v1.0.0, la verificación automatizada de contratos y lógica de negocio mediante pruebas unitarias e integración (`pytest`), y el protocolo reproducible de *benchmarking* (`tests/benchmark_thesis.py`) sobre conjuntos de prueba controlados.
2. **Fase 2 — Validación empírica mediante pruebas a endpoints con muestra piloto de usuarios (por ejecutar):** consiste en un **estudio piloto transversal** en el que participantes del perfil objetivo interactúan directamente con los endpoints REST expuestos del motor (`/api/v1/evaluate/voice` y `/api/v1/evaluate/text`). En esta fase, la consola web `/ui` cumple únicamente una función gráfica demostrativa para ilustrar el flujo de los microservicios a observadores no técnicos, mientras que la recolección formal de datos se realiza directamente a partir de las respuestas JSON estructuradas emitidas por la API.

### 3.2.1 Etapas de ingeniería del sistema

El desarrollo del motor siguió cinco etapas secuenciales:

1. **Ingeniería de datos y construcción del corpus vectorial (`Data Track`):** curación, limpieza de etiquetas residuales (`@article@`, `@video@`, `@feed@`) e indexación de `213,439` fragmentos de documentación técnica especializada en `ChromaDB` (`tech_tutor_corpus`), utilizando embeddings densos de `384` dimensiones (`all-MiniLM-L6-v2`) en espacio de distancia coseno (`hnsw:space: cosine`).
2. **Implementación del subsistema ASR en memoria:** integración de `faster-whisper` (`large-v3-turbo`, cuantización `int8`, `beam_size=1`) con decodificación estrictamente en memoria (`io.BytesIO` y ruta directa cero-copia `pcm16_to_float32`), filtro de actividad vocal (VAD), sesgo léxico técnico (`ASR_INITIAL_PROMPT`), límites tempranos (`5 MB` / `60 s`), semáforo de concurrencia (`ASR_MAX_CONCURRENCY=1`) y cancelación cooperativa.
3. **Implementación del subsistema RAG y núcleo evaluativo (`LLM-as-a-Judge`):** fusión semántica de consultas (`"{escenario_efectivo}: {texto}"`), conexión asíncrona mediante `AsyncOpenAI` hacia Groq Cloud (`openai/gpt-oss-120b`, `temperature=0.1`, *Structured Outputs*), evaluación bajo rúbricas MCER (B1–C2) y cálculo determinista en Python de `lexical_density_ratio`.
4. **Exposición de contratos API, seguridad y observabilidad:** construcción de la API RESTful y WebSocket (`/api/v1/ws/practice`) en `FastAPI` + `Uvicorn`, validación estricta de esquemas con `Pydantic v2`, autenticación *fail-closed* (`hmac.compare_digest`), portales `/docs` y `/redoc`, y consola demostrativa `/ui`.
5. **Contenerización y despliegue:** empaquetado mediante `Dockerfile` *multi-stage* con pre-descarga de modelos (*Zero Cold-Start*) y definición de dos rutas operativas: túnel seguro HTTPS/WSS con `cloudflared` (costo $0) y despliegue continuo 24/7 en contenedor Docker.

---

## 3.3 Técnicas, instrumentos y protocolo de validación

La recolección de datos se apoya exclusivamente en **instrumentos automatizados y métricas cuantitativas emitidas por el propio motor**, eliminando sesgos de apreciación subjetiva.

### 3.3.1 Verificación de software (`pytest`)

Se emplea una suite automatizada de **72 pruebas unitarias y de integración** (`tests/`) con inyección de dependencias y dobles de prueba (`tests/conftest.py`), permitiendo validar en menos de `2 s` y sin acceso a red:
* Cumplimiento de contratos REST y WebSocket (`tests/test_api.py`), autenticación (`401` / cierre WS `1008`), saturación de cola (`429`) y timeouts por etapa (`504`).
* Exactitud de normalización WER/CER, cálculo de WPM, corte temprano de audios `> 60 s` y cancelación cooperativa (`tests/test_asr.py`).
* Sanitización de fragmentos RAG y ordenamiento por distancia coseno (`tests/test_rag.py`).
* Cálculo determinista de densidad léxica y manejo estricto de fallos del LLM (`tests/test_llm.py`).
* Validación de configuraciones y esquemas `Pydantic v2` (`tests/test_config.py`, `tests/test_schemas.py`).

### 3.3.2 Protocolo de *benchmarking* controlado (`tests/benchmark_thesis.py`)

Para contrastar cuantitativamente las metas de la hipótesis, se define un protocolo reproducible en tres comandos que exporta evidencia en formato JSON (`benchmark_results/`):
1. **Exactitud fonética (ASR):** cálculo de $\text{WER}$ y $\text{CER}$ mediante `jiwer` sobre un directorio de audios técnicos de referencia (`benchmark_thesis.py wer --audio-dir ...`), con meta $\text{WER} < 12\%$.
2. **Calidad de recuperación semántica (RAG):** cálculo de $\text{Recall@3}$ y $\text{Recall@5}$ sobre un conjunto de consultas técnicas etiquetadas (`benchmark_thesis.py rag --queries ... --ks 3 5`), con meta $\text{Recall@3} \ge 85\%$.
3. **Latencia de extremo a extremo:** perfilado por etapa (ASR, RAG, LLM) y tiempo total sobre múltiples ejecuciones (`benchmark_thesis.py latency --audio ... --runs 10`), documentando el perfil exacto de hardware evaluado.

### 3.3.3 Protocolo de prueba piloto con usuarios reales en endpoints

Para validar el comportamiento del motor frente a variabilidad acústica y lingüística real de hablantes hispanos sin requerir un estudio longitudinal prolongado, se establece un **estudio piloto transversal** con una muestra sugerida de **10 a 15 participantes** (estudiantes de últimos semestres o egresados de Ingeniería en Computación). Cada participante ejecuta **dos interacciones** directamente contra los endpoints expuestos:

1. **Interacción guiada con texto de referencia (`expected_transcript`):** lectura en voz alta de un fragmento técnico en inglés correspondiente a un escenario del catálogo (`POST /api/v1/evaluate/voice` con `expected_transcript`), permitiendo al motor calcular automáticamente el $\text{WER}$, $\text{CER}$, confianza acústica (`confidence_score`) y velocidad de habla (`words_per_minute`) en hablantes reales.
2. **Interacción libre por escenario técnico:** respuesta oral o escrita a un escenario profesional (por ejemplo, `system_design_interview`, `code_review` o `daily_standup`) enviada a `POST /api/v1/evaluate/voice` o `POST /api/v1/evaluate/text`, registrando el nivel MCER asignado (`cefr_level`), `overall_score`, `grammar_accuracy_score`, `technical_precision_score`, `lexical_density_ratio`, términos técnicos detectados y latencia total.

---

## 3.4 Cronograma de actividades

El desarrollo de la investigación durante el año 2026 refleja formalmente el ajuste de alcance aprobado en septiembre de 2026 (pivote de plataforma monolítica a Motor Core MaaS):

| Periodo (2026) | Fase | Actividades principales | Entregable / Hito |
| :--- | :--- | :--- | :--- |
| **Ene – Ago** | Investigación y diseño inicial | Revisión de literatura (Marco Teórico), protocolo de investigación y curación inicial del corpus técnico (`213,439` vectores en `ChromaDB`). | Protocolo aprobado y base vectorial indexada. |
| **Septiembre** | Reestructuración y API Base | Solicitud formal de ajuste de alcance a Motor MaaS; implementación de endpoints base en `FastAPI` y conexión con `RAGService`. | API Base y Vector Store funcional. |
| **Octubre** | Pipeline Multimodal e Inferencia | Integración de `faster-whisper` (`int8`) con `Groq Cloud` (`openai/gpt-oss-120b`), cálculo determinista de densidad léxica, WebSockets y suite de `72` pruebas. | Motor de evaluación asíncrono `v1.0.0`. |
| **Noviembre** | Despliegue, Benchmarks y Piloto | Contenerización Docker, despliegue con Cloudflare Tunnel, ejecución de `benchmark_thesis.py` y pruebas piloto en endpoints con usuarios. | Sistema desplegado y métricas empíricas recolectadas. |
| **Diciembre** | Cierre y Defensa | Análisis cuantitativo de resultados, redacción final de tesis bajo APA 7 y preparación para defensa de grado (Dic 2026 – Ene 2027). | Documento de tesis completo. |

---

## 3.5 Recursos

* **Recursos humanos:** Sustentante (autor e ingeniero de desarrollo), asesora de tesis (Ing. Luisa Natalia Guerra Cuevas) y muestra piloto de 10 a 15 participantes del área de tecnología para pruebas en endpoints.
* **Recursos de software y servicios:** Python, `FastAPI`, `Uvicorn`, `Pydantic v2`, `faster-whisper` (`CTranslate2`), `ChromaDB`, `SentenceTransformers` (`all-MiniLM-L6-v2`), `jiwer`, `pytest`, `Docker`, `cloudflared`, Hugging Face Hub y API de Groq Cloud (`openai/gpt-oss-120b`).
* **Recursos de hardware e infraestructura:** Equipo de cómputo local para desarrollo y pruebas de túnel (mínimo requerido en ejecución: `3.5–4.5 GB` de RAM y `~4.5 GB` de almacenamiento; recomendado para producción 24/7 en contenedor: `4 vCPU` y `8 GB` de RAM).
