# UC04 — Manage Data

| | |
|---|---|
| **Référence** | [REFERENCE_TEMGOUA_Igor_rapport.pdf](REFERENCE_TEMGOUA_Igor_rapport.pdf) (« BEST OF ») |
| **Diagrammes** | [diagrams/SD1_MVC_general.pdf](diagrams/SD1_MVC_general.pdf) (Model générique) et [diagrams/SD2_model_decompose.pdf](diagrams/SD2_model_decompose.pdf) (Model décomposé en 6 entités) |
| **Écrans** | [hmi/Interfaces.png](hmi/Interfaces.png) |

| | |
|---|---|
| **Acteur principal** | Administrator |
| **Relations** | spécialise *Manage Network* |
| **But** | Gérer le **référentiel** utilisé par tous les autres UC : Product, Batch, Stock, Site, Shipment, Supplier (et aussi TransportMode et Region, à ajouter) |
| **Préconditions** | Admin authentifié |
| **Garantie de succès** | Donnée consultée ou mise à jour après validation ; un message de succès ou d'erreur explicite est affiché |

## Scénario nominal

1. L'admin ouvre « Manage Data » et choisit une catégorie et des filtres (`searchData(category, filters)`).
2. Le Controller interroge l'entité correspondante (`findByCriteria`) et la View affiche les résultats.
3. L'admin sélectionne un enregistrement et le système en affiche le détail (`findById`).
4. L'admin soumet des modifications (exemple du document : changer la date de péremption d'un lot).
5. Le Controller fait `validateChanges`, puis `update`.
6. La View affiche le succès.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| A1 | Consultation seule | Aucune écriture |
| A2 | Données invalides | Erreurs de validation affichées, rien n'est sauvegardé |
| A3 | Enregistrement introuvable | Message « not found » |
| A4 | Erreur de persistance | Message d'échec, jamais de faux succès |

## À compléter à l'implémentation

Les **règles de validation** par entité (le document laisse ce point « pour la conception détaillée »). Points d'attention :

- Une modification de `Batch.expiryDate` ou de `Batch.status` doit être auditée.
- Un `Site` ne se supprime pas : il passe à `active = false`.
- La création d'un `Site` exige une `Region`.

## MVC

`ManageDataView` → `ManageDataController` → `Product`, `Batch`, `Stock`, `Site`, `Shipment`, `Supplier`, `AuditLog`
