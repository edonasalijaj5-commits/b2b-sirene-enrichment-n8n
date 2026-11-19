# 🏢 Pipeline d'Ingestion & Certification Légale SIRENE (n8n B2B Engine)

> **Étude de cas & Réalisation concrète** : Pipeline d'enrichissement Master Data et de vérification légale en temps réel pour CRM B2B (HubSpot, Salesforce, Pipedrive).

---

## 🎯 Contexte & Problématique Métier

Dans la majorité des entreprises B2B, l'ingestion des leads via formulaires web, salons ou campagnes marketing souffre d'un manque critique de gouvernance des données :
- **Données incomplètes ou erronées** : Noms commerciaux imprécis, absence de SIREN/SIRET, fautes d'orthographe.
- **Perte de temps commercial** : 15 à 30 minutes perdues par prospect par les équipes Sales pour rechercher manuellement les fiches légales et les dirigeants.
- **Risques de facturation & impayés** : Édition de devis sans entité juridique certifiée ni adresse de siège valide.
- **Pollution CRM** : Multiplication des doublons et fiches orphelines.

---

## 💡 Solution Architecturale (Production-Grade)

Conception et déploiement d'un workflow automatisé sous **n8n** opérant en temps réel dès la soumission d'un prospect :

```mermaid
graph LR
    A[🌐 Webhook Inbound] -->|nom_saisi, contact_email| B[🔍 HTTP Request - API SIRENE]
    B -->|Données légales INSEE| C[⚙️ Edit Fields - Master Data Normalization]
    C -->|Statut CERTIFIE / REJET| D{⚖️ Nœud If}
    D -->|True : Certifié| E[💾 Formatage Fiche CRM]
    D -->|False : Introuvable| F[🚨 Alerte Commerciale Révision]
```

### 1. Ingestion Temps Réel (Webhook)
- Réception instantanée des prospects via endpoint sécurisé (`GET` / `POST`).
- Extraction dynamique des paramètres de contact (`nom_saisi`, `contact_email`, `prospect_id`).

### 2. Requêtage Officiel Haute Vitesse (API Dinum / Recherche Entreprises)
- Interrogation de la base officielle nationale SIRENE / INSEE / RNE sans frais d'API.
- Moteur de recherche plein-texte flou (*fuzzy matching*) avec limitation de volumétrie optimale (`per_page=1`).

### 3. Normalisation & Résilience Master Data
- Rapprochement multi-étapes sans perte des attributs d'origine (`$('Webhook').item.json`).
- Sécurisation contre les plantages grâce aux valeurs de repli conditionnelles (`|| "INTROUVABLE"`).
- Extraction certifiée : **SIREN**, **Dénomination sociale officielle**, **Code NAF / APE**, **Adresse légale du siège**.

### 4. Aiguillage Décisionnel Métier
- **Circuit Nominal (`CERTIFIE`)** : Formatage standardisé de la fiche pour injection automatique dans le CRM ou la base client.
- **Circuit d'Exception (`A_TRAITER_MANUELLEMENT`)** : Alerte immédiate de l'équipe commerciale pour corriger le nom sans perdre l'opportunité d'affaires.

---

## 📊 Valeur Métier & ROI

| Métrique | Avant automatisation | Avec ce pipeline n8n |
| :--- | :--- | :--- |
| **Temps de qualification légale** | 15 à 20 min / lead | **0.4 seconde** (instantané) |
| **Taux d'erreur SIRET / Facturation** | 12 % à 18 % | **0 %** (source légale certifiée) |
| **Coût par requête** | 0.20 € à 0.50 € (solutions payantes) | **0.00 €** (API publique officielle) |
| **Protection anti-perte de lead** | Leads rejetés ignorés | **100 % alertés pour requalification** |

---

## 🚀 Déploiement & Import en 1 Clic

Le fichier de workflow complet est disponible dans ce dépôt : [`workflow_b2b_sirene.json`](./workflow_b2b_sirene.json).

### Pour l'importer dans votre instance n8n :
1. Téléchargez ou copiez le contenu de `workflow_b2b_sirene.json`.
2. Dans n8n, ouvrez un nouveau workflow et faites simplement **`Ctrl + V`** (ou menu en haut à droite ➔ *Import from File*).
3. Activez le nœud **Webhook** pour commencer à recevoir des leads en direct.

---

## 👩‍💻 Réalisation Technique
- **Stack** : n8n (Docker), REST API (Gouv.fr Recherche Entreprises), JavaScript ES6.
- **Domaine** : Master Data Management (MDM), Data Governance, Automatisation B2B CRM.
