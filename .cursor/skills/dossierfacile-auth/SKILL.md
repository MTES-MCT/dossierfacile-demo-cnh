---
name: dossierfacile-auth
description: Authentifie un usager sur DossierFacile via Keycloak OIDC (flux authorization_code + PKCE) et obtient un token d'accès JWT valide pour l'API (preprod ou production). Utilise ce skill quand tu dois appeler l'API DossierFacile au nom d'un usager, obtenir ou rafraîchir un token, ou configurer le client Keycloak agent.
---

# Authentification DossierFacile

> **Environnement** : lire [`dossierfacile-config/environments.md`](../dossierfacile-config/environments.md) et résoudre `${SSO_URL}`, `${API_URL}`, `${REALM}`, `${CLIENT_ID}` selon l'environnement cible (preprod par défaut).

## Pré-requis

Un client Keycloak dédié à l'assistant doit exister dans le realm `${REALM}`.  
Si ce n'est pas le cas, lire [keycloak-client-setup.md](keycloak-client-setup.md) pour le créer.

Valeurs de connexion :

| Paramètre | Valeur |
|---|---|
| Keycloak base URL | `${SSO_URL}` |
| Realm | `${REALM}` |
| Client ID | `${CLIENT_ID}` |
| Scopes requis | `openid profile email dossier` |
| Grant type | `authorization_code` + PKCE (`S256`) |

## Flux d'authentification (authorization_code + PKCE)

### Étape 1 — Générer le code_verifier et code_challenge

```python
import secrets, hashlib, base64

code_verifier = secrets.token_urlsafe(64)
digest = hashlib.sha256(code_verifier.encode()).digest()
code_challenge = base64.urlsafe_b64encode(digest).rstrip(b'=').decode()
```

### Étape 2 — Démarrer le serveur de callback local

**Avant** d'ouvrir le navigateur, lancer un serveur HTTP sur le port 8888 pour capturer le `code` depuis la redirection. Sans ce serveur, le navigateur affiche une erreur (`chrome-error://`) et l'URL de callback est perdue.

```python
import http.server, urllib.parse, threading, sys

class CallbackHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        params = urllib.parse.parse_qs(urllib.parse.urlparse(self.path).query)
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(b'<html><body><h1>Connexion reussie !</h1></body></html>')
        if 'code' in params:
            print('CODE=' + params['code'][0])
            sys.stdout.flush()
        threading.Thread(target=self.server.shutdown).start()
    def log_message(self, format, *args):
        pass

server = http.server.HTTPServer(('localhost', 8888), CallbackHandler)
print('SERVER_READY')
sys.stdout.flush()
server.serve_forever()
```

Lancer ce script en arrière-plan et attendre `SERVER_READY` avant de continuer.

### Étape 3 — Ouvrir la page de login dans le navigateur (browser MCP)

Construire l'URL d'autorisation :

```
GET ${SSO_URL}/realms/${REALM}/protocol/openid-connect/auth
  ?client_id=${CLIENT_ID}
  &response_type=code
  &scope=openid%20profile%20email%20dossier
  &redirect_uri=http://localhost:8888/callback
  &code_challenge=<code_challenge>
  &code_challenge_method=S256
  &state=<random_state>
```

Naviguer vers cette URL avec le browser MCP. L'usager se connecte avec ses identifiants DossierFacile (email/mot de passe ou FranceConnect). Après connexion, Keycloak redirige vers `http://localhost:8888/callback?code=<authorization_code>&state=<state>`, le serveur local capture le code et affiche `Connexion reussie !`.

Récupérer le paramètre `code` depuis la sortie du serveur ou depuis l'URL visible dans le snapshot du browser MCP.

### Étape 4 — Échanger le code contre un token

```bash
curl -X POST \
  ${SSO_URL}/realms/${REALM}/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id=${CLIENT_ID}" \
  -d "code=<authorization_code>" \
  -d "redirect_uri=http://localhost:8888/callback" \
  -d "code_verifier=<code_verifier>"
```

Réponse JSON :

```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "token_type": "Bearer"
}
```

Stocker `access_token`, `refresh_token` et calculer `expires_at = now + expires_in - 30` (marge de 30 s).

### Étape 5 — Injecter le token dans les appels API

Ajouter le header sur chaque requête vers `${API_URL}/api/` :

```
Authorization: Bearer <access_token>
```

### Étape 6 — Rafraîchir le token avant expiration

Si `now >= expires_at`, exécuter :

```bash
curl -X POST \
  ${SSO_URL}/realms/${REALM}/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token" \
  -d "client_id=${CLIENT_ID}" \
  -d "refresh_token=<refresh_token>"
```

Mettre à jour `access_token`, `refresh_token` et `expires_at`.

Si le refresh échoue (refresh_token expiré), relancer le flux depuis l'étape 2.

## Vérification

Tester que le token est valide :

```bash
curl ${API_URL}/api/tenant/profile \
  -H "Authorization: Bearer <access_token>"
```

Réponse attendue : `200 OK` avec le profil JSON du locataire.

## Référence

Pour la configuration du client Keycloak : [keycloak-client-setup.md](keycloak-client-setup.md)
