## vertex

> \# Projet VERTEX — Contexte

\# Projet VERTEX — Contexte



Application Python/Streamlit d'analyse de performance trail running.

Entrée : fichiers GPX/TCX/FIT. Sortie : score physiologique + rapport PDF.

Positionnement : "Strava montre ce que tu as fait. VERTEX explique ce que ça t'a coûté."

Cible : élites trail mondiaux. Déployé sur Streamlit Community Cloud.



\---



\## Stack \& Architecture



\*\*Python\*\* : 3.11–3.14 (requis par scipy 1.17.1)

\*\*Dépendances critiques\*\* :

\- `streamlit` (lancement : `python -m streamlit run app.py`)

\- `pandas==2.2.3` — NE PAS upgrader (Copy-on-Write pandas 3.0 casse le code)

\- `scipy==1.17.1`

\- `plotly`

\- `numpy`

\- `weasyprint` — PDF HTML/CSS → moteur principal (Linux / Streamlit Cloud)

\- `jinja2` — template Jinja2 pour le PDF WeasyPrint

\- `reportlab>=4.0` — fallback PDF si WeasyPrint indisponible (Windows local)

\- `fpdf2` — conservé (héritage, non utilisé en prod)

\- `lxml` — parsing GPX



\*\*Architecture modulaire\*\* :

```

gpx\_parser.py   → parsing GPX, Haversine, extract\_race\_info

tcx\_parser.py   → parsing TCX

fit\_parser.py   → parsing FIT (Garmin natif)

engine.py       → GAP (Minetti 2002), cardiac\_drift, score, VERDICT

charts.py       → graphiques Plotly + générateur PDF (WeasyPrint + fallback ReportLab)

app.py          → UI Streamlit uniquement (ce fichier)

test\_engine.py  → suite de tests (sections A→S)

```



\*\*Déploiement\*\* : Streamlit Community Cloud · repo `vertex` · branche `main`

\*\*Feedback\*\* : Tally.so (https://tally.so/r/zxeJPM) → Google Sheets



\---



\## Commandes essentielles

```bash

\# Lancer l'application

python -m streamlit run app.py



\# Lancer les tests

python -m pytest test\_engine.py -v



\# Déployer : git push sur main → auto-déploiement Streamlit Cloud

```



\---



\## Conventions de code



\- \*\*Corrections chirurgicales uniquement\*\* — jamais réécrire un fichier entier sans accord explicite

\- Fonctions vectorisées numpy prioritaires sur `.apply()` pandas

\- `@st.cache\_data` sur toutes les fonctions de parsing

\- PDF : WeasyPrint + Jinja2 · fonts Barlow Condensed + DM Mono (TTF dans `assets/fonts/`) · fallback ReportLab sur Windows

\- Imports modules VERTEX : toujours en haut de `app.py`, jamais inline

\- Cadence GPX Garmin : valeur brute unilatérale → multiplier ×2 si < 110 spm



\---



\## Fonctionnalités actives



\- \*\*GAP Engine\*\* : modèle Minetti 2002, version scalaire + vectorisée

\- \*\*cardiac\_drift()\*\* CDC v1.4 : COLLAPSE A/B · NEGATIVE\_SPLIT · DRIFT-CARDIO · DRIFT-NEURO · DRIFT · STABLE

\- \*\*Score performance\*\* : zones dynamiques (FLAT/ASCENDING/DESCENDING) + pondération adaptative

\- \*\*VERDICT\*\* : matrice V1→V7 + V5-C + INSUFFICIENT + V1-NS

\- \*\*PDF\*\* : WeasyPrint + Jinja2 · design system Claude · Barlow Condensed + DM Mono · dark theme · hero score · métriques · quartiles · pattern · reco · élévation SVG · disclaimer médical · 2 pages si contenu déborde

\- \*\*Tally form\*\* : placé HORS des blocs `if st.button` (persist on rerender)

\- \*\*Couche traduction athlète\*\* : zéro terme interne visible (pas de EF, decay\_ratio, COLLAPSE A/B)



\---



\## Sprint actif

\*\*Sprint 9 — CLOS ✅\*\* · 30/04/2026
\- A1 ✅ · CNT Antony · MIXED confirmé · ef\_iso documenté · bug DESCENDING non reproductible
\- A2 ✅ · règle iso-pente 2 paliers · seuils delta\_fc provisoires · commit f1a9cb9
\- A3 ✅ · Coralie Ventoux CDF · baseline STABLE CDC-R2 #4 · commit 9508b57
\- CDC-R2 ✅ · commit 799c76f

\*\*Sprint 10 — OUVERT\*\* · 01/05/2026
\- CD-2 ✅ · PDF redesign complet · WeasyPrint + Jinja2 · design system Claude pixel-perfect · fonts Barlow/DMMono embarquées · _build_weasy_context() + _generate_weasyprint() · fallback ReportLab Windows · packages.txt GTK · élévation SVG · 2 pages · 220/220 verts · 11/05/2026
\- CD-3 · marketing deliverables · spec à écrire
\- SCR-U1 · spec à écrire
\- ELV-B1 · spec à écrire


\---



\## Bugs connus \& décisions figées



\*\*BUG-2\*\* : COLLAPSE + decay < 0.80 → doit déclencher V5-C. Corrigé dans matrice, à intégrer Sprint 5.

\*\*Faux positif V7\*\* : angle mort structurel départ montée (dataset Coralie Toureille). Comportement connu, non corrigé.

\*\*GAP dégradé < 2–3% gradient\*\* : limitation connue modèle Minetti. Disclaimer en place.

\*\*EF biaisée faible D+\*\* : SCI-1 disclaimer actif, SCI-3 étude en cours.



\*\*Décisions figées — ne jamais remettre en question\*\* :

\- `pandas==2.2.3` — version verrouillée

\- Seuil COLLAPSE A : slope < -3.0 bpm/h + chute > 10% (CDC Elena v1.4)

\- Seuil COLLAPSE B : chute > 20% sur segments plats

\- Ultra : ≥50km \*\*ET\*\* ≥4h (critère ET strict — validé Elena CDC)

\- Tally form hors bloc `if st.button`



\---



\## Règles absolues



\- Ne \*\*jamais\*\* exposer les termes internes côté athlète : `EF`, `decay\_ratio`, `COLLAPSE A/B`, `dissociation CV`

\- Ne \*\*jamais\*\* toucher au design system sans validation : `#080E14` · `#41C8E8` · `#C84850` · `#C8A84B` · Barlow Condensed + DM Mono

\- Ne \*\*jamais\*\* upgrader `pandas` au-delà de 2.2.3

\- Ne \*\*jamais\*\* proposer de réécriture complète d'un fichier sans demande explicite

\- Toujours inclure disclaimers médicaux dans UI et PDF

\- Analytics Streamlit désactivé via `.streamlit/config.toml` — ne pas réactiver



\---



\## Fichiers clés



| Fichier | Rôle | Sensibilité |

|---|---|---|

| `engine.py` | Logique métier complète | 🔴 critique |

| `app.py` | UI Streamlit | 🔴 critique |

| `charts.py` | Graphiques + PDF (WeasyPrint + fallback ReportLab) | 🟠 élevée |

| `assets/templates/rapport_template.html.j2` | Template Jinja2 PDF — design system Claude | 🟠 élevée |

| `assets/fonts/` | Fonts TTF embarquées (Barlow × 4, DM Mono × 3) | 🟡 importante |

| `packages.txt` | Dépendances GTK/Pango Streamlit Cloud (WeasyPrint) | 🟡 importante |

| `gpx\_parser.py` | Parsing GPX | 🟠 élevée |

| `test\_engine.py` | Suite tests A→S | 🟡 importante |

| `.streamlit/config.toml` | Analytics off | 🟡 importante |



\*\*Règle absolue tests\*\* : aucun push sur `main` sans 100% tests verts.

Sections actives : A→S + C10/C11/C12 + K. État actuel : 220/220 — tous verts.

---

\## Specs actives — format P1 obligatoire

\### Règle
Tout item backlog doit avoir une spec écrite ici avant d'être codé.
Item sans spec = item inexistant.

\### Format
[ID] — Titre court
- Problème : symptôme observé ou besoin
- Fichiers concernés : engine.py / app.py / charts.py
- Données requises : GPX / dataset / référence
- Critère de clôture : ce qui prouve que c'est résolu
- Bloquant : oui / non

\### Items actifs
— audit moteur 30/04/2026 · sprint correctif 30/04 · 4/4 items clos · SCR-NS1 gelé —

[SCR-NS1 · GELÉ ❄] — Score GAP aveugle au pattern NEGATIVE_SPLIT
- Problème : 6 courses avec NEGATIVE_SPLIT ont un score_gap bas (départ lent tactique pénalisé comme chute d'allure). Écarts dataset vs moteur : CNT -58 · Dylan ChF -50 · Samuel ×2 -21 · Coralie Toureille 2023 -27 · Hivernatrail -17. Le score_gap est calculé avant le pattern CDC → les deux composantes ne se parlent pas.
- Fichiers concernés : engine.py (compute_performance_score) · app.py (affichage)
- Données requises : CNT Antony (Q_MAX_KEY=Q2, decay_corr=0.781, NEGATIVE_SPLIT) · Samuel CDF Long · Coralie Toureille 2023 — min. 3 datasets B7
- Critère de clôture : sur les 6 courses NEGATIVE_SPLIT divergentes, score moteur dans ±10 pts du dataset · Elena valide la logique avant code
- Bloquant : non (scores figés ressenti primes) · priorité haute
- Gel : B7 non satisfait — 1 seul dataset NEGATIVE_SPLIT confirmé (CNT Antony). Dylan ChF = DRIFT-CARDIO, Samuel CDF = COLLAPSE. Requalifié → SCR-MIX1. Déblocage : second GPX NEGATIVE_SPLIT terrain confirmé.

[SCR-ULT1 · CLOS ✅] — Score ULTRA invisible côté athlète
- Problème : duration_ultra=True → affiche "ULTRA" en amber à la place du score numérique. Les 3 sous-scores (GAP/EF/Var) sont calculés mais l'athlète ne voit aucun chiffre global. 4 fichiers concernés : GRV 100M · 24h Ventoux · CCC 2025 · Maxi Race.
- Fichiers concernés : app.py (bloc affichage score VERTEX, condition duration_ultra)
- Données requises : aucune — correction UX pure
- Critère de clôture : score numérique visible + badge ULTRA distinct · sous-scores inchangés · 220/220 tests verts
- Bloquant : non · priorité haute · correction chirurgicale 30 min
- Clôture : score numérique toujours affiché + badge ULTRA · LECTURE PAR COMPOSANTE + note / 100* · 220/220 verts · 30/04/2026

[SCR-EF1 · CLOS ✅] — EF absente sur terrain montagneux malgré FC présente
- Problème : 12 fichiers avec has_hr=True mais score_ef=None. Filtre grade<3% trop restrictif sur ASCENDING/MIXED → plat structurellement rare → ef_source devrait basculer sur GAP_FALLBACK mais ne le fait pas sur ces 12 cas. Perte d'information systématique sur tous les trails montagne.
- Fichiers concernés : engine.py (cardiac_drift · compute_performance_score · fallback SCI-7)
- Données requises : audit_moteur.csv (12 fichiers identifiés) · vérifier ef_source sur chaque cas
- Critère de clôture : score_ef non-None sur au moins 8 des 12 fichiers · Elena valide le seuil de bascule GAP_FALLBACK · B7 ≥2 datasets
- Bloquant : non · investigation préalable requise
- Clôture : _sci7_fallback() élargi à dp_per_km >= 20 · ef_unavailable conditionné dp_per_km < 20.0 (SCI-6 intact sur plat) · nouveau chemin STABLE montagne → score_ef calculé, partial=True, partial_reason distinct · B7 ✅ Jeremy 26km (score_ef=86) + Julien Eynavay (score_ef=88) · 220/220 verts · 30/04/2026

[SCR-MIX1 · CLOS ✅] — apply_decay_correction MIXED trop agressive sur q_max_key = Q3
- Problème : profil MIXED avec sommet en Q3 (trail montée-descente classique) → correction Q4/Q_max donne decay_ratio_corr < 1.0 même quand la course n'est pas dégradée. Le Q3 au sommet est structurellement le quartier le plus lent en allure réelle — la correction interprète la descente Q4 comme une chute d'allure. Issu de l'investigation SCR-NS1.
- Fichiers concernés : engine.py (apply_decay_correction · branche MIXED · q_max_key == Q3)
- Données requises : CNT Antony (q_max=Q2 · decay_corr=0.781) · Dylan ChF (q_max=Q3 · decay_corr=0.870) · Samuel CDF Long (q_max=Q4 · decay_corr=1.0) — B7 satisfait à 3 datasets
- Critère de clôture : decay_ratio_corr cohérent avec ressenti terrain sur les 3 datasets · Elena valide la logique avant code · 220/220 verts
- Bloquant : non · investigation Elena requise avant code
- Clôture : q_max_key ∈ {Q2,Q3} → decay_ratio_corr = Q4/Q1 clipé [0.85,1.20] · decay_mode=Q4/Q1_mix_summit · B7 ✅ CNT Antony(1.20)+Dylan ChF(1.20)+Samuel CDF(1.00 témoin) · G2d mis à jour V4→V2 · 220/220 verts · 30/04/2026

[SCR-ISO1 · CLOS ✅] — Iso-pente invalide sur coureur rapide même parcours
- Problème : iso-pente valide pour Antony CDF Court (2h55) mais invalide pour Dylan CDF Court (2h31) sur parcours identique. Cause : seuil durée ≥3 min par tranche (G4/G8) non atteint sur Q1 ou Q4 quand le coureur est plus rapide. Le seuil est en durée absolue, pas en ratio de temps passé sur la tranche.
- Fichiers concernés : engine.py (_ef_iso_quartile · seuil dur_min < 3.0)
- Données requises : CDF Court Dylan (FLAG F confirmé) · CDF Court Antony (référence valide)
- Critère de clôture : iso-pente disponible pour Dylan CDF Court · seuil revu en % temps tranche ou durée réduite · Elena valide le nouveau seuil · B7 ≥2 datasets
- Bloquant : non · investigation préalable requise
- Clôture : seuil dur_min fixe 3.0min → relatif max(1.0, q_size/60×0.05) · Dylan CDF débloqué · DRIFT-CARDIO révélé · tests G2d/G2e mis à jour · 220/220 verts · 30/04/2026

[CDC-R2 · CLOS ✅]  — Calibration seuils iso-pente cardiac drift
- Problème : ef_iso disponible sur ASCENDING/MIXED mais seuils delta_fc (±4/+5 bpm) non calibrés — 3 datasets collectés, 1 cas limite identifié
- Fichiers concernés : engine.py (cardiac_drift · _ef_iso_quartile · ef_iso_degraded)
- Données requises : 4 datasets collectés ✅ — calibration seuils A2 à faire
- Critère de clôture : seuils delta_fc calibrés sur ≥4 datasets · Elena valide · 220/220 verts
- Bloquant : non
- Datasets :
  · A1 ✅ · CNT Antony · MIXED confirmé · bug DESCENDING non reproductible (version antérieure SCI-8) · ef_iso G4=-14.08% G8=-19.76% · delta_fc G4=-3.07 G8=-4.90 · DRIFT-NEURO confirmé iso
  · A2 ✅ · règle 2 paliers implémentée · seuils delta_fc provisoires DRIFT-CARDIO >+5/+7 bpm · DRIFT-NEURO <-4/-6 bpm · 4 datasets · réévaluation à DRIFT-NEURO #2 · commit f1a9cb9
  · A3 ✅ · Coralie Ventoux CDF · STABLE · MIXED · score=86 · ef_iso G4=-18.12% / G8=-11.01% · ef_iso_degraded=True · delta_fc hors seuil → STABLE maintenu · baseline #4

---
> Source: [antonypellat-alt/Vertex](https://github.com/antonypellat-alt/Vertex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
