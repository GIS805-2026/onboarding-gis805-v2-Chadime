# Board Brief — S01

## Question du CEO

Quelles catégories de produits déclinent? Dans quelles régions, et pourquoi?

## Réponse exécutive

L’analyse des ventes montre que certaines catégories, notamment Automotive et Pet Supplies, présentent un recul progressif dans des régions comme l’Ontario et le Québec. Le déclin semble corrélé à trois facteurs principaux : une diminution des promotions appliquées, une baisse du nombre de clients actifs et une augmentation des prix moyens. Les données disponibles permettent d’identifier les tendances régionales et temporelles avec précision, mais les causes demeurent des hypothèses analytiques plutôt que des causalités démontrées. Une relance promotionnelle ciblée et un suivi plus détaillé des comportements clients sont recommandés.

## Décisions de modélisation
Le grain choisi pour la table des faits est:
Une ligne par ligne de vente transactionnelle

-Les principales mesures utilisées sont:
line_total, quantity, COUNT(DISTINCT customer_id), AVG(unit_price), AVG(discount_pct)

Les principales dimensions mobilisées dans l'analyse sont:

dim_product, dim_store, dim_date, dim_customer, dim_promotion

L’analyse repose sur plusieurs hypothèses :

-une baisse des promotions influence négativement les ventes
-une hausse du prix moyen peut réduire la demande
-une diminution du nombre de clients actifs contribue au déclin observé.

Ces hypothèses reposent sur des corrélations observées dans les données.


## Preuve
Requête pour avoir le nombre de tables dans la db:
duckdb /workspaces/onboarding-gis805-v2-Chadime/db/nexamart.duckdb "SELECT table_name FROM information_schema.tables WHERE table_schema = 'main' ORDER BY table_name;"

Requête pour voir le revenue par catégorie et par région:

SELECT
  p.category,
  s.region,
  SUM(f.line_total) AS total_revenue
FROM raw_fact_sales f
JOIN raw_dim_product p ON f.product_id = p.product_id
JOIN raw_dim_store s ON f.store_id = s.store_id
-- TODO(étudiant): Vérifie que les JOINs préservent le grain de la fact table
-- (1 ligne de fact_sales = combien de lignes après les JOINs ?)
GROUP BY p.category, s.region
ORDER BY p.category, s.region;

Requêtre pour localiser le déclin:

SELECT
  p.category,
  p.product_name,
  s.region,
  DATE_TRUNC('month', f.order_date) AS mois,
  SUM(f.line_total) AS revenue
FROM fact_sales f
JOIN dim_product p ON f.product_id = p.product_id
JOIN dim_store s ON f.store_id = s.store_id
WHERE p.category = 'Automotive' AND s.region = 'Ontario'
GROUP BY p.category, p.product_name, s.region, mois
ORDER BY mois DESC, revenue DESC;

Toutes les autres requêtes que j'ai effectué peuvent être retrouver dans la section ai-usage.md




## Validation
Les validations suivantes ont été réalisées :

Vérification que les jointures ne créent pas de multiplicité dans la table de faits.

Réconciliation des agrégats avec les totaux de raw_fact_sales.

Validation de l’additivité des mesures (line_total, quantity).

Contrôle du nombre attendu de lignes après agrégation.

Comparaison des tendances entre revenus, quantités vendues et nombre de clients.


## Risques / limites
Le modèle présente plusieurs limites importantes :

-absence de données détaillées sur les campagnes marketing ;
-absence d’historique complet des changements de prix ;
-aucune donnée concurrentielle ou économique externe ;
-absence d’information sur les ruptures de stock ;
-impossibilité de prouver une causalité réelle entre promotions et ventes.


## Prochaine recommandation
Ce que je recommande, pour pouvoir répondre à la question du CEO, est de débuter la création de l'entrepôt de données selon la méthode Kimball. De manière à ce que nos données soient centralisées et structurées.
