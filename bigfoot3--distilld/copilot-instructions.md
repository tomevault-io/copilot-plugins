## distilld

> Proxy d'observation et de distillation de protocole entre agents IA.

# distilld — Contexte projet

Proxy d'observation et de distillation de protocole entre agents IA.
Invariant central : **mode shadow strict**. Le proxy relaie sans jamais
bloquer, modifier ni réécrire un message. La promotion d'un schéma ne
change que le reporting, pas le routage.

## Tests

```bash
cd /root/distilld && venv/bin/python -m pytest tests/ -v
```

Ni `pyproject.toml` ni `pytest.ini` dans le dépôt : cette commande n'est déductible d'aucun
artefact, la fiche est son seul domicile.

Commandes `cli.py` (run, demo, mine, schemas, promote/demote, metrics, report) : README,
§ Commandes.

- Ports : proxy 8900, agent B 8002. **8080 est pris par un service prod du VPS.**
- Les logs JSONL s'accumulent dans store/logs/ : purger avant une mesure propre.
- `demo --seed N` génère un trafic aux valeurs inédites, structure identique
  (expérience de généralisation : miner sur la graine par défaut, mesurer sur une autre).

## Pièges

- **GenSON `$schema`** : GenSON émet `"$schema": "http://json-schema.org/schema#"`,
  que jsonschema ne résout pas (DeprecationWarning, erreur à terme). La clé est
  retirée dans `infer_raw_schema()`. Ne pas la réintroduire.
- **Relais avant validation** : dans `proxy/server.py`, le relais ne dépend
  jamais du résultat de la validation shadow (échec = compté + loggé dans
  `echec_validation`, c'est tout). Invariant défensif, ne pas "simplifier".
- **`method_correle`** : les réponses JSON-RPC ne portent pas de `method` ;
  le proxy écrit ce champ sur chaque ligne de réponse au moment du relais.
  `miner/cluster.classify()` et le validator en dépendent. Pas de corrélation
  par id ailleurs.
- **Tests et seuils** : `test_sous_le_seuil` abaisse `min_occurrences` à 3 dans
  une config locale au test (3 occurrences ne généreraient rien avec la config
  par défaut) ; les seuils de promotion restent ceux de config.toml.
- **schemas.json** : écriture atomique (.tmp + os.replace) via `cli._write_schemas`.
  Un statut posé par promote/demote (`manuel: true`) est figé pour les mine suivants.
- **DISTILLD_LIVE=1** : `cli.py demo --live` pose cette variable dans
  l'environnement du sous-processus agent B, qui rédige alors ses `result` via
  Haiku. Lancer demo_b seul en mode live exige de la poser à la main.
- **Budget live partagé A/B** : le compteur = nombre de lignes de
  `store/logs/live_raw.jsonl` (plafond `max_live_calls` dans `[live]`).
  Purger les logs remet le compteur à zéro. La seule normalisation des sorties
  Haiku est `FENCE_RE` (retrait des fences markdown) : rien d'autre, jamais.
- **report ne compte que les lignes à `payload`** : `live_raw.jsonl` (appels
  API bruts) vit dans le même dossier que `exchanges.jsonl` mais n'est pas un
  log de messages. Globber `*.jsonl` sans ce filtre gonfle `messages_total`
  (bug corrigé en session 2026-07-13, ne pas le réintroduire).
- **Cluster `_raw`** : une sortie LLM non parsable part en `{"_raw": ...}` ;
  le miner la clusterise comme n'importe quel squelette et peut apprendre un
  schéma d'erreur de transport (vu : `task.status.reponse` live). Exclure ces
  lignes du corpus avant un mine de production.

## Contraintes

- Dépendances : fastapi, uvicorn, httpx, typer, jsonschema, pydantic, genson.
  anthropic optionnel (mode --live uniquement, jamais appelé sans ANTHROPIC_API_KEY).
- Persistance JSON/JSONL uniquement. Pas de base de données, pas de dashboard.
- Commentaires en français, noms de fonctions en anglais.

---
> Source: [BigFoot3/distilld](https://github.com/BigFoot3/distilld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
