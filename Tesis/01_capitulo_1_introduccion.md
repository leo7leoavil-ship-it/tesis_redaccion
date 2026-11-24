# CAPÍTULO I. INTRODUCCIÓN

> **Estado del documento:** Borrador alineado al pivote arquitectónico **Motor MaaS (`tech-tutor-engine` v1.0.0)** y a `Plantilla(1).docx`.  
> **Pendientes marcados:** `[Verificar referencia completa APA 7]` en citas heredadas del protocolo (`IMCO, 2024`; `ManpowerGroup, 2025`).

---

## 1.1 Planteamiento del problema técnico

En la industria tecnológica contemporánea, la globalización de los equipos de desarrollo, la adopción de metodologías ágiles distribuidas y la expansión de modelos de contratación internacional exigen a los ingenieros en computación un dominio funcional del **inglés técnico**, tanto en su dimensión escrita como oral. No obstante, la formación universitaria tradicional suele priorizar la lectura de comprensión general o el aprendizaje gramatical descontextualizado, lo cual genera una brecha significativa entre el perfil de egreso y las competencias comunicativas demandadas en procesos de reclutamiento técnico, revisiones de código (*code reviews*), discusiones de arquitectura y gestión de incidentes en producción.

Por otra parte, las plataformas comerciales disponibles para el aprendizaje de idiomas (como aplicaciones de consumo masivo orientadas al inglés conversacional) están diseñadas para vocabulario cotidiano y carecen de rigor en terminología especializada de ingeniería de software, infraestructura en la nube y sistemas distribuidos. Asimismo, las soluciones basadas únicamente en modelos de lenguaje de propósito general (LLM) presentan dos limitaciones técnicas críticas cuando se utilizan sin anclaje documental ni procesamiento acústico controlado:

1. **Riesgo de alucinación terminológica y evaluativa:** al carecer de un mecanismo de recuperación sobre documentación técnica verificada, los modelos generativos pueden producir retroalimentación imprecisa o incoherente frente a estándares reales de ingeniería, además de incurrir en errores aritméticos al calcular métricas cuantitativas como la densidad léxica.
2. **Falta de evaluación multimodal integrada y reproducible:** los sistemas conversacionales convencionales no combinan, en un flujo asíncrono y auditable, el reconocimiento automático de voz (ASR) optimizado para jerga técnica con rúbricas estandarizadas del Marco Común Europeo de Referencia para las Lenguas (MCER / CEFR) y métricas objetivas de desempeño fonético y léxico.

En consecuencia, surge el problema técnico de diseñar e implementar un motor de inferencia y evaluación desacoplado (*Model-as-a-Service*, MaaS) capaz de procesar audio y texto técnico en tiempo acotado, anclar la evaluación en un corpus especializado de ingeniería mediante Generación Aumentada por Recuperación (RAG) y emitir diagnósticos deterministas y reproducibles a través de interfaces API estandarizadas.

---

## 1.2 Justificación académica, profesional y social

La presente investigación resulta pertinente para el sector tecnológico nacional e internacional debido a la creciente demanda de ingenieros capaces de operar en entornos globales de alta especialización. El dominio del inglés técnico ha dejado de ser una habilidad complementaria para consolidarse como un requisito operativo crítico en la Industria 4.0. De acuerdo con datos del IMCO (2024) `[Verificar referencia completa APA 7]`, aunque el 80 % de las vacantes de ingeniería requieren el dominio de este idioma, únicamente el 20 % de los candidatos posee el nivel necesario, lo que evidencia la urgencia de herramientas tecnológicas que aceleren la adquisición de vocabulario y fluidez técnica.

Desde una perspectiva **social y económica**, el proyecto busca contribuir a reducir la brecha comunicativa que limita la inserción laboral de estudiantes y egresados de ingeniería de habla hispana en empresas internacionales. Según reportes de ManpowerGroup (2025) `[Verificar referencia completa APA 7]`, la escasez de talento bilingüe especializado frena la competitividad del país frente a dinámicas como el *nearshoring*. Contar con una infraestructura de evaluación automatizada y accesible permite democratizar la práctica de escenarios reales de ingeniería (entrevistas de diseño de sistemas, *daily standups*, *postmortems* de incidentes), favoreciendo la movilidad profesional y el acceso a mejores oportunidades salariales.

Desde el punto de vista **técnico y metodológico**, el proyecto aporta una arquitectura desacoplada (*Model-as-a-Service*) orientada a la mitigación de alucinaciones y al rigor cuantitativo. A diferencia de prototipos monolíticos acoplados a una interfaz gráfica particular, la exposición del sistema mediante una API RESTful asíncrona y canales WebSocket en `FastAPI` permite que múltiples clientes futuros (aplicaciones web, móviles o integraciones CLI) consuman el mismo núcleo de evaluación. Adicionalmente, la combinación de inferencia local en CPU (`faster-whisper` cuantizado en `int8` y `ChromaDB` con `213,439` fragmentos técnicos vectorizados) junto con evaluación estructurada bajo rúbricas MCER/CEFR y cálculo determinista en Python de la densidad léxica, demuestra la factibilidad técnica de construir sistemas educativos reproducibles, auditables y desplegables en infraestructura contenerizada.

---

## 1.3 Objetivo general

Diseñar, implementar y validar cuantitativamente un motor de inferencia y evaluación técnica asíncrono bajo el paradigma *Model-as-a-Service* (MaaS), basado en Reconocimiento Automático de Voz (ASR), Generación Aumentada por Recuperación (RAG) y Modelos de Lenguaje de Gran Escala (LLM), para diagnosticar y retroalimentar la competencia comunicativa en inglés técnico de ingenieros del área de tecnología bajo rúbricas del Marco Común Europeo de Referencia (MCER / CEFR).

---

## 1.4 Objetivos específicos

1. **Construir e indexar un corpus técnico especializado** mediante el procesamiento, sanitización y vectorización de documentación oficial de ingeniería en `ChromaDB` (colección `tech_tutor_corpus` con embeddings `all-MiniLM-L6-v2`), habilitando la recuperación semántica contextualizada por escenario profesional.
2. **Implementar un subsistema de Reconocimiento Automático de Voz (ASR)** en memoria basado en `faster-whisper` (`large-v3-turbo`, cuantización `int8`), con detección de actividad vocal (VAD), sesgo léxico de ingeniería, control de concurrencia y cancelación cooperativa para la transcripción y cálculo de métricas acústicas y fonéticas.
3. **Desarrollar un núcleo de evaluación automatizada (*LLM-as-a-Judge*)** que integre el contexto recuperado por el subsistema RAG y la salida del ASR mediante salidas JSON estructuradas (`openai/gpt-oss-120b` en Groq Cloud) bajo rúbricas MCER/CEFR (niveles B1–C2), garantizando el cálculo determinista de la densidad léxica técnica en Python.
4. **Exponer y contenerizar el motor como una API asíncrona segura** (`FastAPI` + `Uvicorn` con endpoints REST, canal WebSocket *full-duplex*, autenticación *fail-closed*, consola de pruebas `/ui` y documentación OpenAPI), validando su desempeño mediante una suite automatizada de pruebas y un protocolo de evaluación empírica de exactitud fonética (WER), recuperación semántica (Recall@k) y latencia.

---

## 1.5 Hipótesis

La implementación de un motor de evaluación multimodal asíncrono que integra Reconocimiento Automático de Voz (`faster-whisper` con sesgo léxico técnico), Generación Aumentada por Recuperación sobre un corpus especializado de ingeniería (`ChromaDB`) y evaluación estructurada mediante LLM con cálculo determinista de densidad léxica permite diagnosticar y retroalimentar la competencia en inglés técnico bajo estándares MCER/CEFR manteniendo márgenes de exactitud fonética ($\text{WER} < 12\%$) y precisión de recuperación semántica ($\text{Recall@3} \ge 85\%$) adecuados para entornos reales de práctica profesional.

### Variables de investigación

* **Variable independiente:** Arquitectura del motor MaaS (`tech-tutor-engine`) compuesta por el pipeline multimodal `ASR → RAG → LLM-as-a-Judge` con corpus técnico vectorizado y esquemas estrictos de salida.
* **Variables dependientes:**
  * *Exactitud fonética de transcripción técnica:* medida a través de la Tasa de Error de Palabra ($\text{WER}$) y Tasa de Error de Caracteres ($\text{CER}$).
  * *Pertinencia de recuperación semántica:* medida mediante $\text{Recall@k}$ sobre consultas técnicas de ingeniería.
  * *Desempeño operativo del servicio:* medido mediante latencia por etapa y extremo a extremo (segundos), cumplimiento de contratos de API y estabilidad ante concurrencia y fallos.

---

## 1.6 Alcances y limitaciones

### Alcances

* **Espacial / Operativo:** Motor backend contenerizado (`Docker`) y accesible mediante protocolos estándar HTTPS y WSS (tanto en despliegue local con túnel seguro como en servidor VPS o nube), consumible por clientes externos (cURL, Postman, aplicaciones web/móviles) y equipado con una consola web interactiva nativa (`/ui`) y portales de documentación (`/docs`, `/redoc`) sin dependencias externas.
* **Temporal:** El diseño, reestructuración arquitectónica hacia el paradigma MaaS, implementación, pruebas automatizadas y evaluación experimental del proyecto comprenden el periodo de enero a diciembre de 2026.
* **Poblacional / Dominio:** Enfocado en escenarios comunicativos propios de estudiantes de últimos semestres y profesionales hispanohablantes de ingeniería en computación y tecnología, cubriendo nueve escenarios predefinidos (`system_design_interview`, `code_review`, `incident_postmortem`, `architecture_presentation`, `daily_standup`, `technical_debugging`, `devops_ci_cd`, `database_optimization`, `api_design_review`) y un modo `custom` validado.
* **Enfoque técnico:** Desarrollo de software e ingeniería de Inteligencia Artificial aplicada (NLP, ASR y MLOps) con validación cuantitativa reproducible de los subsistemas del motor.

### Limitaciones

* **Exclusión de plataforma frontend monolítica (Trabajo Futuro):** Conforme al ajuste formal de alcance realizado en septiembre de 2026, el desarrollo de una aplicación cliente completa en React/Next.js, la gestión de cuentas de usuario con OAuth 2.0, los tableros gráficos de progreso y la interfaz agéntica visual (*Hemi*) quedan explícitamente delimitados como **Trabajo Futuro**, priorizando el rigor del núcleo de inferencia y evaluación.
* **Dependencia de proveedor externo para inferencia LLM:** Aunque los subsistemas ASR y RAG se ejecutan localmente en memoria, la etapa de razonamiento evaluativo depende de la conectividad y disponibilidad de la API externa de Groq Cloud (`openai/gpt-oss-120b`), aplicando una política estricta de fallos (`HTTP 502` / `504`) sin generación de puntajes simulados ante desconexiones.
* **Restricciones de hardware y concurrencia en CPU:** La configuración base de inferencia ASR en CPU (`int8`, `beam_size=1`) serializa las peticiones mediante un semáforo de concurrencia (`ASR_MAX_CONCURRENCY=1`) y acota las entradas a un máximo de `5 MB` y `60 segundos` de audio para preservar la estabilidad de memoria del servidor (`~3.5–4.5 GB` de RAM).
* **Validación técnica vs. estudio longitudinal con usuarios:** La evaluación empírica de esta tesis se centra en la precisión del motor (WER, Recall@k, latencia y pruebas de contrato), quedando la medición de impacto pedagógico longitudinal en cohortes de estudiantes como línea de investigación posterior.
