# UC15 — Request Exceptional Replenishment

| | |
|---|---|
| **Référence** | [REFERENCE_BERTHAULT_Paul.pdf](REFERENCE_BERTHAULT_Paul.pdf) (« BEST OF ») |
| **Autres versions** | [alt_JALAGUIER_Frederic.pdf](alt_JALAGUIER_Frederic.pdf). Remarque du prof : « léger, pas de numéro, diagramme de classe ». |
| | [alt_auteur-inconnu_diagrammes_et_IHM/](alt_auteur-inconnu_diagrammes_et_IHM/) : PNG de 6 diagrammes de séquence (nominal, validation, doublon, annulation, erreur, modèle), diagramme de classes, UCD, IHM HTML et PNG. Aucun nom dans les fichiers ; c'est **probablement BENABDALLAH Sami** (3ᵉ étudiant sur UC15). Remarque du prof pour lui : « lien entre un modèle et un acteur qui n'est pas dans le diagramme général ». |

| | |
|---|---|
| **Acteur principal** | Pharmacist / Site Manager |
| **Acteur secondaire** | Regional Planner (reçoit la demande ; il la traite dans UC1) |
| **Déclencheur** | Besoin que le réapprovisionnement automatique ne couvrira pas à temps (pic de grippe, pénurie…) |
| **Préconditions** | Pharmacien connecté et rattaché à un site lui-même lié à un DC régional |
| **Garantie minimale** | Aucune demande invalide n'est enregistrée ; le stock ne change pas |
| **Garantie de succès** | Demande enregistrée (`Order` de type EXCEPTIONAL, statut PENDING) et Regional Planner alerté |

## Scénario nominal

1. Le pharmacien clique « Request exceptional replenishment ».
2. Le système affiche le formulaire avec le stock du site.
3. Le pharmacien choisit un médicament et une quantité, autant de fois que nécessaire (une `OrderLine` par médicament).
4. Le pharmacien saisit la date de livraison souhaitée, l'**urgence** (NORMAL, HIGH ou CRITICAL) et le **motif**, puis soumet.
5. Le système vérifie les données et le stock du DC (FEFO).
6. Le système enregistre la demande (PENDING) et crée une `Alert(REPLENISHMENT_REQUEST)` pour le planner.
7. Le système affiche la confirmation avec le numéro de demande.

## Alternatives

| Id | Situation | Réaction |
|---|---|---|
| 3a | Médicament bloqué par un rappel | Avertissement, ligne non ajoutée |
| 5a | Données manquantes ou invalides (quantité ≤ 0, motif vide…) | Champs en erreur, retour à l'étape 4 |
| 5b | Stock DC insuffisant | Quantité manquante affichée ; si le pharmacien confirme, `partial = true` ; sinon retour à l'étape 4 |
| *a | Annulation avant l'étape 6 | Rien n'est enregistré |
| (auteur inconnu) | Demande en doublon | Une demande identique est déjà en cours : avertissement |

## MVC et classes

- **MVC :** `ReplenishmentRequestView` → `ReplenishmentController` → `Stock`, `Product` (appelé « Medicine »), `Order` et `OrderLine` (appelés « ReplenishmentRequest / RequestLine »), `Alert`
- **Synonymes :** voir la table de [MODELE_DOMAINE.md](../00_Global/MODELE_DOMAINE.md). La classe `ExceptionalRequest` de l'auteur inconnu correspond à `Order(type = EXCEPTIONAL)`.
