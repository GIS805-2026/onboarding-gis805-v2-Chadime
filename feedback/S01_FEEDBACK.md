# Rétroaction automatisée -- S01 (Diagnostic fondamental -- NexaMart kickoff)

_Générée le 2026-05-14T23:56:38+00:00 -- Run `20260514T235532Z-d8b8f471`_

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

> Bon diagnostic opérationnel: le brief identifie des catégories et régions en déclin et propose la construction d'un entrepôt. Pour passer à l'excellent, formalisez le traitement des changements historiques, automatisez les checks de qualité et ajoutez un log de processus reproducible.

### Observations par dimension

**Model quality**
- Observation : Vous déclarez clairement le grain (« une ligne par ligne de vente transactionnelle »), listez mesures et dimensions et fournissez un schéma d'analyse par catégorie et région.
- Piste d'amélioration : Précisez le traitement des changements historiques (SCD) et notez les attributs non-additifs (ex. unit_price) avec règles de calcul explicites.

**Validation quality**
- Observation : Vous fournissez des requêtes SQL de validation et indiquez avoir réconcilié les agrégats avec raw_fact_sales.
- Piste d'amélioration : Ajoutez des checks automatisés (NULLs, doublons du grain, SUM(quantity×unit_price) vs SUM(line_total)) et couvrez les cas limites dans la requête.

**Executive justification**
- Observation : La section 'Réponse exécutive' énonce catégories et régions en déclin et la recommandation de créer un entrepôt Kimball.
- Piste d'amélioration : Formulez une recommandation décisionnelle claire avec impact chiffré (ex. pertes projetées et actions prioritaires) en langage non technique.

**Process trace**
- Observation : Le brief ne mentionne ni historique de commits git ni note d'usage d'outil IA ou journal de décisions.
- Piste d'amélioration : Incluez un log de commits (≥3) avec messages significatifs et une note IA décrivant outils et validation humaine.

**Reproducibility**
- Observation : Les requêtes sont fournies mais contiennent des dates redacted et des jointures (géographie via city) qui peuvent empêcher une exécution reproducible sans contexte.
- Piste d'amélioration : Fournissez un README + scripts exécutables (ex. DuckDB), jeux de dates d'exemple non redacted et retirez chemins codés en dur pour permettre un clone→run.

## 3. Déclaration d'utilisation de l'IA

> La déclaration est détaillée et documente clairement les interactions (prompts, requêtes SQL et vérifications). Cependant l'outil est nommé sans précision de version/modèle technique plus fine, ce qui rend la partie « version » générique.

**Sujets bien couverts dans votre déclaration :**

- outils utilisés (nom + version/modèle)
- à quelle étape l'IA a été utilisée
- comment la sortie a été validée par l'humain
- limites ou erreurs observées

## 4. Pistes d'action pour la prochaine itération

- Reprendre la requête de la section « Preuve » pour qu'elle s'exécute sur `db/nexamart.duckdb` et qu'elle produise la forme attendue (voir pistes en section 1).

---

## 5. Traçabilité

- **Run ID :** `20260514T235532Z-d8b8f471`
- **Devoir :** `S01`
- **Étudiant·e :** `Chadime`
- **Commit analysé :** `1a628f9`
- **Audit (côté instructeur) :** `tools/instructor/feedback_pipeline/audit/20260514T235532Z-d8b8f471/Chadime/`
- **Prompts (SHA-256) :**
  - `sql_extractor_system` : `90ee9e277de7a27f...`
  - `rubric_grader_system` : `505f32d1d8319d66...`
  - `ai_usage_grader_system` : `81cb7fdf89bda55a...`
- **Fournisseur (rubrique) :** `openai`
- **Fournisseur (IA-usage) :** `openai` (gpt-5-mini-2025-08-07)

_Ce feedback a été produit par un pipeline automatisé et **revu par l'équipe pédagogique avant publication**. Aucun chiffre ni étiquette de niveau n'est diffusé à ce stade expérimental : l'objectif est uniquement formatif. Ouvrez une issue dans ce dépôt pour toute question._
