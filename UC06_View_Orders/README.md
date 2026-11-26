# UC06 — View Orders

| | |
|---|---|
| **Référence** | [REFERENCE_KAWA_Blondelle.pdf](REFERENCE_KAWA_Blondelle.pdf). Remarque du prof : « OK pour l'HMI » |
| **Diagramme de séquence** | [diagrams/SD_viewOrders.png](diagrams/SD_viewOrders.png). Il manque dans le PDF (emplacement vide), mais l'image est complète. |
| **Prototype** | [prototype/](prototype/) : HTML/CSS/JS ; ouvrir `index.html` dans un navigateur. Données fictives, avec des boutons « DEMO CONTROLS » pour simuler les erreurs. |

| | |
|---|---|
| **Acteur principal** | Supplier (externe, accès restreint, lecture seule) |
| **Préconditions** | Fournisseur authentifié ; compte actif ; organisation enregistrée |
| **Garantie minimale** | Aucune commande n'est modifiée ; **aucune commande d'un autre fournisseur n'est divulguée** |
| **Garantie de succès** | Liste et détail des commandes autorisées |

## Scénario nominal

1. Le fournisseur choisit « View Orders ».
2. La View appelle le Controller avec la session (`getOrders(session)`).
3. Le Controller résout le `supplierId` à partir de la session (`getAuthorizedSupplierId`).
4. Le Controller récupère les commandes de ce fournisseur (`Order.findOrdersBySupplier(supplierId)`).
5. La View affiche la liste (recherche et filtres par statut).
6. Le fournisseur sélectionne une commande.
7. Le Controller la charge avec `findByIdAndSupplier(orderId, supplierId)` (**sécurité : filtrer toujours par les deux**), puis charge les `OrderLine`.
8. La View affiche produits, quantités, dates, destination et statut.

## Alternatives

| Id | Situation | Message |
|---|---|---|
| AF-01 | Session invalide ou permissions insuffisantes | « Access denied » |
| AF-02 | Aucune commande | « No orders available » |
| AF-03 | Erreur de récupération | Message d'erreur |
| AF-04 | Commande d'un autre fournisseur ou inexistante | Message **non divulgateur** (« not found or access denied ») |
| AF-05 | Erreur sur les lignes de commande | « Unable to load order details » |

## MVC et classes

- **MVC :** `OrdersInterface` → `ViewOrdersController` → `Supplier`, `Order`, `OrderLine` (appelée `OrderItem` dans le doc)
- **Lien avec le modèle de groupe :** association `Order *—* Supplier` (« handled by »).
