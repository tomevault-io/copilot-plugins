## fastapi-process-hygiene-windows

> Eviter les processus FastAPI/Uvicorn bloques sur Windows


# FastAPI process hygiene (Windows / PowerShell)

- Objectif: toujours faire un lancement propre et un redemarrage propre, sans processus zombies.
- Port par defaut: `8000` (adapter si besoin).
- Ne jamais supposer qu'un ancien process est mort: verifier puis nettoyer explicitement.
- Ne jamais conclure "serveur OK" sans verification HTTP (`/docs` ou `/health`).

## Lancement propre (premier start)

1) Se placer dans le bon dossier backend et activer l'environnement Python du projet.
2) Verifier/installer les dependances si necessaire.
3) Liberer le port cible avant le lancement.
4) Lancer `uvicorn` via `python -m` (plus fiable que binaire global).

```powershell
$port = 8000
$host = "127.0.0.1"
$app = "app.main:app"

$pids = Get-NetTCPConnection -LocalPort $port -State Listen -ErrorAction SilentlyContinue |
  Select-Object -ExpandProperty OwningProcess -Unique
if ($pids) { $pids | ForEach-Object { Stop-Process -Id $_ -Force -ErrorAction SilentlyContinue } }
Start-Sleep -Seconds 1

python -m uvicorn $app --host $host --port $port --reload
```

## Redemarrage propre (apres modifications)

1) Arreter explicitement le process actif (`Ctrl+C` si terminal interactif).
2) Nettoyer le port pour supprimer tout process residuel.
3) Relancer `uvicorn`.
4) Verifier la sante HTTP avant de continuer.

```powershell
$port = 8000
$host = "127.0.0.1"
$app = "app.main:app"

$pids = Get-NetTCPConnection -LocalPort $port -State Listen -ErrorAction SilentlyContinue |
  Select-Object -ExpandProperty OwningProcess -Unique
if ($pids) { $pids | ForEach-Object { Stop-Process -Id $_ -Force -ErrorAction SilentlyContinue } }
Start-Sleep -Seconds 1

python -m uvicorn $app --host $host --port $port --reload
```

## Verification minimale obligatoire

```powershell
Invoke-WebRequest -Uri "http://127.0.0.1:8000/docs" -UseBasicParsing | Select-Object StatusCode
```

## Regles anti-erreur

- Si le port est occupe apres `Stop-Process`, relancer une seconde verification `Get-NetTCPConnection` avant tout nouveau start.
- Apres modification de dependances (`requirements`, `pyproject`, venv), toujours faire un redemarrage complet.
- Eviter les commandes `Get-Process -Id ...` avec IDs anciens: retrouver d'abord les PIDs depuis le port actif.

---
> Source: [abekkali/MtpFastAPIocr](https://github.com/abekkali/MtpFastAPIocr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
