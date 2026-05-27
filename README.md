# 🏦 Automatisation des Paiements Locatifs — Attijari Bank

> Système intelligent de suivi, d'alerte et de prévision des paiements locatifs pour le parc immobilier d'une banque tunisienne — remplaçant intégralement un processus manuel basé sur Excel.

> ⚠️ Pour des raisons de confidentialité et de protection des données internes de l’entreprise, le code source, les données, les dashboards Power BI et les documents techniques détaillés ne sont pas publiés.  
> Ce dépôt présente uniquement une vue d’ensemble fonctionnelle, technique et architecturale du projet réalisé durant mon stage.

---

## Vue d'ensemble

Attijari Bank gère les baux immobiliers de plus de 100 agences réparties sur toute la Tunisie via son entité Patrimoine Immobilier. Avant ce projet, l'intégralité du suivi — échéances, montants révisés, fins de contrat, confirmations de paiement — était traitée manuellement dans des fichiers Excel, ce qui engendrait des retards, des oublis d'échéances et des reconductions de contrats non souhaitées.

Ce projet remplace ce processus par un pipeline de données complet :

1. **Une base de données relationnelle** centralisant tous les contrats, paiements, agences et fournisseurs
2. **Un moteur d'alertes automatiques** qui envoie des emails contextuels avec boutons de confirmation interactifs
3. **Un tableau de bord Power BI** connecté en direct à la base pour le suivi en temps réel
4. **Un modèle prédictif** estimant l'évolution des prix du marché locatif commercial jusqu'en 2035

---

## Mon rôle

Ce projet a été réalisé en binôme dans le cadre d'un stage de 2 mois (mars – mai 2025). Voici mes contributions spécifiques :

- **Nettoyage et normalisation des données** : standardisation des valeurs (périodicités, noms d'agences, modalités de révision), suppression des doublons et fusion de colonnes redondantes depuis le fichier Excel source

- **Conception du schéma PostgreSQL** : modélisation des 4 tables (`agence`, `fournisseur`, `contrat`, `paiement`) avec clés primaires, clés étrangères et contraintes d'intégrité référentielle

- **Développement des fonctions de calcul métier** : calcul du montant HT/TTC à partir du dernier montant annuel, calcul de la prochaine date d'échéance selon la périodicité, calcul du montant révisé en appliquant le taux de révision et la TVA

- **Système d'alertes** : script Python planifié appliquant les règles J-7 / J-5 / J-2 / quotidien post-échéance, et règles J-30 / J-15 pour les fins de contrat

- **Emails interactifs** : boutons HTML/CSS OUI/NON intégrés dans les mails, reliés à un backend Flask déclenchant des UPDATE PostgreSQL en temps réel

- **Modèle de prévision** : régression polynomiale de degré 2 entraînée sur les données historiques des loyers commerciaux, projetée jusqu'en 2035, évaluée avec MAE, MSE, RMSE et R²

- **Dashboard Power BI** : deux pages connectées en direct à PostgreSQL avec mesures DAX personnalisées, graphiques de répartition et visualisation comparative loyer réel vs. prévision marché

---

## Stack technique

### Langages

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat&logo=postgresql&logoColor=white)
![HTML](https://img.shields.io/badge/HTML%2FCSS-E34F26?style=flat&logo=html5&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-F2C811?style=flat&logo=powerbi&logoColor=black)

### Base de données & Backend

| Outil | Usage |
|---|---|
| PostgreSQL | Base de données relationnelle principale |
| pgAdmin | Interface d'administration et import CSV |
| Flask | Backend API pour les boutons de confirmation |
| SQLAlchemy | ORM Python ↔ PostgreSQL |

### Data & Machine Learning

| Bibliothèque | Usage |
|---|---|
| pandas | Nettoyage, transformation et manipulation des données |
| psycopg2 | Connexion Python ↔ PostgreSQL |
| scikit-learn | Régression polynomiale (PolynomialFeatures + LinearRegression) |
| numpy | Calculs numériques et vectorisation |
| matplotlib | Visualisation du modèle de prévision |

### Automatisation & Communication

| Bibliothèque | Usage |
|---|---|
| smtplib + email | Génération et envoi des emails d'alerte |
| datetime | Calcul et comparaison des échéances |
| threading | Exécution concurrente des tâches |
| subprocess | Orchestration des processus |
| OS Scheduler | Planification quotidienne du script |

### Visualisation

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)

---

## Fonctionnalités clés

- **🔔 Alertes multi-niveaux** — Le script détecte automatiquement chaque jour les échéances imminentes et envoie des emails différenciés selon le niveau d'urgence (rappel, alerte retard, fin de contrat), sans intervention manuelle

- **✅ Confirmation en un clic** — Chaque email contient des boutons OUI/NON qui, au clic, appellent un endpoint Flask et mettent à jour la base en temps réel : statut payé, calcul de la prochaine échéance, ou marquage pour résiliation/renouvellement

- **📊 Dashboard temps réel** — Tableau de bord Power BI affichant les montants payés et à payer, la répartition par région et périodicité, les alertes actives, et le détail par agence via des visualisations interactives

- **📈 Prévision stratégique** — Modèle polynomial comparant l'évolution contractuelle des loyers avec les prix du marché prévisionnels jusqu'en 2035, permettant d'anticiper les risques de renégociation avant qu'ils ne surviennent

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                     SOURCES DE DONNÉES                     │
│              Fichier Excel brut (Patrimoine Immo)          │
└──────────────────────────┬────────────────────────────────┘
                           │  Nettoyage + ETL (Python/pandas)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                BASE DE DONNÉES PostgreSQL                  │
│     agence  │  fournisseur  │  contrat  │  paiement       │
└──────┬───────────────────────────────────────┬────────────┘
       │                                       │
       │ psycopg2 (lecture/écriture)           │ Connexion directe
       ▼                                       ▼
┌──────────────────────┐            ┌────────────────────────┐
│   SCRIPT PYTHON      │            │       POWER BI         │
│                      │            │      (Dashboard)       │
│  • Calcul échéances  │            │                        │
│  • Règles d'alerte   │            │  • Vue globale         │
│  • Génération mails  │            │  • Alertes actives     │
│  • Modèle prévision  │            │  • Détail par région   │
└──────────┬───────────┘            └────────────────────────┘
           │
           ▼
┌──────────────────────┐
│    SERVEUR SMTP      │
│   (Envoi emails)     │
│                      │
│  Mail rappel         │
│  Mail retard         │
│  Mail fin contrat    │
└──────────┬───────────┘
           │  Clic bouton OUI/NON
           ▼
┌──────────────────────┐
│    BACKEND FLASK     │──────► UPDATE PostgreSQL
│      (API REST)      │
└──────────────────────┘
