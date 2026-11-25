# Justificación Técnica de Nomenclatura y Versionado: `hemi-core` (`v1.0.0`)

**Título de la tesis:** *Ingeniería aplicada con arquitectura desacoplada de un motor de inteligencia artificial para diagnosticar inglés técnico en ingeniería de software y nube*  
**Identidad del sistema / agente:** Hemi  
**Motor de inferencia y diagnóstico:** `hemi-core` (`v1.0.0`)

---

## 1. Propósito del documento

Este documento establece y justifica la decisión de adoptar **`hemi-core`** como nombre técnico del motor de inteligencia artificial desacoplado desarrollado en la tesis, así como la asignación explícita de su código de versión inicial (**`v1.0.0`**). El objetivo es asegurar consistencia terminológica entre el código fuente, la documentación técnica, los contratos de interfaz y la memoria escrita de investigación.

---

## 2. Relación entre la identidad del sistema (**Hemi**) y el motor (**`hemi-core`**)

En el desarrollo contemporáneo de sistemas basados en inteligencia artificial, es una práctica consolidada separar la **identidad visible de interacción** del **núcleo computacional de inferencia**. Esta distinción permite comunicar con claridad qué parte corresponde a la experiencia del usuario y qué parte constituye la ingeniería del servicio subyacente:

* **Hemi (Identidad del agente y capa de experiencia):** Representa la identidad conceptual y conversacional del sistema. Su diseño responde a una estética de minimalismo industrial y cumple una función pedagógica y comunicativa: actuar como un colega técnico senior que reduce el filtro afectivo durante la evaluación oral y escrita en inglés técnico, proyectando estados operativos claros (*Idle*, *Listening*, *Analyzing*, *Speaking*, *Correction*).
* **`hemi-core` (Motor desacoplado de diagnóstico):** Constituye el núcleo funcional e ingenieril construido y validado en esta tesis. Opera bajo un enfoque *Model-as-a-Service (MaaS)* e implementa de forma independiente la captura y transcripción automática del habla (ASR), la recuperación semántica sobre un corpus especializado en ingeniería de software y nube (RAG), la evaluación diagnóstica alineada al Marco Común Europeo de Referencia (MCER/CEFR) y el cálculo determinista de métricas léxicas.

Al establecer esta separación desde la concepción del sistema, **Hemi** se integra de forma coherente desde la etapa actual sin exigir la construcción prematura de una interfaz corporizada compleja, dejando preparada una arquitectura limpia para conectar múltiples clientes (interfaces web ligeras, entornos de desarrollo o la interfaz visual reactiva de Hemi en iteraciones futuras).

---

## 3. Justificación del nombre técnico: `hemi-core`

La elección de **`hemi-core`** responde a criterios técnicos, arquitectónicos y de ingeniería de software:

1. **Alineación con el título de la tesis:** El título enfatiza la *ingeniería aplicada con arquitectura desacoplada de un motor de inteligencia artificial*. El sufijo **`core`** comunica de manera precisa que el artefacto desarrollado es el **núcleo de procesamiento y diagnóstico**, evitando ambigüedades con aplicaciones monolíticas o interfaces gráficas finales.
2. **Desacoplamiento explícito:** Nombrar al servicio como `hemi-core` refuerza la separación de responsabilidades (*Separation of Concerns*). El motor expone sus capacidades mediante contratos bien definidos (API REST y WebSocket), lo que permite que distintos clientes consuman el diagnóstico sin acoplarse a la lógica interna de inferencia.
3. **Neutralidad frente a cambios de interfaz:** Mientras las capas visuales o de presentación pueden evolucionar hacia distintos canales (portal web de validación, CLI, plugins o avatar interactivo), el núcleo mantiene su identidad técnica estable como proveedor central de diagnóstico lingüístico-técnico.
4. **Convención estándar en ingeniería:** El formato en minúsculas con guion (`hemi-core`) sigue las convenciones habituales para repositorios, servicios backend, imágenes de contenedores, paquetes y artefactos de despliegue en entornos cloud-native.

---

## 4. Justificación del código de versión (`v1.0.0`)

Incluir explícitamente el código de versión **`v1.0.0`** junto a `hemi-core` aporta rigor metodológico y trazabilidad de ingeniería al proyecto de tesis:

### 4.1. Trazabilidad y reproducibilidad
En un trabajo de ingeniería aplicada, los resultados experimentales (latencia, precisión de transcripción, calidad de recuperación documental, consistencia de evaluación y estabilidad bajo concurrencia) están vinculados a una configuración específica de arquitectura, modelos, esquemas de datos y límites operativos. La etiqueta **`v1.0.0`** delimita con exactitud la línea base (*baseline*) sobre la cual se ejecutaron las pruebas y se obtuvieron las métricas reportadas en la tesis.

### 4.2. Delimitación del alcance entre la tesis y el trabajo futuro
El versionado permite distinguir de forma inequívoca:
* **`hemi-core v1.0.0`:** Alcance concluido y validado en la tesis (arquitectura desacoplada MaaS, flujo de diagnóstico oral/textual, anclaje documental técnico, cálculo de densidad léxica y portales nativos de operación).
* **Versiones posteriores (`v1.1.0`, `v2.0.0`, etc.):** Evoluciones planificadas como trabajo futuro, tales como la incorporación de síntesis de voz (TTS) en tiempo real, el acoplamiento completo con la interfaz visual animada de Hemi o la ampliación del corpus técnico.

### 4.3. Madurez como producto de ingeniería
El uso de **Versionado Semántico (`MAJOR.MINOR.PATCH`)** —iniciando en `1.0.0` para la primera versión funcional completa del motor— refleja que el artefacto cuenta con contratos de entrada/salida estables, manejo controlado de errores, configuración externalizada y verificación automatizada, superando el nivel de un prototipo exploratorio sin control de cambios.

---

## 5. Convención de uso recomendada en la tesis y el repositorio

Para mantener uniformidad en toda la documentación:

* **Primera mención en capítulos principales o resumen:** Utilizar la forma completa para establecer el vínculo entre identidad, motor y versión; por ejemplo: *"el motor de diagnóstico desacoplado `hemi-core` (`v1.0.0`), núcleo del ecosistema Hemi"*.
* **Menciones sucesivas en el cuerpo del texto:** Emplear **`hemi-core`** para referirse al servicio backend/motor y **Hemi** para referirse a la identidad del agente o su proyección de interacción.
* **Encabezados técnicos, etiquetas de repositorio y despliegues:** Mantener la referencia explícita **`hemi-core v1.0.0`**.