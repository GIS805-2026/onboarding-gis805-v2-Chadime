# Board Brief — S01

## Question du CEO

Quelles catégories de produits déclinent? Dans quelles régions, et pourquoi?

## Réponse exécutive

En prenant les deix derniers mois de l'année 2025, l’analyse des ventes montre que certaines catégories, notamment automotives, Pet Supplies et Books&Media on connus les plus gros déclins entre les deux derniers mois (-1897.52$, -1667.05$, -1383.66$). Ces déclins concernent dans l'ordre, les régions de l'Ontario, du Québec et de la Colombie-britannique.
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
Requête que j'ai utilisé pour estimer les revenus mensuels bruts par catégorie et région:

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
    WHERE s.order_date >= '2025-11-01' AND s.order_date < '2026-01-01'
    GROUP BY 1, 2, 3
),
pvt AS (
    SELECT category, region,
        MAX(CASE WHEN mth = '2025-11-01' THEN revenue ELSE 0 END) as nov_2025,
        MAX(CASE WHEN mth = '2025-12-01' THEN revenue ELSE 0 END) as dec_2025
    FROM monthly GROUP BY 1, 2
)
SELECT *, ROUND(dec_2025 - nov_2025, 2) as decline_dollars
FROM pvt
WHERE nov_2025 > 0 AND dec_2025 < nov_2025
ORDER BY decline_dollars ASC;

Requête que j'ai utilisé, pour agréger par régions, mois et catégorie de produits:

SELECT p.category, g.region,
    DATE_TRUNC('month', r.return_date) as month,
    r.return_reason,
    COUNT(*) as nb_returns,
    ROUND(SUM(r.refund_amount), 2) as total_refund
FROM raw_fact_returns r
JOIN raw_dim_product p ON r.product_id = p.product_id
JOIN raw_dim_store st ON r.store_id = st.store_id
JOIN raw_dim_geography g ON st.city = g.city
WHERE r.return_date >= '2025-11-01' AND r.return_date < '2026-01-01'
GROUP BY 1, 2, 3, 4
ORDER BY 1, 2, 3;

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
