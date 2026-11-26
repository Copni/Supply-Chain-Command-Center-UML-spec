# Supply Chain Command Center — Solvana Pharma

Spécification UML du projet ST2OOS (EFREI, 2026-2027), réorganisée pour l'implémentation.

> **À lire en premier :** ce fichier → [00_Global/MODELE_DOMAINE.md](00_Global/MODELE_DOMAINE.md) → le `README.md` du UC que vous implémentez.

> Répartition des tâches :
> - Full : https://claude.ai/artifact/B5UTNgUnfY9ZNT9AHr4WkK
> - Mandatory only : https://claude.ai/artifact/CD1TmqqnKcJJ3FwbaKpWda

---

## 1. Le projet en 30 secondes

Solvana Pharma est un distributeur pharmaceutique fictif : environ 10 fournisseurs, **8 centres de distribution (DC)** régionaux (certains avec chambre froide), environ **250 sites clients** (pharmacies, hôpitaux, canal en ligne) dans 5 pays et **4 modes de transport** (camion frigorifique, fret aérien à température contrôlée, camion standard, coursier).

Le **Supply Chain Command Center (SCCC)** doit offrir :

| # | Fonctionnalité | UC principaux |
|---|---|---|
| a | Vue de stock omnicanale en temps réel (un stock partagé pharmacie et en ligne) | 3, 14 |
| b | Réapprovisionnement intelligent, juste à temps, **FEFO** (*First Expired, First Out* : le lot qui expire en premier sort en premier) | 1, 15 |
| c | Recommandation du meilleur transport (coût, délai, chaîne du froid) | 7, 13, 10 |
| d | Traçabilité de chaque lot, de la source au patient | 11, 12, 9 |
| e | Alertes proactives (stock bas, température, péremption proche, retard, rappel) | 5, 3, 11 |
| f | Tableaux de bord | 8 |

Contraintes transverses : **RGPD** (minimisation des données, aucune donnée patient), **chaîne du froid** jamais rompue, **traçabilité / audit** de chaque action sensible, architecture **MVC** (View → Controller → Model ; la View n'accède jamais directement aux données).

Le contexte complet est dans [00_Global/MOGUO_Contexte_et_UCD.pdf](00_Global/MOGUO_Contexte_et_UCD.pdf).

---

## 2. Structure du dossier

```
UML spec/
├── README.md                     ← ce fichier
├── 00_Global/                    ← documents communs à tout le projet
│   ├── MODELE_DOMAINE.md         ← modèle de classes UNIFIÉ (à utiliser pour coder)
│   ├── UCD_numerotation_officielle.jpeg   ← photo du tableau avec les n° de UC (fait foi)
│   ├── UCD_photo_tableau.png / UCD_propre.jpg
│   ├── ClassDiagram_brouillon_groupe.jpg  ← brouillon du TD 3
│   ├── ClassDiagram_consolide_MOGUO.png   ← diagramme de classes consolidé (le plus complet)
│   ├── MOGUO_ClassDiagram_et_dictionnaire.pdf  ← dictionnaire de données typé
│   └── MOGUO_Contexte_et_UCD.pdf
├── UC01_Manage_Replenishment/ … UC15_Request_Exceptional_Replenishment/
│   ├── README.md                 ← fiche d'implémentation synthétique du UC
│   ├── REFERENCE_<NOM>.pdf       ← document de référence choisi pour ce UC
│   ├── alt_<NOM>.pdf             ← autres versions (utiles en complément)
│   ├── diagrams/ hmi/ prototype/ ← quand ils existent
└── _admin/                       ← liste des étudiants et appréciations du prof, originaux, correspondance ancien → nouveau
```

**Convention :** `REFERENCE_` est la version la plus complète et la mieux jugée (colonne « BEST OF » du prof). Les `alt_` complètent (autre scénario, prototype, diagramme d'états…).

---

## 3. Acteurs

| Acteur | Rôle (`Role`) | Portée des données |
|---|---|---|
| Administrator | `ADMIN` | Tout le réseau |
| Regional Planner | `REGIONAL_PLANNER` | Sa **région** |
| Pharmacist / Site Manager | `PHARMACIST` | Son **site** uniquement |
| Regulatory / Quality Officer | `QUALITY_OFFICER` | Tout le réseau ; **seul** à pouvoir déclencher un rappel |
| Supplier | `SUPPLIER` (et `PARTNER` pour un transporteur) | Uniquement les commandes de **son entreprise**, en lecture seule |
| E-mail / Notification Service | (système externe) | Envoi des e-mails d'activation et des alertes |

---

## 4. Catalogue des cas d'utilisation (numérotation officielle)

| UC | Nom | Acteur principal | Relations | Référence | Autres versions | État |
|---|---|---|---|---|---|---|
| — | **Manage Network** | Admin | parent abstrait de 2, 4, 5 | — | — | Abstrait |
| [01](UC01_Manage_Replenishment/) | Manage Replenishment | Regional Planner | étend UC3 (stock < seuil) | **aucun livrable** | — | ⚠️ À spécifier (proposition dans le README) |
| [02](UC02_Manage_Users/) | Manage Users | Admin | spécialise Manage Network | MOGUO | RAJAONERA | ✅ Très complet |
| [03](UC03_Manage_Stock/) | Manage Stock | Pharmacist | `<<include>>` UC14 | ISABEY | — | ✅ |
| [04](UC04_Manage_Data/) | Manage Data | Admin | spécialise Manage Network | TEMGOUA | — | ✅ (règles de validation à préciser) |
| [05](UC05_Manage_Alerts/) | Manage Alerts | Admin | spécialise Manage Network | TETE NDOM | — | ✅ Cycle de vie détaillé |
| [06](UC06_View_Orders/) | View Orders | Supplier | — | KAWA | — | ✅ + prototype HTML/JS |
| [07](UC07_Manage_Shipment/) | Manage Shipment | Regional Planner | étendu par UC13 | CAVAIGNAC | TUSSEAU | ✅ |
| [08](UC08_View_Dashboard/) | View Dashboard | Admin | — | YERIMA | — | ✅ (fichier nommé « 1 » par erreur) |
| [09](UC09_View_Incoming_Deliveries/) | View Incoming Deliveries | Pharmacist | — | DIOMANDE | — | ✅ |
| [10](UC10_Adjust_System_Recommendation/) | Adjust System Recommendation | Regional Planner | appelé depuis UC13 | ARNOULT | ABOUELFATH | ✅ |
| [11](UC11_Trigger_Network_Wide_Recall/) | Trigger Network-Wide Recall | Quality Officer | étend UC12 | TIH | H'SAIN, ZOUHDI | ✅ Très complet |
| [12](UC12_Handle_Quality_Issue/) | Handle Quality Issue | Quality Officer | étendu par UC11 | DJOTEBONG | BENHAMMADI | ✅ |
| [13](UC13_Review_System_Recommendation/) | Review System Recommendation | Regional Planner | `<<extend>>` UC7 | IBOHN MANDJOMBE | — | ✅ (assez succinct) |
| [14](UC14_View_Site_Stock/) | View Site Stock | Pharmacist | inclus par UC3 | CHEAV & ISABEY | DUVAL | ✅ |
| [15](UC15_Request_Exceptional_Replenishment/) | Request Exceptional Replenishment | Pharmacist | → traité par UC1 / UC7 | BERTHAULT | JALAGUIER, auteur inconnu | ✅ |
| (hors UCD) | **Login / Authentification** | Tous | prérequis de tous les UC | — | — | ⚠️ À faire (visible en bord de tableau : « Lo… ») |

### ⚠️ Deux numérotations circulent

La **numérotation officielle** est celle du tableau ([UCD_numerotation_officielle.jpeg](00_Global/UCD_numerotation_officielle.jpeg)) et de la liste du prof. Elle est utilisée partout ici.

Les documents de **MOGUO** (UC diagram, Class diagram, Manage Users) utilisent une autre numérotation (UC1 à UC16). Correspondance :

| MOGUO | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Officiel** | Network | 2 | 5 | 4 | 8 | 10 | 1 | 7 | 13 | 9 | 3 | 14 | 15 | 12 | 11 | 6 |

---

## 5. Flux métier global

```
 UC4 Manage Data (référentiel : sites, produits, lots, fournisseurs, transports)
 UC2 Manage Users (comptes, rôles, rattachement site / région / fournisseur)
        │
        ▼
 UC14 View Site Stock ◄─include── UC3 Manage Stock ──(stock < seuil)──► Alert LOW_STOCK
                                                                  └──extend──► UC1 Manage Replenishment
 UC15 Request Exceptional Replenishment ──► Order(EXCEPTIONAL, PENDING) ──► UC1 (validation planner)
                                                                                 │
                                                                                 ▼
 UC7 Manage Shipment ◄──extend── UC13 Review System Recommendation ──► UC10 Adjust Recommendation
        │ Shipment + ShipmentLine (lots FEFO)
        ▼
 UC9 View Incoming Deliveries (pharmacien du site destinataire)
        │ livraison → mise à jour du stock (UC3)
        ▼
 UC12 Handle Quality Issue ──extend (gravité critique)──► UC11 Trigger Network-Wide Recall
                                                              │ Batch RECALLED + Alert RECALL par site
                                                              ▼
 UC5 Manage Alerts (Admin)        UC8 View Dashboard (Admin, lecture seule)     UC6 View Orders (Supplier)
```

---

## 6. Ordre d'implémentation conseillé

| Lot | Contenu | Pourquoi |
|---|---|---|
| **0. Socle** | Projet MVC, base de données à partir de [MODELE_DOMAINE.md](00_Global/MODELE_DOMAINE.md), login et session, contrôle de rôle côté Controller, `AuditLog`, jeu de données de test | Tous les UC en dépendent |
| **1. Référentiel** | UC4 Manage Data, UC2 Manage Users | Crée les sites, produits, lots et comptes |
| **2. Stock** | UC14 View Site Stock, UC3 Manage Stock (+ alertes LOW_STOCK et NEAR_EXPIRY) | Cœur du système ; FEFO |
| **3. Commandes** | UC15 Exceptional Replenishment, UC1 Manage Replenishment, UC6 View Orders | Produit les `Order` |
| **4. Expéditions** | UC7 Manage Shipment, UC13 Review, UC10 Adjust, UC9 Incoming Deliveries | Consomme les `Order` ; traçabilité des lots |
| **5. Qualité** | UC12 Handle Quality Issue, UC11 Recall, UC5 Manage Alerts | S'appuie sur la traçabilité (ShipmentLine, StockMovement) |
| **6. Pilotage** | UC8 View Dashboard | Agrège tout le reste, en lecture seule |

### Règles transverses à coder une seule fois

1. **Autorisation côté Controller** (jamais seulement dans la View) : rôle + périmètre (site, région, fournisseur).
2. **FEFO** : un service unique `pickBatchesFEFO(site, product, qty)` qui exclut les lots expirés, en quarantaine ou rappelés. Utilisé par UC3, UC7, UC10, UC15.
3. **Chaîne du froid** : `product.temperatureSensitive ⇒ transportMode.coldChain`. Refus bloquant dans UC7, UC10 et UC13.
4. **Statut de lot bloquant** : un lot `RECALLED`, `QUARANTINED` ou `EXPIRED` ne peut être ni délivré, ni expédié, ni commandé.
5. **Audit** : chaque écriture sensible crée un `AuditLog` (auteur, date, action, ancienne et nouvelle valeur, motif).
6. **Transactions** : chaque opération est « tout ou rien » ; jamais de faux message de succès.
7. **Erreurs standard** : accès refusé, session expirée, base indisponible (bouton « Réessayer »), liste vide.

---

## 7. Points de vigilance

- **UC1 Manage Replenishment n'a aucun livrable** (aucun étudiant). Une spec minimale est proposée dans [UC01_Manage_Replenishment/README.md](UC01_Manage_Replenishment/README.md).
- **Les noms de classes divergent selon les auteurs** (Stock / StockItem, Product / Medicine, Notification / Alert, ExceptionalRequest / ReplenishmentRequest / Order…). Tous sont réconciliés dans [MODELE_DOMAINE.md](00_Global/MODELE_DOMAINE.md).
- **Classes absentes du diagramme de groupe mais nécessaires** : `Region`, `ShipmentLine`, `StockMovement`, `QualityIssue`, `RecallAffectedSite`, `OrderLine`.
- UC7 : TUSSEAU écrit `<<include>>` UC13, mais le diagramme global et CAVAIGNAC écrivent `<<extend>>`. **C'est `extend` qui est retenu.**
- UC6 : le diagramme de séquence n'apparaît pas dans le PDF (emplacement vide), mais il existe en image : [UC06_View_Orders/diagrams/SD_viewOrders.png](UC06_View_Orders/diagrams/SD_viewOrders.png).
