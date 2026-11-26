# UC10 — Adjust System Recommendation

| | |
|---|---|
| **Référence** | [REFERENCE_ARNOULT_Quentin.pdf](REFERENCE_ARNOULT_Quentin.pdf) (« BEST OF » ; remarque du prof : « incohérence avec le diagramme général ») |
| **Autre version** | [alt_ABOUELFATH_Assil.pdf](alt_ABOUELFATH_Assil.pdf) (21 p.) : règles métier, exigences non fonctionnelles, **cycle de vie d'une recommandation** (diagramme d'états), tests d'acceptation, dictionnaire de données. À utiliser en complément. |

| | |
|---|---|
| **Acteur principal** | Regional Planner |
| **Acteur secondaire** | Pharmacist / Site Manager (notifié) |
| **Relations** | appelé depuis UC13 (bouton « Adjust ») |
| **Préconditions** | Planner authentifié ; la recommandation existe, concerne sa région et n'est pas encore expédiée |
| **Garantie minimale** | L'original n'est jamais perdu sans validation ; aucun ajustement ne viole la chaîne du froid ; chaque tentative est journalisée |
| **Garantie de succès** | La recommandation ajustée remplace l'active (`adjustedByPlanner = true`) ; changement journalisé (qui, quand, ancien et nouveau, motif) ; site notifié |

## Scénario nominal

1. Le planner sélectionne une recommandation et clique « Adjust ».
2. Le système affiche l'expédition, le DC source, les lots et leur péremption, le mode, l'itinéraire, le coût, le délai et l'exigence de froid.
3. Le planner modifie le mode, l'itinéraire, le DC source ou la date, et saisit une **justification**.
4. Le planner soumet.
5. Le système vérifie : délai tenu, chaîne du froid, stock disponible, FEFO, région autorisée.
6. Le système recalcule coût et délai et affiche l'original et l'ajusté côte à côte.
7. Le planner confirme.
8. Le système sauvegarde et écrit l'`AuditLog`.
9. Le système notifie le site et affiche une confirmation.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| 1a | Hors région | Accès refusé |
| 2a | Recommandation déjà expédiée ou modifiée | Refus, dernière version affichée, fin |
| 3a | Annulation | Rien n'est modifié |
| 4a | Justification manquante | Retour à l'étape 3 |
| 5a | Chaîne du froid violée | **Refus bloquant**, retour à l'étape 3 |
| 5b | Délai non tenu | Avertissement ; corriger, ou justifier puis continuer |
| 5c | Écart FEFO ou stock insuffisant | Avertissement ; corriger ou justifier |
| 8a | Erreur base de données ou réseau | Rollback, original conservé |
| 8b | Conflit de version | Refus, dernière version affichée (prévoir `Recommendation.version`) |

## MVC et classes

- **MVC :** `AdjustRecommendationView` → `RecommendationController` → `Recommendation`, `Shipment` / `Order`, `Batch`, `TransportMode` (appelé « TransportOption » ici), `AuditLog` (appelé « AdjustmentLog »), `Alert` (appelé « Notification »)
- **Écrans :** liste, écran d'ajustement (formulaire et comparaison), erreurs, confirmation.
