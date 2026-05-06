# Types de documents DossierFacile — Référence complète

Périmètre : dossier individuel (`ALONE`), sans garant.

---

## IDENTIFICATION — Pièce d'identité

Question : *« Quel document d'identité souhaitez-vous fournir ? »*

| Valeur API (`typeDocumentIdentification`) | Libellé français | Documents attendus |
|---|---|---|
| `FRENCH_IDENTITY_CARD` | Carte d'identité | Recto + verso de la carte d'identité française en cours de validité |
| `FRENCH_PASSPORT` | Passeport | Pages avec photo et état civil du passeport français |
| `FRENCH_RESIDENCE_PERMIT` | Carte ou titre de séjour | Titre de séjour français recto + verso |
| `DRIVERS_LICENSE` | Permis de conduire | Recto + verso du permis de conduire |

**Consigne** : la pièce doit être **en cours de validité**.

---

## RESIDENCY — Situation d'hébergement

Question : *« Quelle est votre situation d'hébergement actuelle ? »*

| Valeur API (`typeDocumentResidency`) | Libellé français | Documents attendus |
|---|---|---|
| `TENANT` | Locataire | 3 dernières quittances de loyer |
| `OWNER` | Propriétaire | Taxe foncière ou titre de propriété |
| `GUEST` | Hébergé chez quelqu'un (parents, ami, proche) | Attestation d'hébergement à titre gratuit de moins de 3 mois + pièce d'identité de l'hébergeant |
| `GUEST_ORGANISM` | Hébergé par un organisme (hébergement d'urgence, placement) | Attestation de l'organisme |
| `OTHER_RESIDENCY` | Autre situation (sans-abri, etc.) | Tout document justifiant la situation + description obligatoire |

**Consignes importantes** :
- Pour `TENANT` : **seules les quittances de loyer** sont acceptées. Les factures (eau, électricité…) et l'avis d'imposition ne sont **pas** valides ici.
- Pour `GUEST` : l'attestation doit dater de **moins de 3 mois**.

---

## PROFESSIONAL — Situation professionnelle

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

| Valeur API (`typeDocumentFinancial`) | Libellé français | Documents attendus |
|---|---|---|
| `SALARY` | Revenus professionnels (salaire, indépendant, intermittent…) | 3 derniers bulletins de salaire ou bilans comptables |
| `SOCIAL_SERVICE` | Prestations sociales (chômage, allocations CAF/MSA, APL, RSA…) | Attestation CAF, France Travail ou MSA de moins de 3 mois |
| `PENSION` | Pensions (alimentaire, invalidité, retraite complémentaire…) | Attestation de pension ou jugement de divorce fixant la pension |
| `RENT` | Rentes (revenus locatifs, rente viagère…) | Avis d'imposition avec revenus fonciers ou contrat de rente |
| `SCHOLARSHIP` | Bourses | Notification d'attribution de bourse |
| `NO_INCOME` | Pas de revenus | Déclaration sur l'honneur d'absence de revenus |

**Note** : l'usager peut avoir **plusieurs types de revenus**. Soumettre un appel API séparé (`POST /api/register/documentFinancial`) pour chaque type.

---

## TAX — Avis d'imposition

Question : *« Quelle est votre situation fiscale ? »*

### Niveau 1 — Situation principale

| Valeur API (`typeDocumentTax`) | Libellé français |
|---|---|
| `MY_NAME` | Vous avez un avis d'imposition à votre nom |
| `MY_PARENTS` | Vous êtes rattaché fiscalement à vos parents |
| `OTHER_TAX` | Autre situation fiscale |

### Niveau 2 — Sous-cas pour `MY_NAME`

Si l'usager a un avis à son nom, demander : *« Avez-vous un avis d'imposition français ou étranger ? L'avez-vous déjà reçu ? »*

| Valeur API (étape / `customText`) | Libellé français | Documents attendus |
|---|---|---|
| `TAX_FRENCH_NOTICE` | Avis d'imposition français | Dernier avis d'imposition ou avis de situation déclarative disponible sur impots.gouv.fr |
| `TAX_FOREIGN_NOTICE` | Avis d'imposition étranger | Document fiscal étranger + **traduction française obligatoire** des informations essentielles jointe au même envoi |
| `TAX_NOT_RECEIVED` | Avis non encore reçu | Déclaration sur l'honneur (uniquement si première déclaration en cours) |
| `TAX_NO_DECLARATION` | Pas encore effectué de première déclaration | Déclaration sur l'honneur + justificatif de situation |

### Sous-cas pour `MY_PARENTS`

Documents attendus : avis d'imposition des parents mentionnant le nom de famille du locataire.

### Sous-cas pour `OTHER_TAX`

Documents attendus : tout document fiscal pertinent + description obligatoire de la situation (champ `customText` dans la requête API).
