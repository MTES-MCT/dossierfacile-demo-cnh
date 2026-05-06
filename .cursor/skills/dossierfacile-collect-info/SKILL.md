---
name: dossierfacile-collect-info
description: Collecte auprès de l'usager toutes les informations et fichiers nécessaires pour compléter un dossier DossierFacile individuel (ALONE, sans garant). Utilise ce skill avant d'appeler l'API d'écriture, pour savoir quoi demander à l'usager et quels documents récupérer.
---

# Collecte des informations pour un dossier DossierFacile

**Périmètre : dossier individuel (`ALONE`), sans garant.**

Le type de candidature est toujours `ALONE` — ne pas le demander à l'usager.

## Séquence de collecte

Poser les questions dans cet ordre. Pour chaque étape, attendre la réponse avant de passer à la suivante.

### Étape 1 — Informations personnelles

Demander :
- *« Quel est votre prénom ? »*
- *« Quel est votre nom de famille ? »*
- *« Quel est votre code postal ? »* (ou demander si l'usager réside à l'étranger)

### Étape 2 — Pièce d'identité

Demander : *« Quel document d'identité souhaitez-vous fournir ? »*

Proposer les options (voir `document-types.md` section IDENTIFICATION). **Pas de `categoryStep` pour cette catégorie.**

Une fois le type choisi, demander : *« Pouvez-vous transmettre le(s) fichier(s) correspondant(s) ? »*

### Étape 3 — Situation d'hébergement

Demander : *« Quelle est votre situation d'hébergement actuelle ? »*

Proposer les options (voir `document-types.md` section RESIDENCY).

**`categoryStep` requis si la réponse est `TENANT` ou `GUEST` :**
- `TENANT` → poser : *« Avez-vous vos 3 dernières quittances de loyer, ou une attestation de bon paiement ? »* → `TENANT_RECEIPT` ou `TENANT_PROOF`
- `GUEST` → poser : *« Avez-vous une attestation d'hébergement à titre gratuit ? »* → `GUEST_PROOF` ou `GUEST_NO_PROOF`

Pour les autres sous-catégories (`OWNER`, `GUEST_COMPANY`, `GUEST_ORGANISM`, `SHORT_TERM_RENTAL`, `OTHER_RESIDENCY`) : pas de `categoryStep`, demander directement les documents correspondants.

### Étape 4 — Situation professionnelle

Demander : *« Quelle est votre situation professionnelle principale ? »*

Proposer les options (voir `document-types.md` section PROFESSIONAL). **Pas de `categoryStep` pour cette catégorie.**

Une fois la situation choisie, demander le(s) justificatif(s) correspondant(s).

### Étape 5 — Ressources financières

Demander : *« Quels sont vos types de revenus ? Vous pouvez en avoir plusieurs. »*

Proposer les options (voir `document-types.md` section FINANCIAL).

**`categoryStep` requis pour `SALARY`, `SOCIAL_SERVICE`, `PENSION`, `RENT` — pas de `categoryStep` pour `SCHOLARSHIP` et `NO_INCOME`.**

Pour chaque type sélectionné nécessitant un `categoryStep`, poser une question de précision :

- `SALARY` → *« Êtes-vous salarié, indépendant, intermittent ou artiste-auteur ? Depuis combien de temps ? »*
- `SOCIAL_SERVICE` → *« Quel type d'aide sociale percevez-vous ? »*
- `PENSION` → *« De quel type de pension s'agit-il ? »*
- `RENT` → *« De quel type de rente s'agit-il ? »*

Consulter `document-types.md` section FINANCIAL pour les sous-tables de `categoryStep` et les documents attendus.

### Étape 6 — Avis d'imposition

Demander : *« Quelle est votre situation fiscale ? »*

Proposer les 3 situations principales (`MY_NAME`, `MY_PARENTS`, `OTHER_TAX`) — voir `document-types.md` section TAX.

- `MY_NAME` → demander : *« Avez-vous un avis français ou étranger ? L'avez-vous déjà reçu ? »* → choisir le `categoryStep` correspondant (`TAX_FRENCH_NOTICE`, `TAX_FOREIGN_NOTICE`, `TAX_NOT_RECEIVED`, `TAX_NO_DECLARATION`). Pour `TAX_NOT_RECEIVED` et `TAX_NO_DECLARATION`, envoyer `noDocument: true`, aucun fichier requis.
- `MY_PARENTS` → demander l'avis d'imposition des parents. Pas de `categoryStep`.
- `OTHER_TAX` → demander une description + tout document pertinent. Pas de `categoryStep`, envoyer la description dans `customText`.

## Récapitulatif à la fin

Une fois tout collecté, résumer à l'usager :
- Prénom, nom, code postal
- Les documents collectés par catégorie
- Les `categoryStep` retenus pour résidence, finances et impôts
- Demander confirmation avant de procéder au remplissage du dossier

## Référence complète des types de documents

Pour les valeurs API, libellés français, `categoryStep` et documents attendus par catégorie : lire [document-types.md](document-types.md)
