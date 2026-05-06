# Environnements DossierFacile

**Environnement par défaut : preprod** — utiliser preprod sauf indication explicite de l'usager.

## Preprod

| Variable | Valeur |
|---|---|
| `${SSO_URL}` | `https://sso-preprod.dossierfacile.fr/auth` |
| `${API_URL}` | `https://api-preprod.dossierfacile.fr` |
| `${TENANT_URL}` | `https://locataire-preprod.dossierfacile.fr` |
| `${REALM}` | `dossier-facile` |
| `${CLIENT_ID}` | `dossier-facile-agent` |

> **Note** : l'instance Keycloak preprod utilise le context-path `/auth` (déploiement legacy). Toutes les URLs Keycloak sont donc sous `${SSO_URL}/realms/...` = `https://sso-preprod.dossierfacile.fr/auth/realms/...`.

## Production

| Variable | Valeur |
|---|---|
| `${SSO_URL}` | `https://sso.dossierfacile.logement.gouv.fr` |
| `${API_URL}` | `https://api.dossierfacile.logement.gouv.fr` |
| `${TENANT_URL}` | `https://locataire.dossierfacile.logement.gouv.fr` |
| `${REALM}` | `dossier-facile` |
| `${CLIENT_ID}` | `dossier-facile-agent` |

## Instructions pour l'agent

Avant tout appel API ou SSO dans les skills DossierFacile :

1. Si l'usager a précisé l'environnement (`preprod`, `production`, `prod`), utiliser la colonne correspondante.
2. Sinon, utiliser **preprod par défaut**.
3. Substituer mentalement `${SSO_URL}`, `${API_URL}`, `${TENANT_URL}`, `${REALM}`, `${CLIENT_ID}` dans toutes les URLs et commandes des skills.
