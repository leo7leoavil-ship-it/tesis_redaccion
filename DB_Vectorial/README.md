# Corpus Técnico para Tesis

Estructura organizada del flujo completo de descarga, depuración y vectorización del corpus técnico.

## Estructura de carpetas

### `01_notebooks_pipeline/`
Cuadernos de Jupyter ordenados según la secuencia del pipeline:
1. **`01_descarga_inicial_prueba_fastapi.ipynb`** *(antes `Scripts_de_descargas.ipynb`)*: Script inicial de descarga directa de archivos Markdown de FastAPI y plantilla base con token de GitHub.
2. **`02_descarga_masiva_repositorios_github.ipynb`** *(antes `Script_descarga_mejorado.ipynb`)*: Descarga masiva recursiva desde la API de GitHub (usando `GITHUB_TOKEN` desde `.env`) para múltiples repositorios técnicos (`.md`, `.rst`, `.txt`).
3. **`03_depuracion_y_limpieza_texto.ipynb`** *(antes `Script_de_depuracion.ipynb`)*: Sanitización paralela (`ThreadPoolExecutor`, 8 hilos) para eliminar bloques de código, imágenes, enlaces Markdown y etiquetas HTML, conservando archivos con más de 100 caracteres.
4. **`04_vectorizacion_e_ingesta_chromadb.ipynb`** *(antes `ChromaDB.ipynb`)*: Fragmentación con `RecursiveCharacterTextSplitter` (`chunk_size=600`, `chunk_overlap=60`), generación de embeddings con `all-MiniLM-L6-v2` e ingesta en ChromaDB (colección `tech_tutor_corpus`).

### `02_datos_corpus/`
- **`01_prueba_fastapi/`** *(antes `corpus_tecnico/fastapi/`)*: Archivos de prueba iniciales de FastAPI (`index.md`, `first-steps.md`, `body.md`).
- **`02_crudos_raw/`** *(antes `data/raw/`)*: Documentos técnicos originales descargados de los repositorios.
- **`03_depurados_processed/`** *(antes `data/processed/`)*: Documentos limpios listos para vectorización.

### `03_base_datos_vectorial_final/`
- **`chroma_db_corpus_tecnico/`** *(antes `technical_corpus_db/`)*: Base de datos vectorial persistente de ChromaDB (`chroma.sqlite3` e índices HNSW) con la colección `tech_tutor_corpus`.

### Archivos de configuración
- **`.env`**: Variables de entorno (p. ej. `GITHUB_TOKEN`).
- **`.gitignore`**: Exclusiones recomendadas para control de versiones.
