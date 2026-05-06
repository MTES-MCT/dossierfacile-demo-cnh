# Référence API d'écriture DossierFacile

> Résoudre `${API_URL}` depuis [`dossierfacile-config/environments.md`](../dossierfacile-config/environments.md) selon l'environnement cible (preprod par défaut).

Base URL : `${API_URL}`  
Toutes les routes requièrent : `Authorization: Bearer <access_token>`

---

## Étape obligatoire 1 — Noms du locataire

**À faire en premier.**

```
POST /api/register/names
Content-Type: application/json
```

```json
{
  "tenantId": 31988,
  "firstName": "Marie",
  "lastName": "Dupont",
  "zipCode": "75001",
  "abroad": false
}
```

- `tenantId` : **obligatoire** — identifiant numérique du locataire. Récupérer via `GET /api/tenant/profile` → champ `id`.
- `zipCode` : code postal français (5 chiffres). Peut être omis si `abroad: true`.
- `abroad` : `true` si le locataire réside hors de France.

---

## Étape obligatoire 2 — Type de candidature

**À faire en second. Valeur toujours `ALONE`.**

```
POST /api/register/application/v2
Content-Type: application/json
```

```json
{
  "applicationType": "ALONE"
}
```

---

## Documents — ordre libre

Les 5 routes suivantes peuvent être appelées dans n'importe quel ordre.

---

### Pièce d'identité

```
POST /api/register/documentIdentification
Content-Type: multipart/form-data
```

| Champ form-data | Type | Valeurs possibles |
|---|---|---|
| `typeDocumentIdentification` | string | `FRENCH_IDENTITY_CARD` \| `FRENCH_PASSPORT` \| `FRENCH_RESIDENCE_PERMIT` \| `DRIVERS_LICENSE` \| `FRANCE_IDENTITE` \| `OTHER_IDENTIFICATION` |
| `files` | fichier(s) | PDF, JPG, PNG, HEIC |
| `noDocument` | boolean (optionnel) | `true` si déclaration sans justificatif |

Exemple :

```bash
curl -X POST ${API_URL}/api/register/documentIdentification \
  -H "Authorization: Bearer $TOKEN" \
  -F "typeDocumentIdentification=FRENCH_IDENTITY_CARD" \
  -F "files=@cni-recto.jpg" \
  -F "files=@cni-verso.jpg"
```

---

### Situation d'hébergement

```
POST /api/register/documentResidency
Content-Type: multipart/form-data
```

| Champ form-data | Type | Valeurs possibles |
|---|---|---|
| `typeDocumentResidency` | string | `TENANT` \| `OWNER` \| `GUEST` \| `GUEST_COMPANY` \| `GUEST_ORGANISM` \| `SHORT_TERM_RENTAL` \| `OTHER_RESIDENCY` |
| `files` | fichier(s) | PDF, JPG, PNG, HEIC |
| `customText` | string (optionnel) | Description obligatoire si `OTHER_RESIDENCY` |

Exemple (locataire avec quittances) :

```bash
curl -X POST ${API_URL}/api/register/documentResidency \
  -H "Authorization: Bearer $TOKEN" \
  -F "typeDocumentResidency=TENANT" \
  -F "files=@quittance-mars.pdf" \
  -F "files=@quittance-avril.pdf" \
  -F "files=@quittance-mai.pdf"
```

---

### Situation professionnelle

```
POST /api/register/documentProfessional
Content-Type: multipart/form-data
```

| Champ form-data | Type | Valeurs possibles |
|---|---|---|
| `typeDocumentProfessional` | string | `CDI` \| `CDI_TRIAL` \| `CDD` \| `PUBLIC` \| `INDEPENDENT` \| `RETIRED` \| `UNEMPLOYED` \| `STUDENT` \| `ALTERNATION` \| `INTERNSHIP` \| `CTT` \| `INTERMITTENT` \| `ARTIST` \| `STAY_AT_HOME_PARENT` \| `NO_ACTIVITY` \| `OTHER` |
| `files` | fichier(s) | PDF, JPG, PNG, HEIC |
| `noDocument` | boolean (optionnel) | `true` si déclaration sans justificatif |

Exemple (CDI) :

```bash
curl -X POST ${API_URL}/api/register/documentProfessional \
  -H "Authorization: Bearer $TOKEN" \
  -F "typeDocumentProfessional=CDI" \
  -F "files=@contrat-travail.pdf"
```

---

### Ressources financières

```
POST /api/register/documentFinancial
Content-Type: multipart/form-data
```

| Champ form-data | Type | Valeurs possibles |
|---|---|---|
| `typeDocumentFinancial` | string | `SALARY` \| `SOCIAL_SERVICE` \| `PENSION` \| `RENT` \| `SCHOLARSHIP` \| `NO_INCOME` |
| `files` | fichier(s) | PDF, JPG, PNG, HEIC |
| `noDocument` | boolean (optionnel) | `true` pour `NO_INCOME` |

**Si l'usager a plusieurs types de revenus**, appeler cet endpoint **une fois par type** avec les fichiers correspondants.

Exemple (salaire) :

```bash
curl -X POST ${API_URL}/api/register/documentFinancial \
  -H "Authorization: Bearer $TOKEN" \
  -F "typeDocumentFinancial=SALARY" \
  -F "files=@fiche-paie-mars.pdf" \
  -F "files=@fiche-paie-avril.pdf" \
  -F "files=@fiche-paie-mai.pdf"
```

---

### Avis d'imposition

```
POST /api/register/documentTax
Content-Type: multipart/form-data
```

| Champ form-data | Type | Valeurs possibles |
|---|---|---|
| `typeDocumentTax` | string | `MY_NAME` \| `MY_PARENTS` \| `OTHER_TAX` |
| `files` | fichier(s) | PDF, JPG, PNG, HEIC |
| `customText` | string (optionnel) | Description si `OTHER_TAX`; ou précision du sous-cas (`TAX_FRENCH_NOTICE`, `TAX_FOREIGN_NOTICE`, `TAX_NOT_RECEIVED`, `TAX_NO_DECLARATION`) |
| `noDocument` | boolean (optionnel) | `true` si pas de document (ex. première déclaration) |

Exemple (avis français) :

```bash
curl -X POST ${API_URL}/api/register/documentTax \
  -H "Authorization: Bearer $TOKEN" \
  -F "typeDocumentTax=MY_NAME" \
  -F "customText=TAX_FRENCH_NOTICE" \
  -F "files=@avis-imposition-2025.pdf"
```

Exemple (sans déclaration) :

```bash
curl -X POST ${API_URL}/api/register/documentTax \
  -H "Authorization: Bearer $TOKEN" \
  -F "typeDocumentTax=MY_NAME" \
  -F "customText=TAX_NO_DECLARATION" \
  -F "noDocument=true"
```

---

## Étape obligatoire finale — Déclaration sur l'honneur

**À faire en dernier, après les 5 catégories de documents.**

```
POST /api/register/honorDeclaration
Content-Type: application/json
```

```json
{
  "honorDeclaration": true,
  "clarification": null
}
```

- `clarification` : texte libre optionnel si l'usager souhaite ajouter un message à destination du futur bailleur/propriétaire.
- Après cet appel, le statut global du dossier passe à `TO_PROCESS`.

---

## Opérations de maintenance

### Supprimer un fichier

```
DELETE /api/file/{fileId}
Authorization: Bearer <access_token>
```

Utiliser l'`id` de l'objet `file` récupéré dans la réponse du profil (`documents[].files[].id`).

### Supprimer un document (et tous ses fichiers)

```
DELETE /api/document/{documentId}
Authorization: Bearer <access_token>
```

Utiliser l'`id` de l'objet `document` récupéré dans la réponse du profil (`documents[].id`).  
Après suppression, la catégorie correspondante disparaît du profil et peut être soumise à nouveau.
