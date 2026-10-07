# 119 casos de Birads 3-5

# MamoIA · pipeline docente de detección temprana de cáncer de mama (octubre 2026)

Basado en la tesis de Sandra L. de la Fuente (CICATA-IPN, 2015) y su base DB-2015-IMG (981 imágenes) + DB-segmentation-2015 (101 resultados del CADe). Entregable: `mamografia-ia.zip` (paquete Python `mamoia`, notebooks Colab, API FastAPI, Supabase, prompt de Lovable).

## Decisiones
- PyTorch + Colab; paquete + notebooks.
- LLM intercambiable: template / Claude (default `claude-sonnet-5-5`) / Ollama. El LLM sólo recibe JSON o texto anonimizado (nunca imágenes); BI-RADS y conducta salen de reglas y se validan.
- App web: Lovable (frontend, GitHub sync) → Supabase (Auth, tablas studies/images/analyses/reviews, RLS, Storage, Edge Function `analyze`) → API Python.

## Datos con ground truth (actualización 7-oct-2026)
- DICOM por paciente en Drive (carpeta compartida `1V7nw-jPS-Exgji956VLryd_3kqeKzMk_`, `Paciente_001…119/00000001-4`, sin extensión).
- Reportes en Excel: PX_1-15 (5 B3, 5 B4, 5 B5), PX_16-53 (38 B5), PX_54-119 (66 B4). Total 119: B3 5 · B4 71 · B5 43.
- Extractor por reglas (`text/report_parser.py`): densidad, lateralidad (L49 R42 B13 U15), lesión (masa 80), biopsia en addendum (malignas 020/029/072, benignas 054/060/097). Anonimiza nombres de médicos/fechas.
- Ground truth débil por mama: mama con hallazgo = BI-RADS del reporte; contralateral = negativo si el hallazgo es unilateral; la biopsia manda. Target por defecto `b5_vs_resto`.
- Notebook principal: `notebooks/MamoIA_Colab_DICOM.ipynb` (descarga Drive API → DICOM a PNG → etiquetas → VC 5 folds por paciente → Grad-CAM/CADe → CNN-LSTM → reporte bilateral → export). Probado en seco con DICOM simulados.
- `reportes_extraidos_revision.xlsx` para que el radiólogo valide lateralidad y tipo de lesión.

## Hallazgos sobre la base 2015 (JPEG)
- Nombres `PPPV`/`PPPeSV`; vista 1=L-CC, 2=R-CC, 3=L-MLO, 4=R-MLO. 20 duplicados. Placas en negativo, algunas con apellido impreso.
- Posible correspondencia de IDs con Paciente_001-119 (sin verificar).

## Pendiente
- Correr el notebook con los DICOM reales; revisar `vistas_revision.csv` si faltan etiquetas DICOM de vista.
- Validar las etiquetas extraídas con el radiólogo; desplegar API y app en Lovable.

Nota: Por privacidad de datos, el acceso es restringido. Favor de solicitar más información al correo contacto@sandradelafuente.com
