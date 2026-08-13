## arsante

> Módulo Odoo (localización/vertical) para **Method** que gestiona trámites regulatorios ante el ISP (Instituto de Salud Pública, Chile): cosméticos, dispositivos médicos, desinfectantes y alimentos (UYD). Cada tipo de trámite es un modelo propio con su wizard de creación y vistas asociadas, y todos se agregan en un registro consolidado (`arsante.all_record`) usado para dashboard y facturación.

# arsante

Módulo Odoo (localización/vertical) para **Method** que gestiona trámites regulatorios ante el ISP (Instituto de Salud Pública, Chile): cosméticos, dispositivos médicos, desinfectantes y alimentos (UYD). Cada tipo de trámite es un modelo propio con su wizard de creación y vistas asociadas, y todos se agregan en un registro consolidado (`arsante.all_record`) usado para dashboard y facturación.

## Dependencias
`base`, `account`, `sale`, `contacts`.

## Estructura por dominio

Los trámites siguen un patrón repetido: modelo en `models/`, wizard en `wizard/`, vista en `views/`. Dominios cubiertos:

- **Cosméticos**: `cda_cosmetico_dm`, `registro_cosmetico`, `inscripciones_cosmeticos`, `modificacion_cosmeticos`, `rectificaciones`, `renovaciones_cosmeticas`, `eximiciones_cosmeticos` (+ `eximiciones_cosmeticos_correo.py`), `exim_proceso_cosmeticos`, `registro_isp_exim_cosmeticos`.
- **Dispositivos médicos**: `registro_dispositivos_medicos`, `declaracion_dispositivos_medicos`, `dispositivos_medicos`, `rev_antecedentes_dm`.
- **Desinfectantes**: `registro_desinfectantes`, `modificaciones_desinfectantes`, `renovaciones_desinfectantes`.
- **Alimentos**: `cda_uyd_alimentos`, `uyd_alimentos`.
- **Otros**: `hds_hechas`, `inscripciones`, `registro_isp`, `marca`, `tipo_registro`, `tipo_servicio`.

## Modelos clave

- `arsante.tipo_registro` (`models/tipo_registro.py`): catálogo maestro de tipos de trámite (selection `tipo`), con contadores computados (facturados, cotizados, listos, etc.) usados en el dashboard.
- `arsante.all_record` (`models/all_record.py`): tabla consolidada de todos los registros de trámites, con `registro_id` apuntando al ID del registro específico según `tipo_registro_id`. Se actualiza vía `onchange` cruzado — al modificar campos comunes (`facturado`, `estado`, `documentacion`, etc.) se busca y sincroniza el registro origen por `tipo_registro_id.name`. **Si se agrega un nuevo tipo de trámite, hay que replicar este patrón de sincronización.**

## Integraciones con core Odoo

- `res_company`: flag `es_arsante` que activa comportamiento específico en `sale.order`.
- `sale_order`: agrega `tipo_registro_id`, `tipo_servicio_id`, `marca`, `fecha_pago` y un campo computado `tipo_serv_registro` que combina servicio/registro para mostrar en la nota de venta. Sobrescribe `_prepare_invoice`.
- `account_move`, `res_partner`: extensiones para reportes y datos del trámite.
- `report/`: reportes QWeb para `sale_order` y `account_move`.

## Dashboard

`views/dashboard.xml` + assets en `static/src/{js,css,xml}/tipo_registro_dashboard.*` (OWL). Consume los campos computados de `arsante.tipo_registro`.

## Seguridad

- `security/ir.model.access.csv` + `data/groups.xml`: grupos de acceso por rol.
- Revisar ACL al agregar un modelo nuevo — es fácil olvidar la línea correspondiente en el CSV.

## Automatización

- `data/cron.xml`: tareas programadas (revisar antes de asumir que algo se actualiza solo por acción de usuario).

## Convenciones observadas

- Los modelos no usan `_description`; mantener consistencia si se agregan nuevos.
- Los campos y strings están en español, coherente con el dominio (trámites ISP Chile).
- Los wizards siguen el patrón `<modelo>_wizard.py` + `.xml`, y son el punto de entrada normal para crear un trámite (en vez de crear el registro directo).
- Hay archivos basura de metadatos de descarga (`*:Zone.Identifier`) en `controllers/` — no tocar/no replicar, son residuos de Windows/Google Drive, no parte del módulo.

---
> Source: [Cesar250101/arsante](https://github.com/Cesar250101/arsante) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
