# 🍦 Prévision des ventes de glaces : séries temporelles (MA / ARMA)

Combien de bacs de glace sortir du congélateur demain ? Ce projet répond à cette question à partir d'un simple carnet de ventes journalières, en utilisant les modèles classiques de séries temporelles **MA** et **ARMA** avec `statsmodels`.

> Projet n°6 du **Cahier de vacances 2026** de [Machine Learnia](https://github.com/MachineLearnia/Cahier-Vacances-2026). Le code des exercices est le mien ; l'énoncé, le jeu de données et `utils.py` viennent du cahier (voir [Crédits](#crédits)).

## Problème

Bruno, marchand de glaces sur la plage, ne sait jamais combien de clients il aura (entre 116 et 278 glaces par jour selon les jours). Trop peu de stock et il refuse des clients, trop de stock et il jette.

- **Données** : ventes journalières du 1er juin au 30 septembre, étés 2022 à 2025 (488 jours, 122 jours par saison), dans `data/ventes_glaces.csv`
- **Objectif** : prévoir les ventes du lendemain, puis des 7 prochains jours
- **Métrique** : MAE (erreur absolue moyenne, en nombre de glaces)
- **Découpage** : les 22 derniers jours (fin de l'été 2025) servent de test. Aucun mélange aléatoire, pour ne pas « lire l'avenir ».

## Ce que fait le notebook

1. **Exploration** : chargement avec `parse_dates`, statistiques par saison, visualisation de la série
2. **Baselines** : « toujours la moyenne » et « comme hier »
3. **Autocorrélation** : implémentation à la main de l'ACF, vérifiée contre `statsmodels`, lecture des corrélogrammes ACF / PACF
4. **Modèle MA(q)** : simulation d'une série MA à partir de chocs aléatoires, puis ajustement d'un MA(3) avec `ARIMA(order=(0, 0, 3))`
5. **Modèle ARMA(1,1)** : comparaison avec le MA(3) par l'AIC et le test de Ljung-Box sur les résidus
6. **Prévision à 7 jours** : `forecast(steps=7)`, intervalles de confiance et limites de l'horizon

## Résultats

| Approche | MAE (glaces / jour) |
|---|---|
| Toujours la moyenne | 30,38 |
| Comme hier | 19,00 |
| MA(3), ARMA(1,1) | voir le tableau des scores en fin de partie 4 du notebook |

Points clés :

- Une journée ressemble fortement à la veille : autocorrélation de **0,81** au retard 1, **0,52** au retard 2
- Sur 122 jours, l'ACF semble s'arrêter au retard 3 (piste MA(3)). Sur 488 jours, la mémoire décroît lentement, ce qui justifie un terme AR.
- L'**ARMA(1,1)** obtient un meilleur AIC que le MA(3) avec moins de coefficients, et laisse des résidus sans structure
- Un MA(q) devient plat après q jours de prévision, alors que l'ARMA revient progressivement vers la moyenne
- Prévoir la veille pour le lendemain fonctionne ; à 7 jours, l'intervalle de confiance couvre presque toute la plage des ventes habituelles

## Structure

```
.
├── Projet_06.ipynb      # notebook complet
├── utils.py             # fonctions d'affichage et métriques (fournies par le cahier)
├── data/
│   └── ventes_glaces.csv
├── projet_06.png        # illustration du notebook
├── requirements.txt
└── README.md
```

## Installation et lancement

```bash
git clone https://github.com/Hamed-Hafaiedh/prevision-ventes-glaces.git
cd prevision-ventes-glaces

python -m venv .venv
# Windows : .venv\Scripts\activate    |    Linux / macOS : source .venv/bin/activate
pip install -r requirements.txt

jupyter notebook Projet_06.ipynb
```

Avec `uv` : `uv venv && uv pip install -r requirements.txt`.

Le projet a été développé avec Python 3.13.

## Pour aller plus loin

- Ajouter la saisonnalité hebdomadaire avec un **SARIMA** (`seasonal_order=(P, D, Q, 7)`)
- Ajouter la température comme variable explicative avec **SARIMAX** (`exog=`)
- Comparer avec un modèle de Machine Learning (`HistGradientBoostingRegressor`) nourri de variables calendaires et de retards

## Crédits

Ce projet est tiré du **Cahier de vacances 2026** de [Machine Learnia](https://github.com/MachineLearnia/Cahier-Vacances-2026). L'énoncé, le scénario, le jeu de données, les illustrations et `utils.py` sont issus de ce cahier et appartiennent à leur auteur.