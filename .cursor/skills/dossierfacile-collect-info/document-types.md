# Types de documents DossierFacile — Référence complète

Périmètre : dossier individuel (`ALONE`), sans garant.

> **`categoryStep`** est un champ form-data supplémentaire requis pour certaines sous-catégories (RESIDENCY : TENANT et GUEST ; FINANCIAL : SALARY, SOCIAL_SERVICE, PENSION, RENT). Pour les autres sous-catégories, ne pas envoyer ce champ.

---

## IDENTIFICATION — Pièce d'identité

**Pas de `categoryStep` pour cette catégorie.**

Question : *« Quel document d'identité souhaitez-vous fournir ? »*

| Valeur API (`typeDocumentIdentification`) | Libellé français | Documents attendus |
|---|---|---|
| `FRENCH_IDENTITY_CARD` | Carte d'identité | Recto + verso de la carte d'identité française en cours de validité |
| `FRENCH_PASSPORT` | Passeport | Pages avec photo et état civil du passeport français |
| `FRENCH_RESIDENCE_PERMIT` | Carte ou titre de séjour | Titre de séjour français recto + verso |
| `DRIVERS_LICENSE` | Permis de conduire | Recto + verso du permis de conduire |
| `FRANCE_IDENTITE` | France Identité | Justificatif issu de l'application France Identité |
| `OTHER_IDENTIFICATION` | Autre document d'identité | Tout document officiel avec photo en cours de validité |

**Consigne** : la pièce doit être **en cours de validité**.

---

## RESIDENCY — Situation d'hébergement

Question : *« Quelle est votre situation d'hébergement actuelle ? »*

**`categoryStep` requis uniquement pour `TENANT` et `GUEST`. Pour les autres sous-catégories, ne pas envoyer ce champ.**

| Valeur API (`typeDocumentResidency`) | Libellé français | `categoryStep` requis ? |
|---|---|---|
| `TENANT` | Locataire | Oui — voir sous-table ci-dessous |
| `OWNER` | Propriétaire | Non |
| `GUEST` | Hébergé chez quelqu'un (parents, ami, proche) | Oui — voir sous-table ci-dessous |
| `GUEST_ORGANISM` | Hébergé par un organisme (hébergement d'urgence, placement) | Non |
| `OTHER_RESIDENCY` | Autre situation (sans-abri, etc.) | Non |

### `TENANT` — sous-types (`categoryStep`)

Question : *« Avez-vous vos 3 dernières quittances de loyer, ou une attestation de bon paiement ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus |
|---|---|---|
| `TENANT_RECEIPT` | Quittances de loyer | Les 3 dernières quittances de loyer (quittances uniquement — factures et avis d'imposition refusés) |
| `TENANT_PROOF` | Attestation de bon paiement des loyers | Attestation de bon paiement des loyers de **moins de 3 mois** |

### `GUEST` — sous-types (`categoryStep`)

Question : *« Avez-vous une attestation d'hébergement à titre gratuit ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus |
|---|---|---|
| `GUEST_PROOF` | Avec attestation d'hébergement | Attestation d'hébergement à titre gratuit de **moins de 3 mois** |
| `GUEST_NO_PROOF` | Sans attestation d'hébergement | Compléter l'attestation officielle (service-public.fr), la faire signer par l'hébergeant, puis la fournir |

### Documents pour les autres sous-catégories

| Sous-catégorie | Documents attendus |
|---|---|
| `OWNER` | Taxe foncière ou titre de propriété |
| `GUEST_COMPANY` | Attestation employeur de mise à disposition du logement |
| `GUEST_ORGANISM` | Attestation de l'organisme hébergeant |
| `SHORT_TERM_RENTAL` | Facture d'hôtel ou contrat de location |
| `OTHER_RESIDENCY` | Tout document justifiant la situation + description obligatoire (champ `customText`) |

---

## PROFESSIONAL — Situation professionnelle

**Pas de `categoryStep` pour cette catégorie.**

Question : *« Quelle est votre situation professionnelle principale ? »*

| Valeur API (`typeDocumentProfessional`) | Libellé français | Documents attendus |
|---|---|---|
| `CDI` | CDI | Contrat de travail complet et signé, ou attestation employeur de moins de 3 mois |
| `CDD` | CDD | Contrat à durée déterminée complet et signé |
| `PUBLIC` | Fonctionnaire | Arrêté de nomination ou contrat de la fonction publique |
| `INDEPENDENT` | Indépendant | Extrait Kbis, bilan comptable ou avis de situation SIRENE |
| `RETIRED` | Retraite | Bulletin de pension de retraite |
| `UNEMPLOYED` | Chômage | Attestation France Travail (ex Pôle Emploi) |
| `STUDENT` | Études | Carte étudiante ou certificat de scolarité en cours de validité |
| `ALTERNATION` | Alternance | Contrat d'apprentissage ou de professionnalisation |
| `INTERNSHIP` | Stage | Convention de stage |
| `CTT` | Intérimaire (CTT) | Contrat de mission d'intérim |
| `INTERMITTENT` | Intermittence | Contrat et attestation Audiens ou Pôle Emploi Spectacle |
| `ARTIST` | Artiste-auteur | Attestation de la Maison des Artistes ou AGESSA |
| `STAY_AT_HOME_PARENT` | Parent au foyer | Livret de famille + justificatif de situation du conjoint |
| `NO_ACTIVITY` | Sans activité | Déclaration sur l'honneur ou tout document pertinent |
| `OTHER` | Autre | Tout document justifiant la situation professionnelle |

---

## FINANCIAL — Ressources financières

Question : *« Quels sont vos types de revenus ? »* (plusieurs choix possibles)

**`categoryStep` requis pour `SALARY`, `SOCIAL_SERVICE`, `PENSION`, `RENT`. Pas de `categoryStep` pour `SCHOLARSHIP` et `NO_INCOME`.**

**Note** : l'usager peut avoir **plusieurs types de revenus**. Soumettre un appel API séparé (`POST /api/register/documentFinancial`) pour chaque type avec son `categoryStep` propre.

| Valeur API (`typeDocumentFinancial`) | Libellé français | `categoryStep` requis ? |
|---|---|---|
| `SALARY` | Revenus professionnels | Oui — voir sous-table |
| `SOCIAL_SERVICE` | Prestations sociales | Oui — voir sous-table |
| `PENSION` | Pensions | Oui — voir sous-table |
| `RENT` | Rentes | Oui — voir sous-table |
| `SCHOLARSHIP` | Bourses | Non — fournir la notification d'attribution |
| `NO_INCOME` | Pas de revenus | Non — déclaration sur l'honneur (`noDocument: true`) |

### `SALARY` — sous-types (`categoryStep`)

Question : *« Êtes-vous salarié, indépendant, intermittent ou artiste-auteur ? Depuis combien de temps ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus |
|---|---|---|
| `SALARY_EMPLOYED_LESS_3_MONTHS` | Salarié depuis moins de 3 mois | Tous les bulletins de salaire disponibles (1 ou 2) |
| `SALARY_EMPLOYED_MORE_3_MONTHS` | Salarié depuis plus de 3 mois | 3 derniers bulletins de salaire |
| `SALARY_EMPLOYED_NOT_YET` | Pas encore pris le poste | Page du contrat indiquant la rémunération à venir |
| `SALARY_FREELANCE_AUTOENTREPRENEUR` | Auto-entrepreneur | 3 dernières déclarations mensuelles de recettes, ou dernière déclaration trimestrielle |
| `SALARY_FREELANCE_OTHER` | Autre indépendant | Attestation du comptable ou dernier bilan comptable |
| `SALARY_INTERMITTENT` | Intermittent | 3 derniers justificatifs de déclaration mensuelle France Travail Spectacle |
| `SALARY_ARTIST_AUTHOR` | Artiste-auteur | Déclaration de revenus et d'activité la plus récente (Urssaf) ou dernier avis d'imposition |
| `SALARY_UNKNOWN` | Non renseigné | Non applicable (situation non identifiée) |

### `SOCIAL_SERVICE` — sous-types (`categoryStep`)

Question : *« Quel type d'aide sociale percevez-vous ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus |
|---|---|---|
| `SOCIAL_SERVICE_CAF_LESS_3_MONTHS` | CAF ou MSA depuis moins de 3 mois | Tous les justificatifs de versement disponibles (1 ou 2) |
| `SOCIAL_SERVICE_CAF_MORE_3_MONTHS` | CAF ou MSA depuis plus de 3 mois | Justificatifs de versement pour 3 mois consécutifs |
| `SOCIAL_SERVICE_FRANCE_TRAVAIL_LESS_3_MONTHS` | France Travail (chômage) depuis moins de 3 mois | Toutes les attestations de paiement disponibles (1 ou 2) |
| `SOCIAL_SERVICE_FRANCE_TRAVAIL_MORE_3_MONTHS` | France Travail (chômage) depuis plus de 3 mois | Attestations de paiement pour 3 mois consécutifs |
| `SOCIAL_SERVICE_FRANCE_TRAVAIL_NOT_YET` | France Travail pas encore perçu | Document d'ouverture de droit à l'allocation (ARE) |
| `SOCIAL_SERVICE_APL_LESS_3_MONTHS` | APL depuis moins de 3 mois | Tous les justificatifs disponibles (1 ou 2) |
| `SOCIAL_SERVICE_APL_MORE_3_MONTHS` | APL depuis plus de 3 mois | Justificatifs pour 3 mois consécutifs |
| `SOCIAL_SERVICE_APL_NOT_YET` | APL pas encore perçue | Copie d'écran de la simulation CAF d'aide au logement |
| `SOCIAL_SERVICE_AAH_LESS_3_MONTHS` | AAH depuis moins de 3 mois | Tous les justificatifs disponibles (1 ou 2) |
| `SOCIAL_SERVICE_AAH_MORE_3_MONTHS` | AAH depuis plus de 3 mois | 3 derniers justificatifs de versement |
| `SOCIAL_SERVICE_AAH_NOT_YET` | AAH pas encore perçue | Notification de décision de la MDPH |
| `SOCIAL_SERVICE_OTHER` | Autre aide sociale | Justificatif de l'aide avec montant et fréquence des versements |

### `PENSION` — sous-types (`categoryStep`)

Question : *« De quel type de pension s'agit-il ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus |
|---|---|---|
| `PENSION_STATEMENT` | Pension de retraite avec bulletin | Bulletin de pension de moins de 2 ans |
| `PENSION_NO_STATEMENT` | Pension de retraite sans bulletin | Dernier avis d'imposition |
| `PENSION_DISABILITY_LESS_3_MONTHS` | Pension d'invalidité depuis moins de 3 mois | Tous les justificatifs disponibles (1 ou 2) |
| `PENSION_DISABILITY_MORE_3_MONTHS` | Pension d'invalidité depuis plus de 3 mois | 3 derniers justificatifs de versement |
| `PENSION_DISABILITY_NOT_YET` | Pension d'invalidité pas encore perçue | Attestation de pension d'invalidité |
| `PENSION_ALIMONY` | Pension alimentaire | Justificatif de pension alimentaire (jugement ou attestation) |
| `PENSION_UNKNOWN` | Non renseigné | Non applicable (situation non identifiée) |

### `RENT` — sous-types (`categoryStep`)

Question : *« De quel type de rente s'agit-il ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus |
|---|---|---|
| `RENT_RENTAL_RECEIPT` | Revenus locatifs avec quittance | Une quittance de loyer de moins de 3 mois |
| `RENT_RENTAL_NO_RECEIPT` | Revenus locatifs sans quittance | Dernier avis d'imposition (ligne revenus fonciers) |
| `RENT_ANNUITY_LIFE` | Rente viagère | Attestation de rente viagère |
| `RENT_OTHER` | Autre rente | Attestation de rente |

---

## TAX — Avis d'imposition

Question : *« Quelle est votre situation fiscale ? »*

### Niveau 1 — Situation principale (`typeDocumentTax`)

| Valeur API | Libellé français |
|---|---|
| `MY_NAME` | Vous avez un avis d'imposition à votre nom |
| `MY_PARENTS` | Vous êtes rattaché fiscalement à vos parents |
| `OTHER_TAX` | Autre situation fiscale |

### Niveau 2 — `categoryStep` pour `MY_NAME`

Si l'usager a un avis à son nom, demander : *« Avez-vous un avis français ou étranger ? L'avez-vous déjà reçu ? »*

| Valeur `categoryStep` | Libellé français | Documents attendus | `noDocument` |
|---|---|---|---|
| `TAX_FRENCH_NOTICE` | Avis d'imposition français | Dernier avis d'imposition complet (toutes les pages) ou avis de situation déclarative sur impots.gouv.fr | `false` |
| `TAX_FOREIGN_NOTICE` | Avis d'imposition étranger | Document fiscal étranger + **traduction française obligatoire** des informations essentielles | `false` |
| `TAX_NOT_RECEIVED` | Avis non encore reçu (première déclaration) | Aucun fichier — déclaration sur l'honneur générée automatiquement | `true` |
| `TAX_NO_DECLARATION` | Première déclaration non encore effectuée | Aucun fichier — déclaration sur l'honneur générée automatiquement | `true` |

### `MY_PARENTS` et `OTHER_TAX`

Pour `MY_PARENTS` : pas de `categoryStep` — fournir l'avis d'imposition des parents mentionnant le nom du locataire.

Pour `OTHER_TAX` : pas de `categoryStep` — fournir tout document fiscal pertinent + description obligatoire dans le champ `customText`.
