El siguiente proyecto es integrado a la GUI y requiere conexión a la cuenta Pro de HuggingFace correspondiente

# MamoIA · pipeline docente de detección temprana de cáncer de mama (octubre 2026)

Basado en la tesis de Sandra L. de la Fuente (CICATA-IPN, 2015). Entregable: `mamografia-ia.zip` (paquete Python `mamoia`, notebooks Colab, API FastAPI, docs).

## Decisiones
- PyTorch + Colab; paquete + notebooks (el paquete va embebido en los notebooks).
- LLM intercambiable: template / Claude (`claude-sonnet-5-5`) / Ollama. El LLM sólo recibe JSON o texto anonimizado; BI-RADS y conducta salen de reglas y se validan.
- Despliegue: modelos en Drive → API en Hugging Face Space → app Lovable con Lovable Cloud.

## Datos y resultados (7-oct-2026)
- DICOM en Drive (`1V7nw-jPS-Exgji956VLryd_3kqeKzMk_`): 119 pacientes, 520 archivos, 497 imágenes válidas (23 ilegibles, 3 de ultrasonido Ultrasonix → ahora se excluyen).
- Reportes Excel: B3 5 · B4 71 · B5 43; extractor por reglas + biopsias en addendum (malignas 020/029/072, benignas 054/060/097).
- Target `b5_vs_resto`, VC 5 folds por paciente (EfficientNet-B0 1024×512): AUC imagen 0.65 (IC95 0.56-0.73), mama 0.66 (0.56-0.75), paciente 0.62 (0.51-0.74). Captioner BLEU-4 ≈ 0.41.
- Resultados en Drive: carpeta MamoIA `1uYlWtMLtgsEqhzs2SEmofdUw_2stpTgT` (export_api, checkpoints, labels, outputs, reportes).

## Despliegue
- API sin estado (`/health`, `/analyze`, `/analyze/study`, x-api-key) → Space `QSandradelaf/mammoai-api` (https://qsandradelaf-mammoai-api.hf.space) con `notebooks/MamoIA_Deploy_HF.ipynb`.
- Lovable: proyecto **MammoAI Assistant** (`acfc95ac-60c8-43d3-9fcd-ffab8f318ed9`, workspace Bitlab-IT). Lovable Cloud activo; tablas profiles, user_roles, studies, images, analyses, study_reports, reviews, vista labeled_for_training; buckets privados mamografias/resultados. La función `mamoia-analyze` quedó como server functions TanStack (`src/lib/mamoia.functions.ts`, `mamoia.server.ts`) porque el proyecto no admite Edge Functions. UI: /auth, /estudios, /analisis, /revision, /tablero, /admin/usuarios.
- Secret pendiente en Lovable: `MAMOIA_API_KEY` (lo imprime el notebook de despliegue). `MAMOIA_API_URL` ya está.

## Pendiente
- Ejecutar el notebook de despliegue, poner la clave en Lovable, asignar rol docente, probar con un estudio real.
- Conectar GitHub desde Lovable; validar etiquetas con radiólogo.
