## sparta-ledger

> Evitar BOM en archivos PHP (pantalla en blanco)


# PHP: UTF-8 sin BOM

En este proyecto los archivos PHP **deben guardarse siempre como UTF-8 sin BOM**.
El BOM (Byte Order Mark) al inicio del archivo provoca **pantalla en blanco** o errores "headers already sent" porque PHP envía salida antes de la declaración `<?php`.

## Qué hacer

1. **Al editar PHP:** guardar con codificación **UTF-8** (no "UTF-8 with BOM").
   En Cursor/VS Code: barra inferior → clic en "UTF-8" → "Save with Encoding" → elegir **"UTF-8"**.

2. **Si aparece pantalla en blanco** tras tocar `CapHum.php` u otro PHP, ejecutar:
   ```bash
   c:\xampp\php\php.exe c:\xampp\htdocs\sparta_ledger\scripts\verificar_bom.php --fix
   ```

3. **Pre-commit:** el hook en `.git/hooks/pre-commit` quita BOM de los PHP en staging antes de cada commit. Si el archivo sigue con BOM en disco, volver a guardar como UTF-8 sin BOM o ejecutar el script anterior.

## Configuración

- `.editorconfig`: `[*.php] charset = utf-8` (sin BOM).
- `.vscode/settings.json`: `"files.encoding": "utf8"` y `"[php]": { "files.encoding": "utf8" }`.

Si aun así se introduce BOM (p. ej. al aplicar ediciones desde el asistente), ejecutar `verificar_bom.php --fix` después de editar.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JONMAXI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
