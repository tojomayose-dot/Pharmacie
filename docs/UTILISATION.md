# Guide d’utilisation — PharmaGest

Version du 18 novembre 2026. Ce guide explique les écrans et opérations de l’application. Pour démarrer le serveur, consulter le [manuel d’installation](INSTALLATION.md).

Les menus visibles dépendent des permissions du compte. Les intitulés peuvent varier légèrement entre le menu (« Achats et réceptions ») et le titre de page (« Achats & réceptions »).

## 1. Comprendre les notions

| Notion | Signification |
|---|---|
| Catégorie | Classement de médicaments, par exemple Antalgiques |
| Fiche médicament | Description du produit et de ses prix ; sa création n’ajoute pas de stock |
| Fournisseur | Entreprise auprès de laquelle la pharmacie commande |
| Commande | Produits demandés au fournisseur ; elle n’augmente pas le stock |
| Réception | Enregistrement des produits effectivement livrés ; elle augmente le stock |
| Lot | Série identifiée par un numéro, une péremption et une quantité |
| Stock physique | Quantités enregistrées, y compris les lots bloqués ou périmés |
| Stock disponible | Quantités des lots non bloqués et non périmés ; la caisse contrôle aussi l’autorisation du produit |
| Vente brouillon | Panier en préparation, sans sortie de stock ni paiement enregistré |
| Vente validée | Vente encaissée intégralement, stock déduit et historique conservé |
| Mouvement | Variation de stock liée à une réception, vente, retour ou inventaire |

Le parcours principal est : **Catégorie → Médicament → Fournisseur → Commande → Réception → Vente → Suivi**.

Choisir une unité cohérente pour chaque produit. Si le produit est géré par boîte, les prix, achats et ventes doivent tous correspondre à des boîtes. Le projet ne convertit pas automatiquement une boîte en comprimés.

## 2. Connexion et compte

1. Ouvrir l’adresse annoncée par le serveur, généralement http://127.0.0.1:8000/.
2. Saisir le **nom d’utilisateur** et le **mot de passe**.
3. Valider la connexion pour accéder au tableau de bord.

L’adresse courriel ne remplace pas le nom d’utilisateur. Dans le menu du compte en haut de page, choisir **Modifier mon mot de passe** ou **Se déconnecter**. Sur petit écran, le bouton **Menu** ouvre la navigation.

La page de connexion propose la réinitialisation du mot de passe. En développement, le courriel est affiché dans le terminal du serveur ; un serveur SMTP est nécessaire pour un envoi réel. Ne pas transmettre ses mots de passe dans le rapport ou les captures.

## 3. Rôles et utilisateurs

| Rôle standard | Usage principal |
|---|---|
| Administrateur | Modules métier et gestion des comptes/rôles |
| Pharmacien responsable | Catalogue, achats, stock, ventes et retours ; droit de vérification électronique à contrôler dans le document des permissions |
| Gestionnaire de stock | Catalogue, fournisseurs, commandes, réceptions et inventaires |
| Caissier | Consultation médicaments/lots, clients, panier et encaissement espèces |
| Responsable de gestion | Consultation des données autorisées, rapports, mouvements et audit |

Les rôles se cumulent ; les permissions personnalisées peuvent modifier cette répartition. Le groupe Administrateur et le superutilisateur Django sont distincts : appartenir au groupe ne donne pas automatiquement accès à `/admin/`.

Pour créer un utilisateur habilité :

1. Ouvrir **Utilisateurs et rôles → Créer un compte**.
2. Renseigner l’identifiant, les informations nécessaires et le mot de passe demandé.
3. Choisir un ou plusieurs rôles et laisser **Actif** coché.
4. Enregistrer et communiquer les accès de façon privée.

**Modifier** permet ensuite d’ajuster les informations, rôles et statut. Le formulaire de modification ne permet pas de remplacer directement le mot de passe ; utiliser le changement ou la réinitialisation. Un compte inactif ne peut plus se connecter. L’application refuse la désactivation de son propre compte depuis cet écran.

Pour les droits détaillés et leurs limites, consulter la [gestion des utilisateurs et des permissions](UTILISATEURS_PERMISSIONS.md).

## 4. Créer une catégorie

1. Ouvrir **Catégories** puis l’action d’ajout.
2. Saisir un **Nom** unique et, si utile, une **Description**.
3. Enregistrer.

Une catégorie utilisée par un médicament ne peut pas être supprimée. Éviter de recréer une catégorie déjà présente.

## 5. Créer un médicament

Ouvrir **Médicaments**, puis l’action d’ajout. Avec JavaScript, le formulaire présente des étapes : utiliser **Continuer**, vérifier le récapitulatif, puis **Créer le médicament**. Sans JavaScript, les champs restent accessibles sur la page.

| Champ | Contenu attendu |
|---|---|
| Nom commercial | Nom exact du produit |
| Dénomination générique | Substance active, facultative dans le formulaire |
| Catégorie | Classement déjà créé |
| Forme pharmaceutique | Comprimé, gélule, solution… selon le produit |
| Dosage | Concentration indiquée sur l’emballage ; ce n’est pas la quantité à vendre |
| Laboratoire | Fabricant, facultatif |
| Code-barres | Code exact de l’emballage, facultatif et unique s’il est renseigné |
| Photo | Facultative ; JPEG, PNG ou WebP, maximum 5 Mo |
| Prix d’achat (Ar) | Prix positif pour l’unité de gestion choisie |
| Prix de vente (Ar) | Prix positif pour cette même unité |
| Seuil d’alerte | Niveau à partir duquel surveiller le stock |
| Actif | Fiche utilisable dans les opérations autorisées |
| Ordonnance requise | À renseigner selon les règles de délivrance validées |
| Délivrance autorisée par le responsable | Autorisation de proposer le produit à la caisse |

La fiche n’enregistre pas la quantité livrée, le numéro de lot ou la péremption : ces données sont saisies à la réception. Pour des produits réels, utiliser les informations d’emballage et les prix de la pharmacie ; ne pas confondre prix d’exercice et tarifs réels.

La modification du prix de la fiche ne réécrit pas les anciennes lignes de vente. **Archiver** retire la fiche des nouvelles opérations concernées tout en conservant son historique.

## 6. Créer un fournisseur et un client

### Fournisseur

Dans **Fournisseurs**, ajouter une fiche : raison sociale et téléphone obligatoires ; adresse, courriel, identifiant fiscal et contact principal facultatifs. Laisser **Actif** coché pour pouvoir sélectionner ce fournisseur dans une nouvelle commande. Utiliser les coordonnées des documents du fournisseur ; pour un exercice, identifier explicitement les données fictives.

### Client

Dans **Clients**, renseigner le nom ; téléphone et coordonnées sont facultatifs. Conserver seulement les informations nécessaires à la gestion commerciale. Un client identifié n’est pas obligatoire pour une vente comptoir.

## 7. Préparer et valider une commande

1. Ouvrir **Achats et réceptions → Nouvelle commande**.
2. Sélectionner un fournisseur actif, puis **Créer la commande**.
3. Dans **Ajouter une ligne**, choisir le médicament, la quantité commandée et le prix d’achat unitaire.
4. Cliquer sur **Ajouter à la commande**. Répéter pour les autres produits.
5. Vérifier quantités et total ; **Retirer** permet de corriger une ligne du brouillon.
6. Cocher **Je confirme l’envoi de cette commande**, puis **Valider la commande**.

La commande passe de **Brouillon** à **Commandée**. **Aucun stock n’est ajouté à ce stade.** La confirmation enregistre le statut dans PharmaGest ; elle n’envoie pas automatiquement la commande au fournisseur par courriel ou API.

L’annulation est possible en brouillon ou commandée, avant toute réception. Après réception, l’application ne permet pas d’annuler simplement la commande.

## 8. Réceptionner les produits

Depuis le détail d’une commande commandée ou partiellement réceptionnée :

1. Cliquer sur **Réceptionner**.
2. Choisir la **Ligne de commande**.
3. Saisir la **Quantité reçue** réellement livrée.
4. Recopier le **Numéro de lot** et la **Péremption** des emballages.
5. Vérifier le récapitulatif, cocher **Je confirme la réception effective**, puis **Enregistrer la réception**.
6. Répéter pour les autres lignes ou lots.

Le stock augmente de la quantité reçue. La commande devient **Partiellement réceptionnée** si un reliquat reste, sinon **Entièrement réceptionnée**. Le **Bon de réception PDF** est disponible dans les réceptions enregistrées.

La quantité reçue ne peut pas dépasser le reliquat. Les lots déjà périmés sont refusés. Réutiliser un numéro de lot existant pour le même médicament exige la même péremption et le même coût. Ne pas inventer un numéro ou une date pour une réception réelle.

## 9. Effectuer une vente

### 9.1 Ouvrir la vente

Dans **Ventes et caisse**, cliquer sur **Ouvrir une vente comptoir**. La vente est créée en brouillon sans client identifié.

Pour l’associer à une fiche client, choisir **Avec un client**, sélectionner le client et cliquer sur **Ouvrir la caisse**.

### 9.2 Ajouter au panier

1. Dans **Rechercher ou scanner**, saisir le nom du médicament ou son code-barres, puis **Rechercher**.
2. Choisir le **Médicament** dans la liste et saisir la **Quantité**.
3. Laisser le lot automatique pour le choix **FEFO** : priorité au lot disponible qui expire en premier. Pour choisir un lot précis, ouvrir **Choisir un lot spécifique**.
4. Cliquer sur **Ajouter au panier**.
5. Vérifier produit, lot, quantité, prix unitaire et total. Répéter ou utiliser **Retirer** pour corriger une ligne.

Un même ajout peut être réparti entre plusieurs lots. La disponibilité affichée est indicative : un autre utilisateur peut vendre du stock avant la validation. Un brouillon ne réserve pas globalement les produits.

### 9.3 Encaisser en espèces

1. Dans **Encaissement**, sélectionner **Espèces**.
2. Si l’aide est affichée, saisir le **Montant reçu en espèces (Ar)** pour calculer la monnaie à rendre. Ce montant reçu n’est pas enregistré ; le paiement enregistré correspond au total de la vente.
3. Cocher **Ordonnance contrôlée si nécessaire** uniquement lorsque le contrôle requis a été effectué.
4. Cocher **Je confirme la délivrance et l’encaissement**.
5. Cliquer sur **Encaisser [montant]**.

Le système vérifie à nouveau le stock, la péremption, le blocage et l’autorisation de chaque produit. En cas de refus, lire le message, corriger ou actualiser le panier. Une validation réussie affiche **Vente encaissée**, déduit le stock et enregistre le paiement intégral. Les paiements partiels ne sont pas prévus.

Cliquer sur **Télécharger le reçu PDF** ou **Facture PDF**. Le téléchargement du document est distinct de l’encaissement : un problème d’ouverture PDF ne justifie pas une nouvelle vente.

### 9.4 Paiement électronique

Les choix Mobile money et Carte / virement dépendent des permissions. L’initialisation actuelle du groupe Pharmacien responsable présente un écart de codename : ce groupe ne reçoit pas automatiquement verify_payment ; voir le document des permissions. Un utilisateur habilité doit vérifier le paiement **auprès du prestataire, hors application**, saisir la référence et cocher la vérification avant d’encaisser. PharmaGest ne déclenche ni débit bancaire ni paiement automatique. Le caissier standard utilise les espèces.

Avec JavaScript actif, **F2** rejoint la recherche et **F4** l’encaissement. La validation reste explicite.

## 10. Vérifier les stocks et mouvements

Dans **Stocks et lots**, rechercher le produit ou le numéro de lot. Les filtres permettent de distinguer disponibles, bloqués, périmés et proches de péremption. L’échéance proche couvre aujourd’hui à 90 jours inclus. Le seuil d’alerte signale un stock inférieur ou égal au seuil défini.

Dans **Mouvements de stock**, retrouver les entrées et sorties avec leur auteur et référence. Le **Journal d’audit**, réservé aux comptes autorisés, conserve les actions métier journalisées. Ces pages n’autorisent pas à effacer l’historique d’une vente validée.

## 11. Réaliser un inventaire ou bloquer un lot

1. Dans **Stocks et lots**, ouvrir **Actions → Inventaire / blocage** sur le lot concerné.
2. Compter les unités physiques.
3. Saisir **Quantité physique comptée**, le **Motif du contrôle** et l’état **Bloquer le lot à la vente**.
4. Cocher **Je confirme cette correction de stock**, puis **Confirmer l’inventaire**.

La quantité saisie est le **nouveau total constaté**, pas une quantité à ajouter. Exemple : stock enregistré 17, comptage 15 → saisir 15 ; le mouvement sera −2. Un lot bloqué reste dans le stock physique mais ne peut pas être vendu. Débloquer un lot ne dispense pas du contrôle de péremption et d’éligibilité.

## 12. Enregistrer un retour de vente

Réservé aux utilisateurs disposant du droit de retour.

1. Ouvrir la vente validée et choisir **Enregistrer un retour**.
2. Sélectionner la **Ligne retournée**, la quantité et le motif.
3. Renseigner la **Référence du remboursement effectué** : le remboursement doit avoir été exécuté hors application.
4. Cocher **Produit contrôlé et éligible à la remise en stock** uniquement si les vérifications nécessaires ont été faites.
5. Cocher la confirmation, puis **Confirmer le retour**.

La quantité ne peut pas dépasser ce qui reste retournable. Le montant est calculé au prix historique. Une remise en stock demandée sur un lot périmé ou bloqué fait refuser le retour soumis. Sans remise en stock, le stock n’augmente pas. La vente d’origine et son paiement restent conservés ; une correction complète passe par les retours de ses lignes, pas par une suppression de la vente.

## 13. Tableau de bord et rapports

Choisir une période au tableau de bord pour les ventes et achats analysés. Les alertes de stock et péremption décrivent la situation actuelle, même lorsque la période financière est passée.

| Indicateur | Lecture |
|---|---|
| Chiffre d’affaires brut | Total des ventes validées de la période, avant remboursements |
| Remboursements | Retours remboursés pendant la période |
| Encaissements nets | Paiements confirmés moins remboursements de la période |
| Marge commerciale | Différence entre prix et coûts historiques, corrigée des retours concernés ; les autres charges ne sont pas déduites |
| Achats réceptionnés | Valeur des marchandises effectivement reçues pendant la période |

Dans **Rapports et documents**, choisir la période et le rapport autorisé : ventes, achats réceptionnés, stock ou péremptions. Les rapports de stock/péremption montrent la situation actuelle. Les retours cumulés affichés sur une vente peuvent inclure des remboursements postérieurs à la période sélectionnée.

Les bons de commande/réception sont accessibles sur les achats ; reçus et factures sur les ventes validées. Les PDF sont des documents de gestion : leurs mentions doivent être adaptées et validées avant une exploitation réelle.

## 14. Exercice complet sur données d’apprentissage

Utiliser uniquement une base destinée à l’apprentissage, sans opération réelle. Les prix ci-dessous sont fictifs.

1. Créer une catégorie d’exercice et un produit d’exercice, avec prix d’achat 1 000 Ar, prix de vente 1 500 Ar et seuil 10. Pour pouvoir le vendre dans cet exercice, le rendre actif et autorisé.
2. Créer un fournisseur clairement identifié comme fictif.
3. Commander 20 unités : stock inchangé.
4. Réceptionner les 20 unités, avec lot d’exercice explicite et une date future : stock 20.
5. Ouvrir une vente comptoir, ajouter trois unités et encaisser en espèces : total **4 500,00 Ar**, stock **17**.
6. Consulter le reçu et les mouvements : entrée **+20**, sortie **−3**.

L’exemple suppose un nouveau produit sans stock préalable. Pour la saisie des prix, utiliser une valeur simple comme `1000` ou `1500` ; l’application affiche ensuite les montants formatés.

## 15. Résoudre les difficultés courantes

| Situation | Vérification |
|---|---|
| Aucun médicament trouvé à la caisse | Vérifier le nom, puis Actif et Délivrance autorisée dans la fiche ; afficher tous les produits pour retirer le filtre |
| Produit trouvé, ajout refusé | Vérifier quantité, réception enregistrée, lot non bloqué/non périmé et stock suffisant |
| Stock toujours nul après commande | Réceptionner la livraison ; la commande seule n’ajoute rien |
| Bouton absent ou accès refusé | Vérifier les rôles avec l’administrateur ; ne pas contourner la permission par une URL |
| Ordonnance demandée | Effectuer le contrôle professionnel requis, puis cocher la confirmation correspondante |
| Opération concurrente / base occupée | Attendre la fin de l’autre opération, actualiser et vérifier l’état avant de réessayer |
| Nouvelle validation d’une vente déjà encaissée | Consulter cette vente et son reçu ; ne pas créer une autre vente pour le même paiement |
| Prix ou quantité erroné après validation | Faire traiter la correction selon la procédure autorisée de retour ; ne pas modifier directement la base |

## 16. Portée du guide

Les champs, boutons, droits et règles ont été confrontés aux templates, formulaires, vues et services du projet. Ce document n’ajoute ni fonctionnalité ni capture d’écran. Il ne remplace pas la validation professionnelle des règles de délivrance et ne prétend pas qu’une nouvelle recette navigateur a été exécutée.
