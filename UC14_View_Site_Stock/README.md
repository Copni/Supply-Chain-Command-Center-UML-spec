# UC14 — View Site Stock

| | |
|---|---|
| **Référence** | [REFERENCE_CHEAV_Khemra_ISABEY_Noa.pdf](REFERENCE_CHEAV_Khemra_ISABEY_Noa.pdf) (9 p.). Remarque du prof : « manque les stéréotypes, mais scénario alternatif très intéressant » |
| **Autre version** | [alt_DUVAL_Kelvish/](alt_DUVAL_Kelvish/) : spec d'une page, IHM (4 états), diagramme de classes, diagramme de séquence. Remarque du prof : « un peu léger ». |

| | |
|---|---|
| **Acteur principal** | Pharmacist / Site Manager (son propre site) |
| **Relations** | **inclus** par UC3 Manage Stock |
| **Préconditions** | Connecté ; compte rattaché au site ; stock synchronisé au moins une fois |
| **Garantie minimale** | Aucune modification ; rien n'est exposé en cas d'échec |
| **Garantie de succès** | Stock par lot : quantités physique, réservée et disponible, statut ; tri FEFO ; livraisons entrantes ; heure de dernière synchronisation |

## Scénario nominal

1. L'acteur ouvre « Site Stock ».
2. Le système vérifie l'accès au site.
3. Le système rassemble le stock par lot, les lots et produits, les réservations en ligne actives et les livraisons entrantes.
4. Le système calcule la quantité disponible et le statut de chaque lot, puis trie par date de péremption (FEFO).
5. Le système affiche la synthèse (nombre de lots, unités disponibles, lots bientôt expirés, expirés, en stock bas), le tableau, les livraisons entrantes et l'heure de dernière synchronisation.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| A1 | Accès refusé | « You are not authorised to view this site » ; tentative journalisée |
| A2 | Aucun stock | « No stock recorded for this site » (les livraisons entrantes restent affichées) |
| A3 | Source indisponible ou timeout | « Stock temporarily unavailable » et bouton « Retry » ; **aucune donnée partielle** |
| A4 | Données anciennes | Avertissement « Data last updated at hh:mm » |
| A5 | Recherche ou filtre (produit, statut, chaîne du froid) | Aucun résultat → « No result for this filter » et « Reset filters » |

## Règles métier (à coder telles quelles)

- **BR1 :** `available = max(0, quantity − reservedQuantity)`. Un lot expiré ou en quarantaine a une quantité disponible de 0.
- **BR2 :** tri FEFO (péremption la plus proche en premier).
- **BR3 :** statut d'un lot, par priorité décroissante :
  1. **Quarantined** : lot QUARANTINED ou RECALLED ;
  2. **Expired** : date de péremption dépassée ;
  3. **Expiring soon** : péremption dans moins de N jours (paramètre, par exemple 30) ;
  4. **Low stock** : disponible < `Product.reorderThreshold` ;
  5. **In stock**.
- **BR4 :** aucune donnée patient ; les réservations en ligne n'apparaissent que sous forme de quantités agrégées.
- **BR5 :** le pharmacien ne voit que son site.
- Les seuils (fraîcheur, péremption, réapprovisionnement) sont **configurables**.

## MVC et classes

`SiteStockView` → `SiteStockController` → `User`, `Site` (`lastSyncAt`), `Stock`, `Batch`, `Product`, `Order` (réservations), `Shipment`
