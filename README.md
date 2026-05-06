# dossierfacile-demo-cnh

Démonstrateur du pilotage d'une démarche administrative par un assistant IA.

Ce repository contient les **skills Cursor** permettant à un agent IA de compléter automatiquement un dossier de location sur [DossierFacile](https://www.dossierfacile.logement.gouv.fr/) à partir d'un prompt en langage naturel.

## Objectif

Permettre à un agent Cursor de :
1. S'authentifier sur DossierFacile au nom d'un usager (via Keycloak OIDC)
2. Collecter les informations et justificatifs nécessaires en posant des questions en français
3. Lire l'état d'avancement d'un dossier existant
4. Remplir et soumettre le dossier via l'API DossierFacile

**Périmètre actuel** : dossier individuel (`ALONE`), sans garant.

## Structure

```
.cursor/skills/
├── dossierfacile-config/
│   └── environments.md          # URLs par environnement (preprod / production)
├── dossierfacile-auth/
│   ├── SKILL.md                 # Flux OIDC PKCE — obtention et rafraîchissement du token
│   └── keycloak-client-setup.md # Guide de création du client Keycloak agent
├── dossierfacile-collect-info/
│   ├── SKILL.md                 # Séquence de questions à poser à l'usager
│   └── document-types.md        # Référence complète des types de documents (libellés français)
├── dossierfacile-read-profile/
│   ├── SKILL.md                 # Lecture et interprétation de GET /api/tenant/profile
│   └── profile-schema.md        # Schéma JSON annoté + exemples de réponses
├── dossierfacile-write-api/
│   ├── SKILL.md                 # Contraintes d'ordre et format des appels
│   └── api-reference.md         # Toutes les routes POST/DELETE avec exemples curl
└── dossierfacile-complete-dossier/
    └── SKILL.md                 # Skill d'orchestration de bout en bout
```

## Utilisation

Dans un agent Cursor, invoquer le skill principal :

> **`dossierfacile-complete-dossier`** — *« Complète le dossier DossierFacile de l'usager »*

L'agent orchestre automatiquement les étapes suivantes :

```
1. Authentification Keycloak (authorization_code + PKCE)
2. Lecture du profil existant (GET /api/tenant/profile)
3. Collecte des informations manquantes auprès de l'usager
4. Soumission des justificatifs via l'API
5. Déclaration sur l'honneur finale
```

## Environnements

Les endpoints sont centralisés dans `.cursor/skills/dossierfacile-config/environments.md`.

| Environnement | SSO | API | Interface locataire |
|---|---|---|---|
| **Preprod** (défaut) | `https://auth-preprod.dossierfacile.fr` | `https://api-preprod.dossierfacile.fr` | `https://locataire-preprod.dossierfacile.fr` |
| **Production** | `https://auth.dossierfacile.fr` | `https://api.dossierfacile.fr` | `https://locataire.dossierfacile.fr` |

L'agent utilise **preprod par défaut** sauf mention explicite de l'usager.

## Pré-requis

- Un client Keycloak `dossierfacile-agent` configuré dans le realm `dossier-facile` (voir `.cursor/skills/dossierfacile-auth/keycloak-client-setup.md`)
- Cursor avec accès au browser MCP pour le flux de login

## Licence

MIT — Fabrique numérique du Ministère de la Transition écologique
