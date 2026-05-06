# Configuration du client Keycloak pour l'assistant DossierFacile

> Résoudre `${SSO_URL}` depuis [`dossierfacile-config/environments.md`](../dossierfacile-config/environments.md) selon l'environnement cible.

## Accès à la console d'administration

URL : `${SSO_URL}/admin/master/console/`  
Realm cible : `${REALM}`

## Créer le client `${CLIENT_ID}`

### 1. Créer le client

Dans la console Keycloak → realm `${REALM}` → **Clients** → **Create client** :

| Champ | Valeur |
|---|---|
| Client type | `OpenID Connect` |
| Client ID | `${CLIENT_ID}` |
| Name | `DossierFacile Assistant IA` |

### 2. Configurer le flux d'authentification

Onglet **Capability config** :

| Option | Valeur |
|---|---|
| Standard flow | **ON** (authorization_code) |
| Direct access grants | **OFF** |
| Implicit flow | **OFF** |
| Service accounts roles | **OFF** |
| Client authentication | **OFF** (public client — PKCE uniquement) |

### 3. Configurer les URIs

Onglet **Login settings** :

| Champ | Valeur |
|---|---|
| Valid redirect URIs | `http://localhost:8888/callback` |
| Valid post logout redirect URIs | `http://localhost:8888/` |
| Web origins | `http://localhost:8888` |

### 4. Configurer les scopes

Onglet **Client scopes** → **Add client scope** :  
Ajouter le scope `dossier` (doit exister dans le realm — c'est le scope qui donne accès à `SCOPE_dossier` dans l'API).

Vérifier que les scopes `openid`, `profile` et `email` sont présents dans **Assigned default client scopes**.

### 5. Forcer PKCE

Onglet **Advanced** → **Advanced settings** :

| Champ | Valeur |
|---|---|
| Proof Key for Code Exchange Code Challenge Method | `S256` |

### 6. Vérifier la configuration

Télécharger le fichier de configuration OIDC :  
`${SSO_URL}/realms/${REALM}/.well-known/openid-configuration`

Le champ `code_challenge_methods_supported` doit contenir `S256`.

## Valeurs de configuration résultantes

```
KEYCLOAK_URL=${SSO_URL}
KEYCLOAK_REALM=${REALM}
KEYCLOAK_CLIENT_ID=${CLIENT_ID}
REDIRECT_URI=http://localhost:8888/callback
SCOPES=openid profile email dossier
```

## Notes importantes

- Ce client est **public** (pas de secret) — la sécurité repose uniquement sur PKCE.
- Le scope `dossier` correspond à l'authority `SCOPE_dossier` côté API Spring Security. Sans ce scope, toutes les routes `/api/**` renverront `403 Forbidden`.
- En cas de doute sur le nom exact du scope, vérifier dans la console Keycloak → realm `${REALM}` → **Client scopes**.
- Ce client doit être créé **dans chaque environnement** (preprod et production) séparément.
