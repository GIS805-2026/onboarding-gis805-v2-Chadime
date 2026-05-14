# Rétroaction automatisée -- S01 (Diagnostic fondamental -- NexaMart kickoff)

_Générée le 2026-05-14T22:16:27+00:00 -- Run `20260514T221333Z-7d34bf6a`_

Ce document est produit par un pipeline reproductible (vérification SQL déterministe + analyse LLM du brief et de la déclaration IA). Une revue humaine précède toujours sa publication. **À ce stade expérimental, aucune note ni étiquette de niveau n'est diffusée : l'objectif est purement formatif.**

---

## 1. Vérification automatique de la requête SQL

La requête extraite de votre brief n'a pas pu être validée automatiquement. Quelques pistes constructives ci-dessous pour vous aider à la rendre exécutable et alignee avec la question posée.

_Observation technique : erreur d'exécution SQL: Conversion Error: invalid date field format: "[REDACTED-PHONE]", expected format is (YYYY-MM-DD)_

<details><summary>Requête analysée — cliquez pour déplier</summary>

```sql
WITH monthly AS (
    SELECT
        p.category,
        g.region,
        DATE_TRUNC('month', s.order_date) as mth,
        ROUND(SUM(s.line_total), 2) as revenue
    FROM raw_fact_sales s
    JOIN raw_dim_product p ON s.product_id = p.product_id
    JOIN raw_dim_store st ON s.store_id = st.store_id
    JOIN raw_dim_geography g ON st.city = g.city
    WHERE s.order_date >= '[REDACTED-PHONE]' AND s.order_date < '[REDACTED-PHONE]'
    GROUP BY 1, 2, 3
),
pvt AS (
    SELECT category, region,
        MAX(CASE WHEN mth = '[REDACTED-PHONE]' THEN revenue ELSE 0 END) as nov_2025,
        MAX(CASE WHEN mth = '[REDACTED-PHONE]' THEN revenue ELSE 0 END) as dec_2025
    FROM monthly GROUP BY 1, 2
)
SELECT *, ROUND(dec_2025 - nov_2025, 2) as decline_dollars
FROM pvt
WHERE nov_2025 > 0 AND dec_2025 < nov_2025
ORDER BY decline_dollars ASC;
```

</details>


**Pistes :**
> Requête extraite par LLM (aucun bloc fencé détecté). Encadrez votre requête finale par ```sql ... ``` pour éliminer toute ambiguïté.
> Votre `db/nexamart.duckdb` est absente ou vide ; la requête a été exécutée contre une **base de référence cohorte** (seed instructeur). Les chiffres retournés ne correspondent donc pas à vos propres données : reconstruisez votre base avec `python src/run_pipeline.py` (ou `.\run.ps1 load`) pour valider vos calculs sur votre seed personnel.
> Tables référencées dans votre requête mais absentes de la base : `monthly`, `pvt`.
> Tables disponibles dans `db/nexamart.duckdb` : `raw_bridge_campaign_allocation`, `raw_bridge_customer_segment`, `raw_customer_changes`, `raw_customer_profile_bands`, `raw_customer_scd3_history`, `raw_dim_channel`, `raw_dim_customer`, `raw_dim_date`, `raw_dim_geography`, `raw_dim_product`, `raw_dim_segment_outrigger`, `raw_dim_store`, `raw_fact_budget`, `raw_fact_daily_inventory`, `raw_fact_inventory_snapshot`, `raw_fact_order_pipeline`, `raw_fact_orders_transaction`, `raw_fact_promo_exposure`, `raw_fact_returns`, `raw_fact_sales`.

## 2. Rétroaction pédagogique sur le brief

> Le brief identifie un grain clair, produit des requêtes reproductibles et donne des résultats chiffrés par catégorie et région; cependant il manque la traçabilité (commits et note IA) et des contrôles détaillés pour les cas limites. Pour améliorer, documentez les décisions SCD, fournissez checks automatisés et un README reproduisant les analyses.

### Observations par dimension

**Model quality**
- Observation : Le brief indique clairement le grain («Une ligne par ligne de vente transactionnelle»), les dimensions et mesures utilisées, et donne des résultats chiffrés par catégorie et région.
- Piste d'amélioration : Préciser le pattern SCD et justifier le choix (ex. SCD Type 2 pour catégories), mentionner les attributs clés des dimensions et corriger l'utilisation de mesures non-additives (unit_price).

**Validation quality**
- Observation : L'étudiant fournit deux requêtes (agrégation monthly et retours) et décrit des contrôles (réconciliation des agrégats, vérification des jointures).
- Piste d'amélioration : Ajouter des checks reproductibles traitant les cas limites (NULLs, doublons grain, SUM(quantity×unit_price) versus SUM(unit_price)) et montrer des sorties d'exécution ou counts attendus.

**Executive justification**
- Observation : Le brief répond à la question en donnant catégories/régions en déclin et recommande de «débuter la création de l'entrepôt de données selon la méthode Kimball».
- Piste d'amélioration : Formuler une recommandation décisionnelle plus concrète (par ex. actions prioritaires, impact financier estimé et next steps en 30/60/90 jours) et réduire le jargon technique.

**Process trace**
- Observation : Aucune mention d'un historique de commits git, ni de note sur l'utilisation d'outils IA ou d'un decision log.
- Piste d'amélioration : Ajouter un log de commits (≥3 commits significatifs), une note IA précisant outil/usage et valider manuellement les décisions importantes.

**Reproducibility**
- Observation : Le brief inclut les requêtes SQL mais utilise des intervalles de date redacted et des jointures sur raw_dim_geography sans README ni script d'exécution.
- Piste d'amélioration : Fournir un README + script 'make check' avec chemins relatifs, paramètres d'entrée (dates) et étapes pour reproduire les checks sur un clone propre.

## 3. Déclaration d'utilisation de l'IA

> La déclaration documente bien les interactions (outil nommé, étapes d'utilisation, validations et limites observées). Toutefois, la mention de l'outil manque d'un niveau de précision attendu (version/modèle détaillé) et certains éléments de validation restent un peu génériques.

**Sujets bien couverts dans votre déclaration :**

- outils utilisés (nom + version/modèle)
- à quelle étape l'IA a été utilisée
- comment la sortie a été validée par l'humain
- limites ou erreurs observées

## 4. Pistes d'action pour la prochaine itération

- Reprendre la requête de la section « Preuve » pour qu'elle s'exécute sur `db/nexamart.duckdb` et qu'elle produise la forme attendue (voir pistes en section 1).

---

## 5. Traçabilité

- **Run ID :** `20260514T221333Z-7d34bf6a`
- **Devoir :** `S01`
- **Étudiant·e :** `Chadime`
- **Commit analysé :** `60e6e26`
- **Audit (côté instructeur) :** `tools/instructor/feedback_pipeline/audit/20260514T221333Z-7d34bf6a/Chadime/`
- **Prompts (SHA-256) :**
  - `sql_extractor_system` : `90ee9e277de7a27f...`
  - `rubric_grader_system` : `505f32d1d8319d66...`
  - `ai_usage_grader_system` : `81cb7fdf89bda55a...`
- **Fournisseur (rubrique) :** `openai`
- **Fournisseur (IA-usage) :** `openai` (gpt-5-mini-2025-08-07)

_Ce feedback a été produit par un pipeline automatisé et **revu par l'équipe pédagogique avant publication**. Aucun chiffre ni étiquette de niveau n'est diffusé à ce stade expérimental : l'objectif est uniquement formatif. Ouvrez une issue dans ce dépôt pour toute question._
