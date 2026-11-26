# UC13 — Review System Recommendation

| | |
|---|---|
| **Référence** | [REFERENCE_IBOHN-MANDJOMBE_Naelle-Clara.pdf](REFERENCE_IBOHN-MANDJOMBE_Naelle-Clara.pdf) (4 p., assez succinct) |

| | |
|---|---|
| **Acteurs** | Regional Planner (et Administrator selon le doc) |
| **Relations** | `<<extend>>` UC7 Manage Shipment (point d'extension : étape 6) ; mène à UC10 (Adjust) |
| **Déclencheur** | Le planner ouvre « Review recommendations » depuis Manage Shipment, ou depuis la notification « recommendation waiting » |
| **Préconditions** | Utilisateur identifié ; profil Regional Planner ou Admin ; planner rattaché à au moins une région |
| **Garantie minimale** | Aucune expédition créée, aucun stock réservé sans validation |
| **Garantie de succès** | Recommandation VALIDATED, ADJUSTED ou REJECTED, décision stockée ; si validée, elle peut être adoptée dans UC7 |

## Scénario nominal

1. Le planner sélectionne « Review recommendations ».
2. Le système affiche les recommandations PROPOSED de la région, les plus urgentes d'abord.
3. Le planner en sélectionne une.
4. Le système affiche la commande, le site, les produits et lots, le DC d'origine, le mode et l'itinéraire, le coût, l'ETA, le statut froid et **2 alternatives** (`Recommendation.rank` 2 et 3).
5. Le planner compare.
6. Le planner choisit « Validate ».
7. Le système vérifie que la recommandation est toujours valide, la passe à VALIDATED et stocke la décision.
8. Le système confirme et propose d'adopter la recommandation (retour à UC7).

## Alternatives

| Type | Situation | Réaction |
|---|---|---|
| Erreur | Session expirée | Retour au login |
| Erreur | Recommandation obsolète | Refus, dernière version |
| Erreur | Accès hors région | Accès refusé |
| Cas inhabituel | **Adjust** | Vers UC10. Le Model revérifie stock, délai et froid ; un produit thermosensible sur un mode non compatible est refusé et les valeurs précédentes restaurées. |
| Cas inhabituel | **Reject** | Motif obligatoire (dialogue) |
| Cas inhabituel | Comparer les alternatives | Affichage côte à côte |
| Raccourci | Filtrer, quitter | — |
| Fin | Aucune recommandation en attente | Message |

## Écrans (HMI)

- Cartes de synthèse (en attente, urgentes, chaîne du froid).
- Filtres : recherche, site, type de produit, urgence, case « chaîne du froid ».
- Tableau : urgence, commande, destination, date du besoin, froid, coût, statut, bouton « Review ».
- Exemple d'erreur : « Standard truck is not allowed for Flu vaccine (2–8 °C) ».

## MVC et classes

`RecommendationView` → `RecommendationController` → `Recommendation`, `Order`, `Batch`, `Stock`, `TransportMode`, `DistributionCenter`, `AuditLog`
