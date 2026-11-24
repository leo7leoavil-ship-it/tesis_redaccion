# Estructura Maestra de Tesis · Tech Tutor Engine

**Institución:** Universidad Autónoma de Tlaxcala — Unidad Académica Multidisciplinaria Campus Calpulalpan  
**Programa:** Licenciatura en Ingeniería en Computación  
**Título:** Asistente de Inteligencia Artificial Especializado en Inglés Técnico para Ingenieros del Área de Tecnología  
**Autor:** Jesus Leonardo Beltrán Avila  
**Asesora:** Ing. Luisa Natalia Guerra Cuevas  
**Fecha objetivo:** Diciembre 2026 (Defensa: Diciembre 2026 – Enero 2027)  
**Norma de citación:** APA 7.ª edición (`Normas_Redaccion/Guia-Normas-APA-7ma-edicion.pdf`)

---

## 1. Índice Maestro Corregido (Alineado al Pivote MaaS)

Este índice respeta la estructura oficial de 6 capítulos de `Plantilla(1).docx`, pero sustituye las secciones heredadas del proyecto monolítico anterior (React/Next.js, Hemi, encuestas de usuario, modelo E-R) por el alcance real de **`tech-tutor-engine` v1.0.0** (Motor MaaS desacoplado) e integra el Marco Teórico ya trabajado en `Protocolo_BeltranAvila_JesusLeonardo(1).docx` y `Articulos/`.

```text
PORTADA, AGRADECIMIENTOS, RESUMEN / ABSTRACT, ÍNDICE GENERAL, ÍNDICE DE FIGURAS Y TABLAS

CAPÍTULO I. INTRODUCCIÓN
  1.1 Planteamiento del problema técnico
  1.2 Justificación académica, profesional y social
  1.3 Objetivo general
  1.4 Objetivos específicos
  1.5 Hipótesis
  1.6 Alcances y limitaciones (Espacio, Tiempo, Población, Enfoque y Delimitación MaaS)

CAPÍTULO II. MARCO TEÓRICO
  2.1 Antecedentes y evolución del aprendizaje de lenguas asistido por tecnología
      2.1.1 Evolución tecnológica en la enseñanza de idiomas
      2.1.2 Aprendizaje mediado por la tecnología (CALL / MALL / IA)
      2.1.3 Entornos Virtuales de Aprendizaje (EVA)
  2.2 Marco pedagógico y lingüístico en ingeniería
      2.2.1 Constructivismo social y su adaptación a entornos digitales
      2.2.2 Aprendizaje significativo aplicado a la ingeniería
      2.2.3 Fonética y fonología del idioma inglés (evaluación bajo MCER / CEFR B1–C2)
      2.2.4 Estrategias de autoaprendizaje y autonomía del ingeniero
      2.2.5 Fundamentos de gamificación y sistemas de retención (proyección para clientes)
  2.3 Marco tecnológico: Inteligencia Artificial y Procesamiento del Lenguaje y Habla
      2.3.1 Procesamiento de Lenguaje Natural (NLP), Modelos de Lenguaje (LLM) y arquitectura RAG
      2.3.2 Procesamiento Digital del Habla y Reconocimiento Automático de Voz (ASR)
      2.3.3 Arquitectura Model-as-a-Service (MaaS), APIs asíncronas (REST/WebSocket) y seguridad fail-closed
  2.4 Ética y gobernanza de la Inteligencia Artificial en el contexto educativo mexicano
      2.4.1 Privacidad, seguridad y protección de datos (procesamiento en memoria)
      2.4.2 Equidad algorítmica y mitigación de sesgo lingüístico en la evaluación
      2.4.3 El binomio Humano-IA y la transformación del rol docente

CAPÍTULO III. METODOLOGÍA
  3.1 Enfoque y tipo de investigación (Tecnológica / Aplicada, enfoque cuantitativo-experimental)
  3.2 Diseño metodológico y arquitectura de desarrollo del motor MaaS
      3.2.1 Fases de ingeniería: Data Track (Corpus), ASR, RAG, LLM-as-a-Judge, API/WS y Despliegue
      3.2.2 Diseño de subsistemas (ASR en memoria, ChromaDB 213,439 vectores, Groq + densidad léxica)
  3.3 Técnicas, instrumentos y protocolo de validación cuantitativa
      3.3.1 Suite automatizada de pruebas unitarias e integración (72 pruebas en pytest)
      3.3.2 Protocolo de benchmarking empírico (`benchmark_thesis.py`: WER, Recall@3, Latencia)
  3.4 Cronograma de actividades (Enero – Diciembre 2026, reflejando el ajuste de septiembre 2026)
  3.5 Recursos tecnológicos, de software y de infraestructura

CAPÍTULO IV. RESULTADOS
  4.1 Implementación del sistema (`tech-tutor-engine` v1.0.0)
      4.1.1 Arquitectura general y flujo multimodal asíncrono (`evaluation_pipeline.py`)
      4.1.2 Subsistema ASR (`faster-whisper large-v3-turbo`, int8, VAD, cancelación cooperativa)
      4.1.3 Subsistema RAG (`ChromaDB`, `all-MiniLM-L6-v2`, sanitización y fusión de escenarios)
      4.1.4 Subsistema LLM-as-a-Judge (`openai/gpt-oss-120b`, MCER/CEFR y densidad léxica determinista)
      4.1.5 Contratos de interfaz (REST, WebSocket, 9+1 escenarios), seguridad y consola `/ui` + `/docs`
      4.1.6 Estrategia de contenerización y despliegue (Cloudflare Tunnel y Docker)
  4.2 Resultados de pruebas y benchmarks
      4.2.1 Cobertura funcional y de contratos (`pytest -v`: 72 pruebas)
      4.2.2 Exactitud fonética (WER / CER)
      4.2.3 Calidad de recuperación semántica (Recall@k)
      4.2.4 Perfilado de latencia por etapa y extremo a extremo
  4.3 Evaluación del cumplimiento de objetivos y contrastación de hipótesis

CAPÍTULO V. DISCUSIÓN
  5.1 Interpretación técnica de los resultados (trade-offs CPU int8, beam_size=1, RAG vs. alucinación)
  5.2 Comparación con soluciones y estudios previos
  5.3 Implicaciones académicas, profesionales y de ingeniería de software
  5.4 Limitaciones técnicas y de alcance del estudio

CAPÍTULO VI. CONCLUSIONES
  6.1 Logros alcanzados
  6.2 Cumplimiento de objetivos
  6.3 Aportes a la Ingeniería en Computación
  6.4 Recomendaciones y Trabajo Futuro (Frontend React/Next.js, agente Hemi, evaluación con usuarios)

REFERENCIAS (Formato APA 7.ª edición con DOI/ISSN/URL)

ANEXOS (Actualizados al Motor MaaS)
  Anexo A. Diagrama de arquitectura del sistema y flujo de pipeline multimodal
  Anexo B. Especificación OpenAPI 3.x y esquema de contratos JSON (Pydantic v2)
  Anexo C. Fragmentos relevantes de código fuente (ASR, RAG, LLM y seguridad fail-closed)
  Anexo D. Resultados completos de la suite de pruebas (`pytest`) y salidas JSON de `benchmark_thesis.py`
  Anexo E. Guía operativa de despliegue (Docker / Cloudflare Tunnel)
```

---

## 2. Matriz de Congruencia Metodológica

| Objetivo Específico | Variable / Dimensión | Instrumento / Implementación | Meta / Criterio de Éxito | Capítulo donde se desarrolla |
| :--- | :--- | :--- | :--- | :--- |
| **OE1:** Construir e indexar un corpus técnico especializado para recuperación semántica. | Calidad y cobertura del corpus (`tech_tutor_corpus`) | `ChromaDB` + `all-MiniLM-L6-v2` (384d, distancia coseno) + sanitización + fusión de escenarios (`9 + custom`). | 213,439 fragmentos indexados; **Recall@3 ≥ 85 %** (`benchmark_thesis.py rag`). | Cap. III (3.2), Cap. IV (4.1.3, 4.2.3) |
| **OE2:** Implementar un subsistema ASR asíncrono en memoria optimizado para terminología técnica. | Precisión de transcripción y control de concurrencia | `faster-whisper` (`large-v3-turbo`, CPU `int8`, `beam_size=1`), VAD, `initial_prompt`, semáforo (1), cancelación cooperativa. | **WER < 12 %** (`benchmark_thesis.py wer`); rechazo temprano >5 MB / >60 s; sin I/O en disco. | Cap. III (3.2), Cap. IV (4.1.2, 4.2.2) |
| **OE3:** Integrar un evaluador LLM-as-a-Judge determinista bajo rúbricas MCER/CEFR (B1–C2). | Precisión evaluativa y determinismo estructural | `openai/gpt-oss-120b` (Groq Cloud, `temp=0.1`, Structured Outputs) + `compute_lexical_density()` en Python. | Salida JSON válida sin alucinación aritmética; política estricta de fallos (`502`/`504`). | Cap. III (3.2), Cap. IV (4.1.4, 4.2.1) |
| **OE4:** Exponer el motor como servicio MaaS seguro, contenerizado y verificable cuantitativamente. | Disponibilidad, seguridad, latencia y mantenibilidad | FastAPI REST + WebSocket (`/api/v1/ws/practice`), auth *fail-closed*, `/ui`, `/docs`, `/redoc`, Docker multi-stage. | **72/72 pruebas** en `pytest`; latencia perfilada por etapa (meta < 2.5 s o documentada por perfil hardware). | Cap. III (3.3), Cap. IV (4.1.5–4.1.6, 4.2.1, 4.2.4) |

---

## 3. Mapa de Insumos del Proyecto por Capítulo

| Capítulo | Archivos fuente en el workspace | Notas de uso |
| :--- | :--- | :--- |
| **Cap. I** | `Plantilla(1).docx`, `Antigua_version/Protocolo_BeltranAvila_JesusLeonardo(1).docx`, `Antigua_version/SOLICITUD DE RESTRUCTURACIÓN...docx`, `Articulos/Documento_completo.pdf-PDFA.pdf`, `Documentos_tecnicos/Especificacion_tech_engine(1).md` | Reemplazar referencias a UI React/Next.js por alcance MaaS; verificar referencias completas de IMCO (2024) y ManpowerGroup (2025). |
| **Cap. II** | `Articulos/6.1.1` a `6.3.3`, `Articulos/IJEH-19-1-178-182.pdf` (Ding 2025), `Articulos/RE1_SIEF-V16N2-June2023-Huang.pdf` (Huang 2023), `Articulos/sustainability-15-12451.pdf` (Kamalov 2023), `Antigua_version/Terminos del aprendizaje.pdf`, `Antigua_version/Protocolo...docx` | Migrar secciones `6.1`–`6.4` del Protocolo, enriquecer con citas directas y páginas de los PDFs, y actualizar menciones tecnológicas al stack final. |
| **Cap. III y IV** | `Documentos_tecnicos/Especificacion_tech_engine(1).md`, `Documentos_tecnicos/README.md`, `Documentos_tecnicos/GUIA_DESPLIEGUE_PRODUCCION.md`, `Documentos_tecnicos/openapi(1).json` | Usar cifras y nombres de módulos exactos. Integrar resultados reales de `tests/benchmark_thesis.py` cuando estén disponibles. |
| **Cap. V y VI** | `Documentos_tecnicos/Especificacion_tech_engine(1).md`, `Antigua_version/SOLICITUD DE RESTRUCTURACIÓN...docx`, resultados de Cap. IV | Contrastar métricas obtenidas contra metas (`WER < 12%`, `Recall@3 ≥ 85%`) y documentar React/Next.js + Hemi + pruebas con usuarios como Trabajo Futuro. |
| **Estilo y Citas** | `Normas_Redaccion/Guia-Normas-APA-7ma-edicion.pdf`, `Normas_Redaccion/Conectores_textuales.Nivel. B2...`, `Normas_Redaccion/lista-de-conectores.pdf` | Aplicar APA 7 en toda cita narrativa/parentética y cuidar cohesión académica con conectores formales. |
