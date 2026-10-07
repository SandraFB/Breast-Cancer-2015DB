El siguiente proyecto es integrado a la GUI y requiere conexión a la cuenta Pro de HuggingFace correspondiente

# MamoIA · pipeline docente de detección temprana de cáncer de mama (octubre 2026)

Basado en la tesis de Sandra L. de la Fuente (CICATA-IPN, 2015). Entregable: `mamografia-ia.zip` (paquete Python `mamoia`, notebooks Colab, API FastAPI, docs).

## Decisiones
- PyTorch + Colab; paquete + notebooks (el paquete va embebido en los notebooks).
- LLM intercambiable: template / Claude (`claude-sonnet-5-5`) / Ollama. El LLM sólo recibe JSON o texto anonimizado; BI-RADS y conducta salen de reglas y se validan.
- Despliegue: modelos en Drive → API en Hugging Face Space → app Lovable con Lovable Cloud.
- Vistas: en DICOM, ImageLaterality y el texto del protocolo ("MAMOGRAFIA DCHA, CC") mandan sobre lo que elija el usuario; la API corrige y devuelve `avisos` tipo `vista_corregida`.

## Datos y resultados (7-oct-2026)
- DICOM en Drive (`1V7nw-jPS-Exgji956VLryd_3kqeKzMk_`): 119 pacientes, 520 archivos, 497 imágenes válidas (23 ilegibles, 3 de ultrasonido Ultrasonix → ahora se excluyen). FUJIFILM, MONOCHROME1, JPEG Lossless (no se ve en navegador).
- Reportes Excel: B3 5 · B4 71 · B5 43; extractor por reglas + biopsias en addendum (malignas 020/029/072, benignas 054/060/097).
- Target `b5_vs_resto`, VC 5 folds por paciente (EfficientNet-B0 1024×512): AUC imagen 0.65 (IC95 0.56-0.73), mama 0.66 (0.56-0.75), paciente 0.62 (0.51-0.74). Captioner BLEU-4 ≈ 0.41.
- Resultados en Drive: carpeta MamoIA `1uYlWtMLtgsEqhzs2SEmofdUw_2stpTgT` (export_api, checkpoints, labels, outputs, reportes).

## Despliegue
- API sin estado (x-api-key) → Space `QSandradelaf/mammoai-api` (https://qsandradelaf-mammoai-api.hf.space, Docker, requiere HF PRO) con `notebooks/MamoIA_Deploy_HF.ipynb`.
  - `/health`; `POST /preview` (1-8 archivos → miniatura JPG b64 en orientación original + lateralidad/proyección del DICOM + `confiable`; ~1 s/archivo, sin modelo).
  - Síncronos: `/analyze`, `/analyze/study`. Asíncronos (evitan el 524 del proxy HF a ~100 s): `POST /jobs/analyze`, `POST /jobs/analyze/study` → `{job_id}`; `GET /jobs/{id}` → queued|running|done(result)|error. Cola de 1 worker, TTL 1 h, en memoria.
  - `MAMOIA_THREADS` (2) evita sobresuscripción; warmup; GLCM sólo de ROIs finales; `tiempos_s` por etapa (~2 s/imagen CPU).
- Lovable: proyecto **MammoAI Assistant** (`acfc95ac-60c8-43d3-9fcd-ffab8f318ed9`, workspace Bitlab-IT). Lovable Cloud; tablas profiles, user_roles, studies, images, analyses (+ `avisos` jsonb), study_reports, reviews, vista labeled_for_training; buckets privados mamografias/resultados (miniaturas en `resultados/previews/<image_id>.jpg`). Server functions TanStack (`src/lib/mamoia.functions.ts`, `mamoia.server.ts`).
  - `callApi` (commit bd42667): `/jobs…` + sondeo 3 s hasta 6 min; respaldo síncrono si 404; errores limpios, 524 traducido.
  - Vista previa (commit 354e522): tras subir, tarjetas RCC/LCC/RMLO/LMLO con miniatura, selectores precargados del DICOM, insignia "Leído del DICOM" y advertencia si difiere; pestaña Imagen usa la miniatura; avisos de vista corregida en reporte.
- Secrets Lovable: `MAMOIA_API_URL`, `MAMOIA_API_KEY` (en Drive `export_api/mamoia_api_key.txt`).

## Pendiente
- Re-ejecutar `MamoIA_Deploy_HF.ipynb` para publicar la API con `/jobs` y `/preview`; probar un estudio en la app.
- Opcional: `ANTHROPIC_API_KEY` en el Space para redactar con Claude.
- Asignar rol docente, conectar GitHub desde Lovable, validar etiquetas con radiólogo, re-entrenar excluyendo ultrasonido.
