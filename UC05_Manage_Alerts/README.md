# UC05 — Manage Alerts

| | |
|---|---|
| **Référence** | [REFERENCE_TETE-NDOM_Grecie.pdf](REFERENCE_TETE-NDOM_Grecie.pdf) (17 p.). Remarque du prof : « gestion des alertes très poussée », mais IHM hors template commun |

| | |
|---|---|
| **Acteur principal** | Administrator (seul acteur) |
| **Relations** | spécialise *Manage Network* |
| **Hors périmètre** | La création automatique d'alertes (faite par les autres UC) et le déclenchement d'un rappel (UC11) |
| **Préconditions** | Admin authentifié, autorisé à voir les alertes de tout le réseau |
| **Garantie minimale** | Pas de faux succès ; pas de mise à jour partielle |
| **Garantie de succès** | Alerte affichée avec son contexte (site, lot) ; toute mise à jour est sauvegardée avec son événement d'historique |

## Scénario nominal (consultation)

1. L'admin ouvre « Manage Alerts ».
2. Le système affiche la liste : type, priorité, statut, site, date.
3. L'admin recherche ou filtre.
4. L'admin sélectionne une alerte.
5. Le système affiche le détail : description, heure de détection, site, lot, et températures relevées et admises le cas échéant.
6. L'admin ouvre l'historique d'activité.
7. L'admin revient à la liste (aucune modification).

## Scénario étendu (suivi)

| Étape | Action | Transition |
|---|---|---|
| 8 | **Acknowledge** | NEW → IN_PROGRESS (+ audit) |
| 9 | **Assign** | Choix d'un assigné éligible et d'instructions |
| 10 | Actions correctives | Enregistrer l'avancement |
| 11 | **Resolve** | IN_PROGRESS → RESOLVED (note de résolution obligatoire) |
| 12 | **Close** | RESOLVED → CLOSED (vérification préalable requise) |

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| A1, A2 | Aucune alerte, ou aucun résultat | État vide |
| A3 | Alerte supprimée | « Not found » |
| A4 | Acknowledge sur une alerte qui n'est pas NEW | Refusé |
| A5 | Permission manquante | Action masquée **et** refusée côté serveur |
| A6 | Assigné invalide | Erreur de validation |
| A7, A8 | Note ou vérification manquante | Statut conservé |
| A9 | Erreur base de données | Rollback (mise à jour et événement d'historique ensemble) |
| A10 | Annulation | Rien n'est sauvegardé |
| A11 | Demande de rappel | Refusé ici, renvoi vers UC11 |

## Règles métier

| Id | Règle |
|---|---|
| BR-02 | Statuts : NEW, IN_PROGRESS, RESOLVED, CLOSED. |
| BR-06 | Chaque mise à jour génère un événement d'audit. |
| BR-07 | Clôturer une alerte n'autorise **pas** la remise en circulation d'un lot. |
| BR-08 | Pas de rappel depuis ce UC. |

## MVC et classes

- **MVC :** `AlertView` → `AlertController` → `Alert`, `Site`, `Batch`, `User` (assigné), `AuditLog`
- **Classe optionnelle :** `AlertAction` (actions correctives), si l'équipe garde l'étape 10.
