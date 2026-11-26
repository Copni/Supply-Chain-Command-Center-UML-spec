# UC03 — Manage Stock

| | |
|---|---|
| **Référence** | [REFERENCE_ISABEY_Noa.pdf](REFERENCE_ISABEY_Noa.pdf). Remarque du prof : « un peu léger pour les alternatives mais pas mal » |

| | |
|---|---|
| **Acteur principal** | Pharmacist / Site Manager |
| **Acteurs secondaires** | Regional Planner, Quality Officer (notifiés) |
| **Relations** | `<<include>>` UC14 View Site Stock ; étendu par UC1 Manage Replenishment (stock < seuil) |
| **Déclencheur** | Réception d'une livraison, délivrance de médicaments, inventaire, casse… |
| **Préconditions** | Pharmacien authentifié et rattaché à un site ; le produit et le lot existent |
| **Garantie minimale** | Le stock ne change pas en cas d'échec ; il n'est jamais négatif ; un lot expiré ne sort jamais |
| **Garantie de succès** | Stock à jour et visible sur tous les canaux (pharmacie et en ligne) ; alerte créée si le stock passe sous le seuil |

## Scénario nominal

1. Le pharmacien ouvre la page de gestion du stock.
2. Le système affiche le stock du site (**include UC14**).
3. Le pharmacien choisit un produit et un type de mouvement (IN, OUT ou ADJUSTMENT).
4. Le système affiche les lots du produit en ordre **FEFO**.
5. Le pharmacien saisit le lot, la quantité et le motif.
6. Le système vérifie que le lot existe, qu'il n'est pas expiré et que la quantité est possible.
7. Le système affiche un récapitulatif.
8. Le pharmacien confirme.
9. Le système met à jour `Stock.quantity` et crée un `StockMovement`.
10. Le système compare le stock au seuil.
11. Le système affiche le stock mis à jour.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| 6a | Lot inexistant | Erreur, retour à l'étape 5 |
| 6b | Quantité supérieure au stock | Erreur, retour à l'étape 5 |
| 6c | Lot expiré | Mouvement refusé et Quality Officer notifié |
| 8a | Annulation | Rien n'est modifié, retour à l'étape 2 |
| 10a | Stock < seuil | Alerte LOW_STOCK (pharmacien et planner) et `Order(AUTOMATIC)` créée (**extend UC1**) |
| 10b | Lot proche de la péremption | Alerte NEAR_EXPIRY |
| * | Erreur réseau ou base | Opération annulée, stock inchangé |

> Ajouter aussi un refus si le lot est QUARANTINED ou RECALLED (invariant 2 de [MODELE_DOMAINE.md](../00_Global/MODELE_DOMAINE.md)).

## MVC et classes

- **MVC :** `StockView` → `StockController` → `Stock`, `Batch`, `Product`, `StockMovement`, `Alert`, `Order`
- **Écrans :** voir la section 5 du PDF (formulaire de mouvement, récapitulatif, erreur).
