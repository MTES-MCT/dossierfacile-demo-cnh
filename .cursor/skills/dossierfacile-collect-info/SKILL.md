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

Proposer les options (voir `document-types.md` section IDENTIFICATION pour les valeurs API).

Une fois le type choisi, demander : *« Pouvez-vous transmettre le(s) fichier(s) correspondant(s) ? »*

### Étape 3 — Situation d'hébergement

Demander : *« Quelle est votre situation d'hébergement actuelle ? »*

Proposer les options (voir `document-types.md` section RESIDENCY).

Une fois la situation choisie, demander les documents correspondants (quittances, attestation, etc.).

### Étape 4 — Situation professionnelle

Demander : *« Quelle est votre situation professionnelle principale ? »*

Proposer les options (voir `document-types.md` section PROFESSIONAL).

Une fois la situation choisie, demander le(s) justificatif(s) correspondant(s).

### Étape 5 — Ressources financières

Demander : *« Quels sont vos types de revenus ? Vous pouvez en avoir plusieurs. »*

Proposer les options (voir `document-types.md` section FINANCIAL).

Pour chaque type de revenu sélectionné, demander les justificatifs correspondants.

### Étape 6 — Avis d'imposition

Demander : *« Quelle est votre situation fiscale ? »*

Proposer les 3 situations principales (voir `document-types.md` section TAX).

Si `MY_NAME` : demander si l'avis est français ou étranger, puis si l'usager l'a déjà reçu.  
Si `MY_PARENTS` : demander l'avis d'imposition des parents.  
Si `OTHER_TAX` : demander une description de la situation + tout document pertinent.

## Récapitulatif à la fin

Une fois tout collecté, résumer à l'usager :
- Prénom, nom, code postal
- Les 5 types de documents collectés avec leurs fichiers
- Demander confirmation avant de procéder au remplissage du dossier

## Référence complète des types de documents

Pour les valeurs API, libellés français et documents attendus par catégorie : lire [document-types.md](document-types.md)
