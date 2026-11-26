# UC12 — Handle Quality Issue

| | |
|---|---|
| **Référence** | [REFERENCE_DJOTEBONG-TIDONG_Ivane.pdf](REFERENCE_DJOTEBONG-TIDONG_Ivane.pdf) (13 p., bilingue EN/FR, « BEST OF » ; remarque du prof : « pas facile à utiliser, pourquoi réinventer le diagramme global, pas d'IHM ») |
| **Autre version** | [alt_BENHAMMADI_Sara.pdf](alt_BENHAMMADI_Sara.pdf) (« BEST OF ») : version plus courte, **avec IHM** (tableau de bord qualité). Ce sont la page 1 et les pages 8 à 13 du document commun avec ZOUHDI. |

| | |
|---|---|
| **Acteur principal** | Regulatory / Quality Officer |
| **Acteurs secondaires** | Pharmacist et Regional Planner (notifiés) |
| **Relations** | étendu par UC11 (gravité critique → rappel) |
| **Déclencheur** | Ouverture d'un problème qualité signalé (alerte température, retour fournisseur, inspection, signalement d'un site) |
| **Préconditions** | Authentifié avec le rôle QUALITY_OFFICER ; problème enregistré ; lot identifié et relié aux expéditions |
| **Garantie minimale** | Chaque décision est auditée ; aucun lot bloqué ou débloqué sans trace ; en cas d'échec, aucun statut ne change |
| **Garantie de succès** | Problème à jour ; nouveau statut du lot ; sites concernés identifiés et notifiés |

## Scénario nominal

1. L'officer ouvre la liste des problèmes qualité.
2. Le système affiche les problèmes ouverts, triés par gravité.
3. L'officer sélectionne un problème.
4. Le système affiche le lot, le site, le type, la date, la gravité et le statut.
5. L'officer demande les sites qui ont reçu le lot.
6. Le système affiche les expéditions et les sites (via `ShipmentLine`).
7. L'officer choisit une décision (**QUARANTINE, MONITORING ou CLOSE**) et saisit un commentaire.
8. Le système vérifie les droits et la cohérence de la décision.
9. Le système enregistre la décision : statut du `QualityIssue` et du `Batch` (QUARANTINE → `Batch.status = QUARANTINED`).
10. Le système notifie les pharmaciens et planners concernés.
11. Le système journalise et confirme.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| 2a | Aucun problème ouvert | Message, fin |
| 4a | Problème déjà clos ou pris par un autre officer | Lecture seule |
| 6a | Lot introuvable | Retour à l'étape 3 |
| 6b | Aucun site n'a reçu le lot | Clôture directe possible |
| 7a | Gravité **critique** | Possibilité de lancer **UC11** (`<<extend>>`) |
| 8a | Décision invalide ou commentaire manquant | Retour à l'étape 7 |
| 9a | Erreur d'enregistrement | Rien n'est conservé |
| 10a | Échec de notification | Reste en attente et sera renvoyée ; le UC réussit quand même |
| *a | Session expirée | Réauthentification, rien n'est sauvegardé |

## Règles métier

| Id | Règle |
|---|---|
| BR1 | Seul le QUALITY_OFFICER traite ce UC. |
| BR2 | Toute décision porte un commentaire. |
| BR3 | Un problème clos est en lecture seule. |
| BR4 | Le rappel reste un UC séparé (UC11). |
| BR5 | Seuls les sites ayant reçu le lot sont notifiés (RGPD). |
| BR6 | L'audit n'est pas modifiable. |

## MVC et classes

`QualityIssueForm` → `QualityIssueController` → `QualityIssue`, `Batch`, `Shipment`/`ShipmentLine`, `Site`, `Alert` (appelé « Notification »), `AuditLog`
