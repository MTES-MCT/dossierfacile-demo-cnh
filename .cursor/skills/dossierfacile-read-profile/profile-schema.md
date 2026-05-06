# Schéma de réponse — GET /api/tenant/profile

## Structure JSON annotée

```json
{
  "id": 12345,                          // identifiant interne du locataire
  "keycloakId": "uuid-keycloak",        // sub du JWT Keycloak
  "email": "locataire@example.com",
  "firstName": "Marie",
  "lastName": "Dupont",
  "zipCode": "75001",                   // null si abroad=true
  "abroad": false,

  "applicationType": "ALONE",          // null | "ALONE" | "COUPLE" | "GROUP"
  "honorDeclaration": false,           // true une fois la déclaration finale soumise

  "status": "INCOMPLETE",              // statut global (voir valeurs ci-dessous)
  "tenantType": "CREATE",              // type de parcours

  "documents": [
    {
      "id": 101,
      "documentCategory": "IDENTIFICATION",   // IDENTIFICATION | RESIDENCY | PROFESSIONAL | FINANCIAL | TAX
      "documentSubCategory": "FRENCH_IDENTITY_CARD",  // valeur spécifique au type choisi
      "documentStatus": "TO_PROCESS",          // TO_PROCESS | VALIDATED | DECLINED | INCOMPLETE
      "noDocument": false,                     // true si déclaration sans justificatif
      "customText": null,                      // texte libre pour OTHER_TAX, OTHER_RESIDENCY, etc.
      "watermarkDocument": {
        "name": "document-watermarked.pdf"
      },
      "files": [
        {
          "id": 201,
          "originalName": "cni-recto.jpg",
          "size": 1234567,
          "path": "/api/file/resource/201"
        }
      ],
      "documentDeniedReasons": []             // motifs de refus si documentStatus=DECLINED
    }
  ],

  "guarantors": [],                    // toujours vide dans le périmètre ALONE sans garant

  "apartmentSharing": {
    "id": 99,
    "status": "INCOMPLETE",
    "tenants": [...]                   // liste des locataires du foyer (un seul pour ALONE)
  },

  "lastUpdateDate": "2026-05-06T10:30:00Z",
  "creationDateTime": "2026-05-01T08:00:00Z"
}
```

## Valeurs de `status` / `documentStatus`

| Valeur | Signification |
|---|---|
| `INCOMPLETE` | Données ou documents manquants |
| `TO_PROCESS` | Complet, en attente de vérification par un opérateur |
| `VALIDATED` | Validé |
| `DECLINED` | Refusé — voir `documentDeniedReasons` |

## Exemple : dossier en cours (3 documents sur 5)

```json
{
  "firstName": "Marie",
  "lastName": "Dupont",
  "zipCode": "75001",
  "applicationType": "ALONE",
  "honorDeclaration": false,
  "status": "INCOMPLETE",
  "documents": [
    { "documentCategory": "IDENTIFICATION", "documentStatus": "TO_PROCESS", "documentSubCategory": "FRENCH_IDENTITY_CARD" },
    { "documentCategory": "RESIDENCY",      "documentStatus": "VALIDATED",  "documentSubCategory": "TENANT" },
    { "documentCategory": "FINANCIAL",      "documentStatus": "TO_PROCESS", "documentSubCategory": "SALARY" }
  ]
}
```

→ Manquent : `PROFESSIONAL`, `TAX`, et `honorDeclaration`.

## Exemple : dossier complet prêt à soumettre

```json
{
  "firstName": "Marie",
  "lastName": "Dupont",
  "zipCode": "75001",
  "applicationType": "ALONE",
  "honorDeclaration": false,
  "status": "INCOMPLETE",
  "documents": [
    { "documentCategory": "IDENTIFICATION", "documentStatus": "TO_PROCESS" },
    { "documentCategory": "RESIDENCY",      "documentStatus": "TO_PROCESS" },
    { "documentCategory": "PROFESSIONAL",   "documentStatus": "TO_PROCESS" },
    { "documentCategory": "FINANCIAL",      "documentStatus": "TO_PROCESS" },
    { "documentCategory": "TAX",            "documentStatus": "TO_PROCESS" }
  ]
}
```

→ Tous les 5 documents sont présents. Prêt pour `POST /api/register/honorDeclaration`.  
→ Le statut global passera à `TO_PROCESS` après la déclaration sur l'honneur.

## Exemple : document refusé

```json
{
  "documentCategory": "IDENTIFICATION",
  "documentStatus": "DECLINED",
  "documentSubCategory": "FRENCH_IDENTITY_CARD",
  "documentDeniedReasons": [
    { "messageItem": "La pièce d'identité est expirée." }
  ]
}
```

→ Informer l'usager du motif et soumettre un nouveau document via `POST /api/register/documentIdentification`.
