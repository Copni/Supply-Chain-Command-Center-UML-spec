# UC08 — View Dashboard

| | |
|---|---|
| **Référence** | [REFERENCE_YERIMA_Jean-Charles.pdf](REFERENCE_YERIMA_Jean-Charles.pdf) |
| ⚠️ **Numérotation** | Le fichier d'origine s'appelait « 1 – View dashboard », mais View Dashboard est le **UC8** sur le tableau et dans la liste du prof (le UC1 est Manage Replenishment). |

| | |
|---|---|
| **Acteur principal** | Administrator |
| **Déclencheur** | Connexion de l'admin, ou clic sur le menu Dashboard |
| **Préconditions** | Admin authentifié, rôle autorisé ; données réseau existantes |
| **Garanties** | **Lecture seule** : aucune donnée n'est modifiée, en cas de succès comme d'échec |

## Scénario nominal

1. L'admin demande le dashboard.
2. Le système vérifie le rôle.
3. Le système récupère les niveaux de stock par site.
4. Le système récupère les expéditions en cours.
5. Le système récupère les alertes et rappels actifs.
6. Le système calcule les KPI (`computeKPIs()`, dans le Controller).
7. Le système affiche le dashboard.
8. L'admin consulte.

## Alternatives et exceptions

| Id | Situation | Réaction |
|---|---|---|
| A1 | Filtres (région, site, période, produit) | Recalcul ; boucle depuis l'étape 8 |
| A2 | Rupture de stock ou rappel actif | Mis en évidence en haut du dashboard |
| A3 | Aucune donnée | « No data available » |
| E1 | Accès refusé | Message d'erreur |
| E2 | Base de données indisponible | Message d'erreur et bouton « Retry » |

## Écran (HMI)

- Barre de filtres
- 4 cartes KPI
- Tableau « Critical alerts » en tête
- Tableau « Stock level per site » (stock comparé au seuil)
- États vide et erreur

## MVC et classes

- **MVC :** `DashboardView` → `DashboardController` → `User`, `Role`, `Site`, `Stock`, `Product`, `Shipment`, `Alert` (et `Recall`)
- **Conseil :** implémenter ce UC **en dernier** ; il agrège les données des autres UC.
