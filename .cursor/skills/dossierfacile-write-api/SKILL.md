---
name: dossierfacile-write-api
description: Complète un dossier DossierFacile via l'API d'écriture (routes POST/DELETE). Couvre les noms, le type de candidature, les 5 catégories de documents (identité, résidence, profession, finances, impôts) et la déclaration sur l'honneur. Utilise ce skill pour savoir quelle route appeler, avec quel payload, lors du remplissage d'un dossier individuel (ALONE, sans garant).
---

# API d'écriture DossierFacile

**Base URL** : `${API_URL}` — lire [`dossierfacile-config/environments.md`](../dossierfacile-config/environments.md) pour résoudre cette variable (preprod par défaut).  
**Auth** : `Authorization: Bearer <access_token>` sur toutes les routes  
**Périmètre** : dossier individuel (`ALONE`), sans garant

## Contraintes d'ordre

1. **Noms** → obligatoirement en premier
2. **Type de candidature** → obligatoirement en second
3. **Documents** (identité, résidence, profession, ressources, impôts) → **ordre libre**
4. **Déclaration sur l'honneur** → obligatoirement en dernier, après les 5 catégories

## Routes résumées

| Route | Méthode | Body | Quand |
|---|---|---|---|
| `/api/register/names` | POST | JSON | Étape 1 |
| `/api/register/application/v2` | POST | JSON | Étape 2 |
| `/api/register/documentIdentification` | POST | multipart | Pièce d'identité |
| `/api/register/documentResidency` | POST | multipart | Résidence |
| `/api/register/documentProfessional` | POST | multipart | Situation pro |
| `/api/register/documentFinancial` | POST | multipart | Ressources |
| `/api/register/documentTax` | POST | multipart | Impôts |
| `/api/register/honorDeclaration` | POST | JSON | Validation finale |
| `/api/file/{id}` | DELETE | — | Supprimer un fichier |
| `/api/document/{id}` | DELETE | — | Supprimer un document |

Pour les payloads complets et exemples : lire [api-reference.md](api-reference.md)

## Format multipart

Pour toutes les routes de documents :

```
Content-Type: multipart/form-data
Authorization: Bearer <access_token>
```

Champs du formulaire :
- `typeDocument*` (string) — valeur API du type de document (ex. `FRENCH_IDENTITY_CARD`)
- `documents` (fichier) — un ou plusieurs fichiers PDF, JPG, PNG ou HEIC
- `noDocument` (boolean) — souvent obligatoire selon l'endpoint/sous-type
- `categoryStep` (string) — requis pour certains sous-types (résidence, financier, fiscal)
- `monthlySum` (number) — requis pour certains revenus (ex. `SALARY`)

Exemple Python :

```python
import requests

files = [("files", open("cni-recto.jpg", "rb")), ("files", open("cni-verso.jpg", "rb"))]
files = [("documents", open("cni-recto.jpg", "rb")), ("documents", open("cni-verso.jpg", "rb"))]
data = {"typeDocumentIdentification": "FRENCH_IDENTITY_CARD", "noDocument": "false"}

resp = requests.post(
    f"{API_URL}/api/register/documentIdentification",  # API_URL résolu depuis environments.md
    headers={"Authorization": f"Bearer {access_token}"},
    files=files,
    data=data,
)
```

## Gestion des erreurs

| Code HTTP | Cause probable | Action |
|---|---|---|
| `401 Unauthorized` | Token expiré ou absent | Rafraîchir le token (voir `dossierfacile-auth`) |
| `403 Forbidden` | Scope `dossier` manquant | Vérifier la configuration du client Keycloak |
| `400 Bad Request` | Payload invalide ou champ manquant | Vérifier le nom exact du champ (`typeDocument*`) dans `api-reference.md` |
| `413 Payload Too Large` | Fichier trop volumineux | Compresser ou réduire la résolution du fichier |

## Notes

- En cas de `400` sur `categoryStep`, renseigner la valeur attendue par le sous-type:
  - `TENANT` -> `TENANT_RECEIPT`
  - `SALARY` -> ex. `SALARY_EMPLOYED_MORE_3_MONTHS` ou `SALARY_EMPLOYED_LESS_3_MONTHS`
  - `MY_NAME` (taxe) -> ex. `TAX_FRENCH_NOTICE`
- En cas de `400` sur `noDocument`, envoyer explicitement `noDocument=false` lorsque des fichiers sont transmis.
- En cas de `400` sur `monthlySum` pour `SALARY`, envoyer un montant strictement positif (ex. `2000`).
