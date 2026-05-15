# Rétroaction automatisée -- S01 (Diagnostic fondamental -- NexaMart kickoff)

_Générée le 2026-05-15T12:29:28+00:00 -- Run `20260515T122624Z-00a5a04f`_

Ce document est produit par un pipeline reproductible (vérification SQL déterministe + analyse LLM du brief et de la déclaration IA). Une revue humaine précède toujours sa publication. **À ce stade expérimental, aucune note ni étiquette de niveau n'est diffusée : l'objectif est purement formatif.**

> ⚠️ **Avertissement instructeur (à retirer avant publication) :** cette analyse a été générée avec `--skip-pull`. Le contenu correspond au commit local et **n'est peut-être pas la dernière version poussée par l'étudiant·e**.

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
> Tables référencées dans votre requête mais absentes de la base : `monthly`, `pvt`.
> Tables disponibles dans `db/nexamart.duckdb` : `raw_bridge_campaign_allocation`, `raw_bridge_customer_segment`, `raw_customer_changes`, `raw_customer_profile_bands`, `raw_customer_scd3_history`, `raw_dim_channel`, `raw_dim_customer`, `raw_dim_date`, `raw_dim_geography`, `raw_dim_product`, `raw_dim_segment_outrigger`, `raw_dim_store`, `raw_fact_budget`, `raw_fact_daily_inventory`, `raw_fact_inventory_snapshot`, `raw_fact_order_pipeline`, `raw_fact_orders_transaction`, `raw_fact_promo_exposure`, `raw_fact_returns`, `raw_fact_sales`.

## 2. Rétroaction pédagogique sur le brief

> Le brief répond directement à la question CEO avec chiffres et un grain explicite, et propose une recommandation opérationnelle (Kimball). Il manque toutefois la traçabilité (commits, note IA) et des contrôles reproductibles formels pour mettre ce diagnostic en production.

### Observations par dimension

**Model quality**
- Observation : « Le grain choisi pour la table des faits est: Une ligne par ligne de vente transactionnelle » et la brief liste dim_product, dim_store, dim_date, dim_customer, dim_promotion.
- Piste d'amélioration : Documenter les patterns SCD (ex. SCD Type 2), expliciter le traitement de unit_price non-additif et fournir un DDL minimal (CREATE TABLE) pour le schéma en étoile.

**Validation quality**
- Observation : La section Preuve inclut une requête avec agrégation mensuelle et une seconde pour les retours, et la section Validation mentionne réconciliation des agrégats.
- Piste d'amélioration : Ajouter des checks reproductibles traitant les NULLs, contrôler les doublons sur le grain et inclure des cas limites (p.ex. mois sans ventes, SUM(weight)=1) dans un script 'make check'.

**Executive justification**
- Observation : La Réponse exécutive donne des chiffres précis de déclin par catégorie et région et la Prochaine recommandation propose de créer un entrepôt selon Kimball.
- Piste d'amélioration : Raccourcir en langage décisionnel (150–300 mots), expliciter l'action demandée au CEO (p.ex. valider le budget/plan S02) et ajouter l'impact attendu en KPIs.

**Process trace**
- Observation : Aucune mention d'un historique de commits git, d'une note IA ou d'un decision log dans le brief.
- Piste d'amélioration : Fournir l'historique git (≥3 commits significatifs), documenter l'usage d'outils IA si utilisés et ajouter un court decision log sur les choix de modélisation.

**Reproducibility**
- Observation : Les requêtes contiennent des placeholders '[REDACTED-PHONE]' et il n'y a pas d'instructions de reproduction claires.
- Piste d'amélioration : Inclure un README et un script de test (ex. DuckDB + make check) sans chemins codés en dur et remplacer les placeholders par des jeux de données d'exemple pour cloner et reproduire.

## 3. Déclaration d'utilisation de l'IA

> La déclaration documente clairement l'utilisation de GitHub Copilot Chat, les étapes d'interaction et plusieurs démarches de validation humaine. Indiquez toutefois la version du modèle (ou sa configuration) et précisez davantage la méthode de validation humaine pour obtenir la note maximale.

**Sujets bien couverts dans votre déclaration :**

- outils utilisés (nom + version/modèle)
- à quelle étape l'IA a été utilisée
- comment la sortie a été validée par l'humain
- limites ou erreurs observées

## 4. Pistes d'action pour la prochaine itération

- Reprendre la requête de la section « Preuve » pour qu'elle s'exécute sur `db/nexamart.duckdb` et qu'elle produise la forme attendue (voir pistes en section 1).

---

## 5. Traçabilité

- **Run ID :** `20260515T122624Z-00a5a04f`
- **Devoir :** `S01`
- **Étudiant·e :** `Chadime`
- **Commit analysé :** `51d903f`
- **Audit (côté instructeur) :** `tools/instructor/feedback_pipeline/audit/20260515T122624Z-00a5a04f/Chadime/`
- **Prompts (SHA-256) :**
  - `sql_extractor_system` : `90ee9e277de7a27f...`
  - `rubric_grader_system` : `505f32d1d8319d66...`
  - `ai_usage_grader_system` : `81cb7fdf89bda55a...`
- **Fournisseur (rubrique) :** `openai`
- **Fournisseur (IA-usage) :** `openai` (gpt-5-mini-2025-08-07)

_Ce feedback a été produit par un pipeline automatisé et **revu par l'équipe pédagogique avant publication**. Aucun chiffre ni étiquette de niveau n'est diffusé à ce stade expérimental : l'objectif est uniquement formatif. Ouvrez une issue dans ce dépôt pour toute question._
