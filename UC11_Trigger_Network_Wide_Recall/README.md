# UC11 — Trigger a Network-Wide Recall

| | |
|---|---|
| **Référence** | [REFERENCE_TIH_Meriem.pdf](REFERENCE_TIH_Meriem.pdf) (27 p., « BEST OF ») : la spec la plus complète (traçabilité incomplète, livraison partielle des alertes) |
| **Autres versions** | [alt_HSAIN_Mohamed-Amine.pdf](alt_HSAIN_Mohamed-Amine.pdf) (« BEST OF ») : **diagramme d'états du rappel**, règles R1 à R7, **10 tests d'acceptation**. Remarque du prof : « DSEQ très complet, HMI pas terrible ». |
| | [alt_ZOUHDI_Yousri.pdf](alt_ZOUHDI_Yousri.pdf) (« BEST OF ») : version plus courte (pages 1 à 7 du document commun avec BENHAMMADI) |

| | |
|---|---|
| **Acteur principal** | Regulatory / Quality Officer (**seul** autorisé) |
| **Acteurs secondaires** | Notification Service (système) ; Pharmacist (informé) ; Admin (suivi) |
| **Relations** | étend UC12 Handle Quality Issue (gravité critique) |
| **Préconditions** | Authentifié avec le rôle QUALITY_OFFICER ; numéro de lot et problème qualité connus |
| **Garantie de succès** | `Recall` enregistré (motif, date, initiateur) ; lot RECALLED (ni délivrable, ni expédiable) ; tous les sites traçables identifiés ; une `Alert` RECALL par site avec son statut de livraison ; audit |
| **Garantie d'échec** | Aucun rappel si l'autorisation ou la validation échoue ; si des alertes échouent, le rappel reste actif en **ACTIVE_PARTIAL** |

## Scénario nominal (version TIH)

1. L'officer ouvre « Trigger a network-wide recall » (HMI 1).
2. L'officer saisit le n° de lot, le motif, la référence du rapport qualité et des notes, puis clique « Preview affected network ».
3. La View appelle `previewRecall(batchNumber, reason, notes, qualityRef)` sur le Controller.
4. Le Controller vérifie le rôle et les champs obligatoires.
5. Le Controller récupère le `Batch`, les sites qui détiennent le lot (`Stock`), les sites qui l'**ont reçu** (`ShipmentLine` et `StockMovement`), puis les `Site`.
6. Le Model renvoie lots, sites, quantités et dates.
7. Le Controller fusionne et vérifie la traçabilité ; la View affiche la prévisualisation (HMI 2) avec les éventuels avertissements.
8. L'officer coche la confirmation et clique « Confirm network recall ».
9. **En une transaction :** `Recall` créé (IN_PROGRESS), lot RECALLED, audit.
10. Pour chaque site : `Alert(RECALL)` créée et envoyée, statut de livraison stocké.
11. Le rappel passe à ACTIVE (ou ACTIVE_PARTIAL).
12. La View affiche l'identifiant, le nombre de sites et les statuts d'envoi (HMI 3).

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| A1 | Pas Quality Officer | Refus, audit ACCESS_DENIED |
| A2 | Lot vide ou inconnu | Champ en erreur |
| A3 | Traçabilité incomplète | Avertissement visible ; **ne jamais inventer de site** ; confirmation possible avec justification (`gapAcknowledged`) |
| A4 | Annulation à la prévisualisation | Seul un événement d'audit est enregistré |
| A5 | Échec partiel des notifications | ACTIVE_PARTIAL ; possibilité de réessayer ou de suivre manuellement |
| A6 | Base de données ou service indisponible | Rollback, erreur explicite |
| R4 (H'SAIN) | Rappel déjà actif pour ce lot | Avertissement, **pas de doublon** |

## Règles clés (H'SAIN)

| Id | Règle |
|---|---|
| R1 | Rôle vérifié **côté Controller**. |
| R2 | Rechercher et prévisualiser ne modifient rien. |
| R3 | Inclure les sites ayant reçu le lot même si leur stock actuel est à 0 (utiliser l'historique). |
| R5 | Une alerte ou un échec enregistré pour chaque site. |
| R6 | Audit complet. |
| R7 | Aucune donnée patient. |

## Tests d'acceptation

Les tests T1 à T10 de H'SAIN (section 9) sont directement utilisables comme tests d'intégration.

## MVC et classes

- **MVC :** `RecallView` → `RecallController` → `RecallService` (Model) → `Batch`, `Stock`, `ShipmentLine`, `StockMovement`, `Site`, `Recall`, `RecallAffectedSite`, `Alert`, `AuditLog`, `NotificationService`
