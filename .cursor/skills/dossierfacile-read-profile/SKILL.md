---
name: dossierfacile-read-profile
description: Lit et interprète le profil d'un locataire DossierFacile via GET /api/tenant/profile. Utilise ce skill pour connaître l'état d'avancement d'un dossier, identifier les sections manquantes ou refusées, et déterminer ce qu'il reste à compléter.
---

# Lecture du profil locataire DossierFacile

> **Environnement** : lire [`dossierfacile-config/environments.md`](../dossierfacile-config/environments.md) et résoudre `${API_URL}` selon l'environnement cible (preprod par défaut).

## Appel API

```
GET ${API_URL}/api/tenant/profile
Authorization: Bearer <access_token>
```

Paramètres optionnels (non requis pour la consultation de base) :
- `source`, `utm_source`, `utm_medium`, `utm_campaign` — données d'acquisition marketing, ignorer.

## Interprétation de la réponse

La réponse est un objet JSON représentant le profil complet du locataire. Voir [profile-schema.md](profile-schema.md) pour le schéma détaillé.

### Champs clés à vérifier

**Informations personnelles :**

| Champ | Présent si... |
|---|---|
| `firstName` | Le nom a été renseigné (`POST /api/register/names`) |
| `lastName` | idem |
| `zipCode` | idem (null si `abroad: true`) |

**Type de candidature :**

| Champ | Valeur attendue |
|---|---|
| `applicationType` | `"ALONE"` — si null, l'étape `POST /api/register/application/v2` n'a pas été faite |

**Documents — vérifier pour chaque catégorie :**

```
documents[].documentCategory  ∈ { IDENTIFICATION, RESIDENCY, PROFESSIONAL, FINANCIAL, TAX }
documents[].documentStatus    ∈ { TO_PROCESS, VALIDATED, DECLINED, INCOMPLETE }
documents[].noDocument        — true si l'usager a déclaré ne pas avoir de document
```

Un document est considéré **valide** si `documentStatus === "VALIDATED"`.  
Un document est considéré **présent** si la catégorie apparaît dans `documents[]` (quel que soit le statut).

**Déclaration sur l'honneur :**

| Champ | Valeur |
|---|---|
| `honorDeclaration` | `true` si la déclaration finale a été soumise |

**Statut global du dossier :**

| Valeur | Signification |
|---|---|
| `INCOMPLETE` | Des sections sont manquantes |
| `TO_PROCESS` | Dossier complet, en attente de vérification |
| `VALIDATED` | Dossier validé par un opérateur |
| `DECLINED` | Dossier refusé |

## Algorithme : ce qui manque

Pour un dossier `ALONE` sans garant, vérifier dans cet ordre :

```
1. firstName et lastName présents ?              → sinon : POST /api/register/names
2. applicationType === "ALONE" ?                 → sinon : POST /api/register/application/v2
3. Document IDENTIFICATION présent ?             → sinon : POST /api/register/documentIdentification
4. Document RESIDENCY présent ?                  → sinon : POST /api/register/documentResidency
5. Document PROFESSIONAL présent ?               → sinon : POST /api/register/documentProfessional
6. Document FINANCIAL présent ?                  → sinon : POST /api/register/documentFinancial
7. Document TAX présent ?                        → sinon : POST /api/register/documentTax
8. honorDeclaration === true ?                   → sinon : POST /api/register/honorDeclaration
```

Les étapes 3 à 7 peuvent être réalisées dans n'importe quel ordre.

## Cas particulier : document refusé

Si `documentStatus === "DECLINED"`, le document existe mais a été rejeté par un opérateur.  
Lire `documentDeniedReasons` pour connaître le motif, en informer l'usager, et soumettre un nouveau document via le même endpoint POST.

## Référence du schéma JSON

Voir [profile-schema.md](profile-schema.md) pour la structure complète et des exemples de réponses.
