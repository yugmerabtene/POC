# 📘 Cahier des charges – PoC Auth en microservice (conforme ISO UE)

## 🎯 Objectif
Mettre en place un **microservice Auth** pour gérer l’inscription et la connexion des entreprises, avec une vérification du numéro SIRET via l’API du gouvernement, des normes de sécurité élevées (conformité ISO), et un frontend React.js.

---

## 🏗️ Architecture technique

| Élément          | Technologie                       |
|------------------|------------------------------------|
| Frontend         | React.js                          |
| Backend Auth     | Node.js (Express) **ou** FastAPI  |
| Base de données  | MySQL                             |
| Communication    | REST API                          |
| Sécurité         | JWT, bcrypt, OTP, CORS, CSRF      |

---

## 📋 Fonctionnalités du service Auth

### 🔐 Inscription
- Champs requis :
  - Email
  - Mot de passe (conforme ISO)
  - Nom
  - Prénom
  - Nom de l'entreprise
  - Numéro de SIRET (vérifié via API INSEE/Sirene)
- Vérification par regex
- Stockage sécurisé des données (hash + chiffrement)
- Envoi d’un email de confirmation avec token

### 🔑 Connexion
- Email + mot de passe
- Vérification du token email
- 2FA : OTP envoyé par mail
- Authentification via JWT avec durée limitée

### ⚙️ Autres routes
- `GET /me` – Données utilisateur connecté
- `DELETE /me` – Suppression de compte
- `POST /verify-email/:token` – Vérification email
- `POST /verify-otp` – Validation 2FA

---

## 🔎 Validation des champs

| Champ            | Regex                                                      |
|------------------|------------------------------------------------------------|
| Email            | `/^[\w-.]+@([\w-]+\.)+[\w-]{2,4}$/`                        |
| Mot de passe     | `/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[\W_]).{12,}$/`     |
| Prénom/Nom       | `/^[A-Za-zÀ-ÿ '-]{2,50}$/`                                 |
| SIRET            | `/^\d{14}$/`                                               |

---

## 💾 Modèle de base de données (MySQL)

```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  company_name VARCHAR(255) NOT NULL,
  siret VARCHAR(14) UNIQUE NOT NULL,
  email_verified BOOLEAN DEFAULT FALSE,
  verification_token VARCHAR(255),
  two_factor_enabled BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## ✅ Conformité ISO – Implémentations & Documentation

### 🔐 ISO/IEC 27001 – Sécurité de l'information (SMSI)

| Exigence                         | Implémentation dans le PoC                        |
|----------------------------------|----------------------------------------------------|
| Stockage sécurisé                | bcrypt/argon2 pour le hash de mot de passe        |
| Contrôle d'accès                 | JWT, TTL, restriction des rôles                   |
| Authentification forte           | 2FA par OTP via email                             |
| Journalisation                   | Logs des connexions, tentatives, timestamps       |
| Protection contre attaques       | CORS, CSRF, XSS, Rate limiting, Helmet.js         |
| Tests de vulnérabilité           | OWASP Top 10 intégré dans les bonnes pratiques    |

### 🧾 ISO/IEC 29115 – Authentification d'identité

| Niveau de garantie | Mise en œuvre dans le PoC                               |
|---------------------|----------------------------------------------------------|
| Niveau 1            | Email + token de vérification                            |
| Niveau 2            | Vérification SIRET via API INSEE                         |
| Niveau 3            | Authentification + OTP email, suivi des connexions       |

> 💡 Niveau 4 non implémentable dans un PoC sans eID ou biométrie.

### ☁️ ISO/IEC 27018 – Données personnelles dans le cloud

| Exigence                         | Implémentation dans le PoC                        |
|----------------------------------|----------------------------------------------------|
| Consentement et finalité         | Notice lors de l'inscription                      |
| Accès restreint aux données      | Endpoints limités, pas de données sensibles exposées |
| Protection des données en transit| HTTPS requis (à documenter même en dev)           |
| Droit à l'effacement             | Route DELETE `/me`                                |
| Stockage UE sécurisé             | Mention de l’hébergement dans un cloud européen   |

---

## 🧾 Résumé ISO – Implémenter et Documenter

| Norme         | Implémenter dans le code                   | Documenter dans le README / PoC |
|---------------|--------------------------------------------|----------------------------------|
| ISO 27001     | Authentification forte, hash, logs, 2FA, JWT, sécurité OWASP | Politique de sécurité, mesures de protection |
| ISO 29115     | Vérification SIRET, Email token, 2FA       | Niveau d'identité (Niveau 2 ou 3) |
| ISO 27018     | Consentement, suppression compte, API minimaliste, chiffrement en transit | Localisation des données, sécurité des PII |

---

## ✉️ Système de vérification par mail

- Utilisation d'un SMTP sécurisé (ex : Gmail OAuth2 ou Mailgun)
- Envoi :
  - Mail de confirmation avec lien `/verify-email/:token`
  - Mail OTP pour 2FA (code temporaire)
- Le token doit être signé, horodaté, et limité dans le temps

---

## 🔐 Sécurité générale

- ✅ Regex front + back
- ✅ JWT TTL (ex : 15 min) + Refresh token (optionnel)
- ✅ Rate limiter sur `/login`
- ✅ Helmet.js, CORS, CSRF
- ✅ ORM (Sequelize / Prisma / SQLAlchemy) → protection injections SQL

---

## 🧪 Tests

- Tests unitaires sur :
  - Validation des entrées
  - Hash mot de passe
  - Génération + vérification token
- Tests e2e via Postman (collection fournie)
- Documentation OpenAPI (Swagger)
