# Épreuve finale - 420-4D2-MA - Hiver 2025

**Note pour la postérité : l'examen était trop long pour 3 heures, plusieurs élèves n'ont pas fini.**

- **Durée** : 3 heures.
- **Modalité** : Individuelle, en classe sous la supervision de l’enseignant.
- **Documentation** : Notes de cours, vos travaux pratiques, documentation officielle des différentes bibliothèques utilisées.
- **Intelligence artificielle** : Interdite.
- **Remise** : À la fin de l’épreuve, vous devez remettre votre projet sur Omnivox. Évitez d'inclure `node_modules`.

## 🎯 Objectif

Vous allez développer une API REST pour un système RH de candidatures. Deux types
d’utilisateurs peuvent interagir avec le système :

- **Candidats** : peuvent consulter les offres et postuler.
- **RH (administrateurs)** : peuvent ajouter des offres (manuellement ou par CSV), voir toutes les candidatures, consulter des statistiques.

> [!IMPORTANT]  
> Vous devez suivre les étapes ci-dessous dans l’ordre. Par exemple, **ne faites
> pas** toutes les routes avant de commencer à écrire des tests (ex.
> `request.http`), suivez les étapes une à une.

> [!NOTE]  
> Par défaut, si l'énoncé indique "champs requis", cela signifie que le corps de la
> requête doit être au format JSON et contenir les champs requis.

---

## ✅ 1. Mise en place du projet

- Créez un projet Node.js.
- Le projet doit pouvoir s'installer avec `npm install`.
- Le projet doit pouvoir se lancer avec `npm run dev`.
- Créez une structure claire du projet au fur et à mesure que vous avancez. Votre code ne doit pas être entièrement dans le fichier `index.js`.
- Créez un fichier `request.http` pour tester votre API. Ce fichier sera utilisé à chaque étape.

### Base de données

- Utilisez Knex et SQLite pour la base de données.
- Votre code doit pouvoir créer la base de données et les tables automatiquement.
- Vous pouvez concevoir la base de données de façon incrémentale, au fur et à mesure que vous développez les fonctionnalités.

---

## 🔐 2. Authentification

> [!NOTE]  
> Un utilisateur administrateur doit déjà être configuré dans la base. Utilisez
> son compte pour tester les routes RH. Identifiants par défaut : `email: admin@rh.com`, `password: admin123`

### Routes à créer

#### POST `/auth/register`

* Champs requis :

  * `name` (obligatoire, non vide)
  * `email` (obligatoire, format valide)
  * `password` (min. 6 caractères)
  * `role` : toujours `"user"` (le rôle `"admin"` est réservé et ne peut pas être attribué à l'inscription)

* Réponse : `201 Created` avec `{ id, name, email, role }`

#### POST `/auth/login`

* Champs requis :
  * `email` (obligatoire, format valide)
  * `password` (min. 6 caractères)

* Réponse : `200 OK` et retourne un token JWT.

#### GET `/auth/me`

* JWT requis dans l’en-tête `Authorization: Bearer <token>`
* Réponse : infos de l'utilisateur connecté : `{ id, name, email, role }`

### Test requis : `request.http`

* Créer correctement un utilisateur
* Créer incorrectement un utilisateur
* Tester le login et sauvegarder les tokens (admin et user)
* Tester le `GET /auth/me` avec le token
* Tester le `GET /auth/me` sans token

---

## 📄 3. Gestion des offres

Une offre est un poste à pourvoir. Une offre est constituée de :
- `title` : titre de l’offre
- `description` : description de l’offre
- `deadline` : date limite de candidature (format ISO `YYYY-MM-DD`)

### Routes à créer

#### POST `/offers` (admin uniquement)

* Champs requis :

  * `title` (obligatoire, min. 5 caractères)
  * `description` (obligatoire, min. 10 caractères)
  * `deadline` (date future, format ISO `YYYY-MM-DD`)

* Réponse : `201 Created` avec l’offre ajoutée

#### GET `/offers`

* Accessible à tous les utilisateurs connectés
* Retourne un fichier CSV avec toutes les offres

#### GET `/offers/:id`

* Détail d’une offre spécifique sous forme JSON

### Test requis : `request.http`

* Création d’offres valides et invalides
* Connexion en tant que user
* Consultation d’une offre spécifique
* Téléchargement du fichier CSV avec toutes les offres

---

## 📁 4. Import d’offres par CSV

### Route à créer

#### POST `/offers/upload` (admin uniquement)

* Format CSV attendu :

  ```csv
  title,description,deadline
  Développeur Web,Développement front et back,2025-06-30
  Analyste QA,Tests automatisés,2025-07-15
  ```

* Réponse : `{ inserted: <nombre> }`

### Test requis : `request.http`

* Upload d’un fichier CSV valide.
* Upload d’un fichier CSV invalide.
* Vérification des offres importées via `GET /offers`

---

## ✉️ 5. Candidature des utilisateurs

### Route à créer

#### POST `/applications` (user uniquement)

* Champs requis :

  * `offer_id` (doit correspondre à une offre existante)
  * `cv` (min. 20 caractères)
  * `motivation` (min. 20 caractères)

* Réponse : `201 Created` avec la candidature

Votre application doit enregistrer dans la base de données la date à laquelle la candidature a été soumise.

#### GET `/applications` (user uniquement)

* Retourne uniquement les candidatures de l'utilisateur connecté

### Test requis : `request.http`

* Soumission d’une candidature valide et invalide
* Consultation de ses candidatures

---

## 🛠 6. Interface RH : voir les candidatures

### Routes à créer (admin uniquement)

#### GET `/admin/applications`

* Retourne un fichier contenant toutes les candidatures **au format CSV** avec :

  * nom du candidat
  * titre de l’offre
  * date de candidature

#### GET `/admin/applications/:id`

* Retourne le détail d’une candidature

#### GET `/admin/stats`

* Retourne un fichier de statistiques d’utilisation **au format JSON** :

  ```json
  [
    { "offer_title": "Développeur Web", "applications": 5 },
    { "offer_title": "Analyste QA", "applications": 3 }
  ]
  ```

### Test requis : `request.http`

* Consultation des candidatures et des statistiques

---

## 📝 Barème 

Voici le barème de l'évaluation qui provient du plan de cours.

| Critère d’évaluation                                                                 | Pondération |
|:-------------------------------------------------------------------------------------|:-----------:|
| Installer l'application ainsi que ses dépendances                                    |    10%      |
| Les demandes sont programmées et les attentes quant aux fonctionnalités demandées sont remplies |    30%      |
| Le code source est produit selon les standards                                       |    30%      |
| L'étudiant a utilisé les techniques appropriées (fonctions, méthodes, composants, modules, etc.) pour arriver à ses fins |    30%      |

Bonne chance 🚀
