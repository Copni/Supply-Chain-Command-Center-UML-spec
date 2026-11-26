# UC09 — View Incoming Deliveries

| | |
|---|---|
| **Référence** | [REFERENCE_DIOMANDE_Malick.pdf](REFERENCE_DIOMANDE_Malick.pdf) (« BEST OF » ; remarque du prof : « un peu léger sur les screens ») |

| | |
|---|---|
| **Acteur principal** | Pharmacist / Site Manager |
| **Préconditions** | Authentifié, rattaché à un site, rôle autorisé |
| **Garanties** | **Lecture seule** ; en cas d'échec, message explicite |

## Scénario nominal

1. Le pharmacien ouvre « Incoming deliveries ».
2. Le système vérifie le rôle et identifie le site (`getSiteByUser`).
3. Le système cherche les `Shipment` non livrés à destination de ce site (`findByDestination`).
4. Pour chaque expédition, le système récupère les lignes (produit, lot, quantité, péremption), le mode de transport et l'ETA.
5. Le système récupère les alertes ouvertes liées (retard, température, rappel).
6. Le système affiche la liste triée par ETA.
7. Le pharmacien sélectionne une livraison.
8. Le système affiche le détail : lignes, lots, péremption, statut chaîne du froid.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| A1 | Aucune livraison | « No delivery scheduled » |
| A2 | Filtre (statut, produit, numéro de lot) | `filterDeliveries`, nouvelle requête |
| A3 | Livraison avec alerte | Mise en évidence et bandeau d'alerte |
| A4 | Rôle non autorisé ou session expirée | Redirection vers le login |
| A5 | Base de données indisponible | Message d'erreur et bouton « Retry » |

## MVC et classes

- **MVC :** `IncomingDeliveriesView` → `DeliveryController` → `User`, `Site`, `Shipment`, `ShipmentLine`, `Batch`, `TransportMode`, `Alert`
- **Objet de transfert :** le Controller renvoie des `DeliveryDTO` à la View.

## Lien

La confirmation de réception (passage à DELIVERED, puis entrée en stock) n'est **pas** dans ce UC. Elle relève de UC7 (variante 3a) et de UC3 (mouvement IN).
