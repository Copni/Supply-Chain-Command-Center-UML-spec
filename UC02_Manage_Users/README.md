# UC02 — Manage Users

| | |
|---|---|
| **Référence** | [REFERENCE_MOGUO_Ariane.pdf](REFERENCE_MOGUO_Ariane.pdf) (29 p., « BEST OF ») : spec très complète, environ 21 écrans HMI, règles métier, exigences non fonctionnelles |
| **Autre version** | [alt_RAJAONERA_Emmanuel.pdf](alt_RAJAONERA_Emmanuel.pdf). Remarque du prof : « je ne comprends pas les lignes de vie à gauche » |
| ⚠️ **Numérotation** | MOGUO numérote ses UC autrement (voir le README racine). Son « UC4 Manage Data » correspond au UC04 officiel. |

| | |
|---|---|
| **Acteur principal** | Administrator |
| **Acteur secondaire** | E-mail Service (externe) |
| **Relations** | spécialise *Manage Network* ; `<<include>>` « Consult/search users » et « Assign role and site » ; `<<extend>>` modifier, désactiver, réinitialiser |
| **Préconditions** | Admin authentifié (rôle ADMIN, statut ACTIVE) ; sites et fournisseurs déjà créés (UC4) |
| **Garantie minimale** | Rien n'est sauvegardé partiellement ; aucun mot de passe n'est jamais affiché |
| **Garantie de succès** | Action effectuée, rôle et rattachement cohérents, e-mail envoyé si besoin, `AuditLog` écrit |

## Scénario nominal : créer un utilisateur

1. Menu Manage Network → Manage Users.
2. Le système affiche la liste des utilisateurs (recherche ; filtres rôle, site, statut).
3. Clic sur « Add a user ».
4. Le système affiche le formulaire.
5. L'admin saisit nom, prénom, e-mail, téléphone, rôle, puis le site ou le fournisseur selon le rôle.
6. Le système vérifie : champs obligatoires, format de l'e-mail, unicité de l'e-mail, cohérence rôle ↔ rattachement.
7. Le système crée l'utilisateur en statut **PENDING** et écrit `USER_CREATED` dans l'audit.
8. Le système envoie un e-mail d'activation (lien à usage unique, valable 72 h).
9. Le système affiche une confirmation.
10. L'utilisateur choisit son mot de passe ; son statut passe à **ACTIVE**.

## Variantes

| Id | Description |
|---|---|
| A1 | Rechercher et consulter (aucun résultat → « No user found ») |
| A2 | Modifier ; si le rôle ou le site change, afficher l'avant/après pour confirmation ; audit `USER_MODIFIED` |
| A3 | Désactiver (motif obligatoire, statut INACTIVE) ou réactiver |
| A4 | Réinitialiser le mot de passe (lien de 24 h) ; refusé si le compte est inactif |

## Erreurs et cas particuliers

| Id | Situation | Réaction |
|---|---|---|
| E1 | Champ manquant | Formulaire réaffiché avec l'erreur |
| E2 | E-mail déjà utilisé | Message (proposer de réactiver le compte existant) |
| E3 | Rattachement manquant pour le rôle | Message |
| E4 | Service e-mail indisponible | Envoi mis en file et relancé |
| E5 | Erreur base de données | Rollback |
| E6 | Session expirée | Retour au login |
| U1 | Dernier admin actif | Désactivation refusée |
| U2 | Désactiver son propre compte | Refusé |
| U4 | Modification concurrente | Conflit signalé |

## Règles métier

| Id | Règle |
|---|---|
| BR1 | Seul un ADMIN accède à ce UC. |
| BR2 | L'e-mail est unique, toutes les tables confondues. |
| BR3 | Un utilisateur a exactement un rôle. |
| BR4 | Le rôle impose un rattachement (voir l'invariant 5 de [MODELE_DOMAINE.md](../00_Global/MODELE_DOMAINE.md)). |
| BR5 | Il reste toujours au moins un admin actif. |
| BR6 | Pas de suppression physique d'un compte. |
| BR7 | Mot de passe haché (bcrypt), jamais visible. |
| BR8 | Tout est audité. |

## MVC et classes

- **MVC :** `ManageUsersView` → `UserController` → `User`, `Site`, `Supplier`, `AuditLog`, `EmailService`
- **Écrans :** HMI-01 à HMI-21 dans la référence (login, liste, détail, formulaire, confirmations, erreurs).
