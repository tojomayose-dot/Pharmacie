# Gestion des utilisateurs et des permissions — PharmaGest

Version du **18 novembre 2026**. Ce document décrit les comptes, les rôles standard et les contrôles réellement définis dans le projet. Il complète le [guide d’utilisation](UTILISATION.md) et le [manuel d’installation](INSTALLATION.md).

Les tableaux ci-dessous sont établis à partir de `setup_roles.py`, des formulaires, vues, services et templates. Ils décrivent le paramétrage prévu par le code : les groupes et permissions actuellement enregistrés dans la base n’ont pas été inspectés pour cette documentation.

## 1. Compte, rôle et permission

| Notion | Signification |
|---|---|
| Utilisateur | Compte Django : identifiant, mot de passe protégé par hachage, coordonnées nécessaires et statut actif |
| Groupe / rôle | Ensemble de permissions, par exemple Caissier ou Gestionnaire de stock |
| Permission | Droit précis de consulter ou d’exécuter une opération |
| Compte actif | Peut s’authentifier avec des identifiants valides ; les opérations restent limitées par ses permissions |
| Compte staff | Peut accéder à l’administration Django si actif ; les droits d’administration sont ensuite contrôlés |
| Superutilisateur actif | Dispose de toutes les permissions, en complément des restrictions spécifiques appliquées par le projet |

Un utilisateur peut appartenir à plusieurs groupes. Les permissions de ses groupes et ses permissions individuelles se **cumulent**. Un rôle n’introduit pas de cloisonnement entre plusieurs pharmacies : les droits portent sur cette installation.

Le groupe **Administrateur** est un rôle métier ; il ne rend pas automatiquement le compte `is_staff` ou `is_superuser`. Le compte créé par `createsuperuser` est un superutilisateur Django, distinct de ce groupe.

## 2. Créer et modifier les comptes

### Premier compte

À la première installation, après les migrations et l’initialisation des groupes, l’installateur autorisé crée le premier compte avec `python manage.py createsuperuser`. Cette opération n’est pas nécessaire à chaque démarrage. Aucun mot de passe commun n’est fourni.

### Créer un utilisateur depuis PharmaGest

1. Se connecter avec un compte habilité.
2. Ouvrir **Utilisateurs et rôles**, puis **Créer un compte**.
3. Saisir un nom d’utilisateur unique, le prénom/nom et le courriel utiles.
4. Définir et confirmer le mot de passe demandé par le formulaire.
5. Sélectionner le ou les rôles nécessaires et vérifier **Actif**.
6. Enregistrer puis transmettre les accès par un moyen privé.

La liste nécessite `auth.view_user`, la création `auth.add_user`. Les comptes créés par ce formulaire ne deviennent pas automatiquement staff ou superutilisateurs. Sans rôle ni permission individuelle, un compte ne gagne pas de droit métier simplement en étant actif.

### Modifier, désactiver ou réactiver

Dans **Utilisateurs et rôles**, cliquer sur **Modifier**. Avec `auth.change_user`, le formulaire permet de modifier prénom, nom, courriel, groupes et statut actif. Il ne permet pas de changer l’identifiant existant, les permissions individuelles, les indicateurs staff/superutilisateur ou le mot de passe.

Décocher **Actif** désactive le compte sans supprimer l’historique de ses opérations. Recocher le champ le réactive. L’écran ne propose pas de suppression de compte.

Protections réellement présentes :

- Un utilisateur ne peut pas désactiver son propre compte depuis cette vue.
- Un compte qui n’est pas superutilisateur ne peut pas modifier un superutilisateur depuis cette vue.
- Le formulaire refuse la désactivation d’un superutilisateur ; son message renvoie à l’administration et à un autre superutilisateur habilité.

### Mot de passe et connexion

La connexion utilise le **nom d’utilisateur et le mot de passe**, pas l’adresse courriel. Un compte inactif est refusé par le mécanisme Django utilisé.

Le menu du compte propose **Modifier mon mot de passe** et **Se déconnecter**. La page de connexion permet la réinitialisation par courriel ; le backend console de développement affiche le message dans le terminal et n’envoie pas de véritable courriel. SMTP est requis pour l’envoi réel. La modification d’une fiche utilisateur ne réinitialise pas son mot de passe.

## 3. Les cinq rôles standard

| Rôle | Responsabilité principale |
|---|---|
| Administrateur | Ensemble des modules métier, gestion des comptes et attribution des groupes |
| Pharmacien responsable | Catalogue, achats, lots, caisse, retours et consultation de l’audit ; réserve sur le droit électronique ci-dessous |
| Gestionnaire de stock | Catalogue, fournisseurs, approvisionnement, réceptions, inventaires et mouvements |
| Caissier | Consultation du catalogue/lots, clients, panier et encaissement en espèces |
| Responsable de gestion | Consultation du catalogue, achats, ventes, stock, mouvements, rapports et audit |

La vue détaillée ci-dessous concerne ces groupes standard, sans cumul ni permission individuelle supplémentaire.

## 4. Matrice fonctionnelle des permissions

**A** = Administrateur ; **P** = Pharmacien responsable ; **S** = Gestionnaire de stock ; **C** = Caissier ; **G** = Responsable de gestion. « Oui » signifie que le groupe standard possède les droits du parcours. Les contrôles métier (stock, état de la commande, péremption, etc.) restent obligatoires.

| Action | A | P | S | C | G |
|---|---|---|---|---|---|
| Consulter médicaments, catégories et lots | Oui | Oui | Oui | Oui | Oui |
| Créer / modifier un médicament | Oui | Oui | Oui | — | — |
| Archiver un médicament par l’action dédiée | Oui | Oui | — | — | — |
| Créer / modifier une catégorie | Oui | Oui | Oui | — | — |
| Supprimer une catégorie inutilisée | Oui | Oui | — | — | — |
| Consulter les fournisseurs | Oui | Oui | Oui | — | Oui |
| Créer / modifier / archiver un fournisseur | Oui | — | Oui | — | — |
| Consulter / créer / modifier les clients | Oui | Oui | — | Oui | — |
| Archiver un client par l’action dédiée | Oui | — | — | — | — |
| Consulter les commandes et réceptions | Oui | Oui | Oui | — | Oui |
| Créer / modifier / confirmer / annuler une commande | Oui | Oui | Oui | — | — |
| Réceptionner une commande | Oui | Oui | Oui | — | — |
| Effectuer un inventaire / blocage de lot | Oui | Oui | Oui | — | — |
| Consulter les mouvements de stock | Oui | Oui | Oui | — | Oui |
| Consulter les ventes | Oui | Oui | — | Oui | Oui |
| Ouvrir une vente et gérer le panier | Oui | Oui | — | Oui | — |
| Encaisser une vente en espèces | Oui | Oui | — | Oui | — |
| Encaisser après vérification électronique | Oui | — | — | — | — |
| Enregistrer un retour | Oui | Oui | — | — | — |
| Exporter stock / péremptions | Oui | Oui | Oui | Oui | Oui |
| Exporter ventes / reçus / factures | Oui | Oui | — | Oui | Oui |
| Exporter achats / bons de commande / réception | Oui | Oui | Oui | — | Oui |
| Consulter le journal d’audit | Oui | Oui | — | — | Oui |
| Consulter / créer / modifier les comptes et leurs rôles | Oui | — | — | — | — |

Tous les comptes connectés peuvent accéder au tableau de bord et à la page des rapports ; seules les données autorisées et les liens d’export correspondants y sont accessibles. Un compte sans permission ne reçoit pas toutes les statistiques par cet accès commun.

## 5. Permissions techniques importantes

Les permissions métier ont le préfixe `pharmacy.` ; celles des comptes et groupes ont le préfixe `auth.`.

| Permission | Contrôle associé |
|---|---|
| `view_medicine`, `add_medicine`, `change_medicine`, `delete_medicine` | Consultation, création, modification et action d’archivage de médicament |
| `view_purchase`, `add_purchase`, `change_purchase` | Consultation, création et gestion d’une commande selon son état |
| `receive_purchase` | Réception de marchandises et entrée de stock |
| `view_lot` | Consultation des lots et export stock/péremptions |
| `adjust_stock` | Correction du comptage et blocage/déblocage d’un lot |
| `view_sale`, `add_sale`, `change_sale` | Consultation, ouverture et gestion du panier de vente |
| `validate_sale` | Validation de la délivrance, paiement intégral et sortie de stock |
| `verify_payment` | Contrôle externe manuel d’un paiement électronique, en plus du droit d’encaissement |
| `return_sale` | Retour de vente et référence du remboursement déjà réalisé |
| `view_stockmovement`, `view_auditlog` | Consultation des mouvements et du journal d’audit |
| `auth.view_user`, `auth.add_user`, `auth.change_user` | Liste, création et modification des comptes |
| `auth.view_group` | Permission attribuée au groupe Administrateur ; ne crée pas à elle seule un écran de gestion fine des groupes |

Le rôle Administrateur reçoit toutes les permissions de l’application pharmacy et quatre permissions auth : view_user, add_user, change_user et view_group. Il ne reçoit pas automatiquement toutes les permissions de l’application auth.

Django définit aussi des permissions de modèle sans route métier correspondante. Avoir une permission `delete_sale`, par exemple, ne crée pas un bouton permettant de supprimer une vente validée. Les opérations commerciales restent corrigées par les services et parcours prévus.

## 6. Comment l’application applique les droits

1. **Authentification** : les pages métier nécessitent une connexion. Les comptes inactifs ne peuvent pas se connecter avec le backend utilisé.
2. **Vues Django** : `gate()` et les contrôles `has_perm()` / `services.require()` refusent les accès non autorisés ; l’affichage d’un bouton n’est pas la seule protection.
3. **Services** : les mutations commerciales vérifient leurs permissions et règles avant de modifier les transactions et stocks. Un accès refusé ne donne pas le droit de poursuivre l’opération.
4. **Formulaires et CSRF** : les saisies et confirmations sont validées et les POST protégés contre les requêtes non autorisées par jeton CSRF. CSRF et permission remplissent deux fonctions distinctes.
5. **Templates** : `perms.*` masque les actions auxquelles le compte n’a pas accès. Les URL restent contrôlées côté serveur.
6. **Traçabilité** : les opérations prévues inscrivent l’auteur, la date et un AuditLog ; les créations/modifications de comptes par la vue métier sont enregistrées atomiquement avec leur audit.

L’utilisateur non connecté est normalement dirigé vers la connexion. Un utilisateur connecté sans le droit requis reçoit un refus d’accès, généralement HTTP 403. Les indicateurs et PDF ont leurs propres contrôles ; les photos servies par la route media exigent aussi connexion et `view_medicine`.

## 7. Nuances à connaître

### Autorisation d’un médicament

Le libellé « Délivrance autorisée par le responsable » n’a pas de permission spécifique. MedicineForm expose ce champ : les droits de création/modification de médicament permettent de renseigner `authorized`, `requires_prescription` et `active`. Le Gestionnaire de stock standard peut donc modifier ces cases. La matrice décrit le code, sans attribuer cette action exclusivement au Pharmacien responsable.

### Archivage par formulaire ou par action dédiée

Le bouton d’archivage exige `delete_*`. Les formulaires de modification peuvent aussi changer le champ actif avec `change_*`. Les lignes d’archivage de la matrice désignent l’action dédiée, pas toutes les possibilités de désactivation par formulaire. Une catégorie utilisée ne peut pas être supprimée.

### Écart constaté dans l’initialisation du rôle Pharmacien responsable

Le modèle Sale définit la permission `verify_payment`. Dans ROLE_PERMISSIONS, le rôle Pharmacien responsable indique l’action `verify_payment`, mais la boucle d’initialisation lui ajoute le suffixe `_sale` : elle cherche donc `verify_payment_sale`. Cette permission n’est pas définie par le modèle et le filtre Django ne l’attribue pas.

**Conséquence du code actuel : le groupe Pharmacien responsable standard ne reçoit pas automatiquement le droit de vérifier les paiements électroniques.** Le groupe Administrateur reçoit toutes les permissions pharmacy et possède bien `verify_payment`. Un superutilisateur actif dispose également de ce droit ; des permissions personnalisées peuvent modifier les accès effectifs.

La matrice tient compte de ce comportement, plutôt que du seul intitulé de rôle. Cet écart est documenté sans correction du code ou des groupes. Une future modification de cette règle devra faire l’objet d’une tâche distincte ; relancer setup_roles seul ne corrige pas le problème.

### Paiements et retours

Le Caissier standard encaisse en espèces ; la vérification électronique et les retours nécessitent des droits supplémentaires. Même avec ces droits, la vérification du paiement et l’exécution du remboursement se font hors application. Aucune passerelle bancaire n’est configurée par l’attribution d’un rôle.

### Administration Django

L’accès à `/admin/` requiert un compte actif staff et les permissions pertinentes. Le groupe Administrateur seul ne suffit pas. Les transactions commerciales, lots, mouvements et audits y sont enregistrés avec ReadOnlyAdmin : les écritures sont interdites par ces classes, y compris lorsqu’un compte dispose de permissions larges.

MedicineAdmin autorise les modifications permises, produit un audit à l’enregistrement et interdit la suppression. Category, Supplier et Customer utilisent l’administration standard ; toutes leurs écritures administratives ne produisent pas automatiquement un AuditLog métier. Les connexions mettent à jour last_login et la session ; le projet ne leur ajoute pas un journal de connexion dans AuditLog.

## 8. Initialisation et personnalisation des rôles

À la première installation, après migrate :

```bash
python manage.py setup_roles
```

La commande crée les groupes et **remplace les permissions attribuées à ces groupes par celles prévues dans le code**. Ne pas la relancer systématiquement après personnalisation. Elle ne crée pas les comptes de l’équipe et ne modifie pas leurs mots de passe. Aucune exécution de cette commande n’a été faite pour rédiger ce document.

L’interface PharmaGest attribue des groupes existants aux comptes. Elle ne fournit pas un éditeur complet des permissions individuelles ou de la définition des groupes ; une personnalisation fine relève d’un administrateur Django habilité dans `/admin/`. Conserver une distinction entre groupes standard et configurations personnalisées.

## 9. Exemples de répartition

- Un compte avec seulement **Caissier** peut préparer une vente et encaisser en espèces, sans réceptionner une commande ni enregistrer un retour.
- Un compte avec seulement **Gestionnaire de stock** peut réceptionner et inventorier, sans disposer des fonctions de vente.
- Un compte cumulant **Caissier** et **Gestionnaire de stock** réunit ces deux ensembles de droits ; ce cumul ne lui donne pas automatiquement `verify_payment` ou `return_sale`.
- Un **Responsable de gestion** consulte les données prévues et télécharge les rapports accessibles, sans modifier les stocks ou valider des ventes.

Attribuer les rôles correspondant aux tâches réellement confiées. Pour diagnostiquer un bouton absent ou un refus, vérifier avec le responsable les groupes et éventuelles permissions individuelles ; ne pas tenter de contourner l’accès par une URL.

## 10. Vérification documentaire

Sources : `pharmacy/management/commands/setup_roles.py`, `pharmacy/forms.py`, `pharmacy/views.py`, `pharmacy/services.py`, `pharmacy/admin.py`, `config/urls.py` et les templates de navigation/comptes. La matrice a été construite à partir de la déclaration des rôles, de la transformation des codenames dans setup_roles et des permissions réellement définies sur les modèles, sans lire les comptes ni modifier la base. Aucune nouvelle recette d’authentification, attribution de droits ou opération commerciale n’a été exécutée pour ce document.
