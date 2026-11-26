# UC01 — Manage Replenishment

> ⚠️ **Aucun livrable.** Ce UC n'a été attribué à personne (KAMOUN Omar n'avait pas de UC et était absent).
> La spécification ci-dessous est une **proposition**. Elle est déduite de ce que les autres UC attendent de UC1 : UC3 (extension sur stock bas), UC15 (traitement des demandes exceptionnelles), UC7 (commandes « à expédier ») et le contexte MOGUO (« suivre et valider les commandes juste à temps des sites de sa région, FEFO »).

| | |
|---|---|
| **Acteur principal** | Regional Planner |
| **Acteurs secondaires** | Pharmacist / Site Manager (notifié de la décision) |
| **Relations** | `<<extend>>` Manage Stock (UC3) quand le stock passe sous le seuil |
| **Déclencheur** | Une alerte LOW_STOCK (UC3), une demande exceptionnelle soumise (UC15), ou l'ouverture de la page « Replenishment » |
| **Préconditions** | Le planner est authentifié et rattaché à une `Region` |
| **Garantie minimale** | Aucune commande n'est modifiée sans validation explicite ; tout est journalisé |
| **Garantie de succès** | La commande passe à VALIDATED (et devient visible dans UC7) ou à REJECTED avec un motif ; le site est notifié |

## Scénario nominal (proposé)

1. Le planner ouvre « Replenishment ».
2. Le système liste les `Order` en statut PENDING des sites de sa région, AUTOMATIC et EXCEPTIONAL, triées par urgence puis par date souhaitée.
3. Le planner sélectionne une commande.
4. Le système affiche les lignes, le stock du site, le stock disponible au DC régional (lots en ordre FEFO) et le motif ou l'urgence.
5. Le planner ajuste éventuellement les quantités, puis valide.
6. Le système vérifie la disponibilité au DC (FEFO, lots non bloqués), passe la commande à VALIDATED et journalise.
7. Le système notifie le pharmacien du site. La commande apparaît dans UC7.

## Variantes et erreurs

- **Création automatique** (côté système, déclenchée par UC3) : quand `available < Product.reorderThreshold` et qu'aucune commande ouverte n'existe déjà pour ce produit et ce site, le système crée une `Order(AUTOMATIC, PENDING)` avec une quantité calculée.
- **5a. Rejet** : un motif est obligatoire ; la commande passe à REJECTED et le pharmacien est notifié.
- **6a. Stock DC insuffisant** : proposer une validation partielle (`partial = true`) ou laisser la commande en attente.
- **6b. Produit concerné par un rappel** : la ligne est refusée.
- **Erreurs standard** : hors région (accès refusé), conflit de version, base indisponible.

## Classes

`Order`, `OrderLine`, `Product`, `Batch`, `Stock`, `Site`, `Region`, `Alert`, `AuditLog`

## À trancher en équipe

- La formule de quantité automatique (seuil × k, ou couverture de N jours).
- Une commande VALIDATED doit-elle déclencher directement une `Recommendation` (UC7 / UC13) ?
