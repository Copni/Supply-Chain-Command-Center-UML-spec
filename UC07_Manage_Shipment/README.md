# UC07 — Manage Shipment

| | |
|---|---|
| **Référence** | [REFERENCE_CAVAIGNAC_Romain.docx](REFERENCE_CAVAIGNAC_Romain.docx) (« BEST OF ») |
| **Diagramme** | [diagrams/SD_manage_shipment.png](diagrams/SD_manage_shipment.png) |
| **Écrans** | [hmi/](hmi/) : liste, planification, récapitulatif, erreur chaîne du froid. Images extraites du .docx. |
| **Autre version** | [alt_TUSSEAU_Marin.pdf](alt_TUSSEAU_Marin.pdf). ⚠️ Il écrit `<<include>>` UC13 ; le diagramme global retient `<<extend>>`. |

| | |
|---|---|
| **Acteur principal** | Regional Planner |
| **Acteurs secondaires** | Pharmacist (site de destination), Supplier / Carrier Partner |
| **Relations** | étendu par UC13 Review System Recommendation (point d'extension : étape 6) ; alimente UC9 |
| **Déclencheur** | Une commande VALIDATED (« To ship ») attend, ou une expédition en cours doit être mise à jour |
| **Préconditions** | Planner authentifié et rattaché à une région ; commande validée ; modes de transport enregistrés |
| **Garantie minimale** | Rien n'est créé en cas d'échec ; jamais de produit froid en transport non réfrigéré ; jamais de lot expiré |
| **Garantie de succès** | `Shipment` créé (lots, mode, coût, ETA) ; lots réservés au DC ; commande en PLANNED ; visible du site ; `ShipmentLine` assure la traçabilité ; alerte si retard |

## Scénario nominal

1. Le planner ouvre la page des expéditions.
2. Le système affiche les commandes de la région à expédier et les expéditions en cours.
3. Le planner sélectionne une commande.
4. Le système affiche les produits, les quantités, le site de destination, la date souhaitée et le besoin de chaîne du froid.
5. Le système choisit les lots au DC en **FEFO** et vérifie que le stock suffit.
6. Le système liste les modes de transport (coût, délai) et **recommande** le meilleur (`recommendMode()`). *Point d'extension : UC13.*
7. Le planner garde le mode recommandé ou en choisit un autre, et saisit la date de départ.
8. Le système vérifie la chaîne du froid et calcule l'ETA.
9. Le système affiche un récapitulatif.
10. Le planner confirme.
11. Le système crée le `Shipment` et ses `ShipmentLine`, réserve le stock (`reservedQuantity`) et passe la commande à PLANNED.
12. Le système affiche l'expédition planifiée.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| 3a | Expédition en cours | Changer son statut (IN_TRANSIT, DELIVERED) ou l'annuler si elle n'est pas partie. Annulation : la réservation est libérée. Livraison : le stock du site destinataire est mis à jour (UC3). |
| 5a | Stock insuffisant | Expédition partielle proposée ; sinon la commande reste à expédier et une alerte LOW_STOCK est créée |
| 5b | Lot qui expirerait avant l'arrivée | Ignoré, on prend le lot suivant |
| 6a | Aucun mode compatible (froid ou date) | Message ; le planner change la date |
| 8a | Mode non conforme à la chaîne du froid | **Refusé**, retour à l'étape 7 |
| 8b | ETA après la date souhaitée | Avertissement ; si le planner confirme, alerte DELAY (planner et pharmacien) |
| 10a | Annulation | Rien n'est créé |
| * | Erreur base de données | Rollback |

## MVC et classes

- **MVC :** `ShipmentInterface` → `ShipmentController` → `Order`, `Stock`, `TransportMode`, `Shipment`, `ShipmentLine`, `Alert`, `Recommendation`
- **Attributs à ajouter au modèle de groupe** (signalés par l'auteur) : `wantedDate`, `estimatedArrival`, `cost`, `delay`, `coldChain`, `reservedQuantity`, `ShipmentLine`. Tous sont déjà intégrés dans [MODELE_DOMAINE.md](../00_Global/MODELE_DOMAINE.md).
