# KBO‑Projet — README

Résumé
-------
Application Symfony pour importer des fichiers CSV KBO et gérer un CRUD d'entreprises (Entreprise + activités, établissements/succursales, dénominations, contacts, etc.). Ce README explique comment configurer l'environnement, initialiser la base, lancer les migrations, importer les CSV et utiliser l'interface CRUD fournie (templates Twig).

Prérequis
---------
- Windows (vous travaillez avec XAMPP)
- XAMPP démarré (Apache et MySQL)
- PHP >= 8.x compatible avec votre projet
- Composer installé
- (optionnel) Symfony CLI pour faciliter le serveur local

Structure importante
-------------------
- src/Entity — entités Doctrine (Entreprise, Establishment, Activity, Denomination, Contact, Category, Branch, Snapshot)
- src/Repository — repositories
- src/Controller — EntrepriseController (CRUD)
- src/Form — EnterpriseType (formulaire)
- src/Services — service d'import CSV (ImporteService / ImportActiviteService)
- src/Command — commandes console d'import
- templates/enterprise — vues Twig pour index/new/show/edit
- config/packages/doctrine.yaml — config Doctrine (DB)
- .env — configuration (DATABASE_URL)

Configuration de la base (XAMPP / MySQL / MariaDB)
-------------------------------------------------
1. Lancer XAMPP (Apache, MySQL).
2. Créer la base et l'utilisateur (exemple, exécuter dans mysql shell ou phpMyAdmin) :

```sql
CREATE DATABASE `kbo_projet` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'kbo'@'127.0.0.1' IDENTIFIED BY 'kbo_password';
GRANT ALL PRIVILEGES ON `kbo_projet`.* TO 'kbo'@'127.0.0.1';
FLUSH PRIVILEGES;
```

3. Mettre à jour `.env` (ex. DATABASE_URL) :
DATABASE_URL="mysql://kbo:kbo_password@127.0.0.1:3306/kbo_projet?serverVersion=mariadb-10.11&charset=utf8mb4"

Important MariaDB vs MySQL
-------------------------
Si vous utilisez MariaDB, forcez la plateforme MariaDB dans `config/packages/doctrine.yaml` :
- ajouter/mettre `server_version: 'mariadb-10.11'` sous `doctrine.dbal` (ou passer serverVersion dans DATABASE_URL).
Sinon DBAL peut lancer des requêtes incompatibles (erreur FULL_COLLATION_NAME).

Installation & initialisation
-----------------------------
Depuis la racine du projet (Windows CMD / PowerShell):

1. Installer dépendances :
   composer install

2. Regénérer l'autoload (si besoin) :
   composer dump-autoload

3. Vider le cache :
   php bin/console cache:clear

4. Vérifier le mapping Doctrine :
   php bin/console doctrine:mapping:info
   php bin/console doctrine:schema:validate

5. Générer la migration (si entités modifiées) :
   php bin/console doctrine:migrations:diff

6. Appliquer les migrations :
   php bin/console doctrine:migrations:migrate

Si `make:migration` / `doctrine:migrations:diff` lève une erreur liée à FULL_COLLATION_NAME, ajustez `server_version` comme indiqué plus haut puis clear cache et réessayez.

Importer les CSV (sans fixtures)
-------------------------------
Le projet contient un service d'import CSV et une commande console (vérifier la commande disponible) :

- Lister les commandes disponibles :
  php bin/console list

- Recherche rapide de commandes liées à l'import :
  php bin/console list | findstr /I import

- Exemple d'exécution (nom de commande à vérifier dans src/Command) :
  php bin/console app:import /chemin/vers/csv_folder

Points importants pour l'import :
- Les CSV fournis doivent être parsés par le service `src/Services/ImporteService.php` (ou ImportActiviteService).
- Lors de l'import : créer d'abord les entités Activity / Category / Code / etc. puis créer les Entreprises en les liant aux Activities existantes.
- L'import ne doit pas passer par AppFixtures — le service peuple directement la BDD.

CRUD (interface + templates)
---------------------------
Routes principales (exemples dans EntrepriseController) :
- /enterprises — liste (index)
- /enterprises/new — création (new)
- /enterprises/{id} — affichage (show)
- /enterprises/{id}/edit — modification (edit)
- /enterprises/{id}/delete — suppression (post)

Templates : `templates/enterprise/` contient :
- index.html.twig
- new.html.twig
- edit.html.twig
- show.html.twig
- _form.html.twig

Relations essentielles
---------------------
- Entreprise (ManyToOne) -> Activity (obligatoire pour création)
- Entreprise (OneToMany) -> Establishment (cascade persist + remove, orphanRemoval=true)
- Denomination, Contact, Branch liés à Entreprise (OneToMany)
- Suppression d'une Entreprise doit supprimer ses établissements (cascade)

Exécution locale
----------------
- Démarrer le serveur Symfony (si installé) :
  symfony serve

- Ou utiliser le serveur PHP intégré (dev uniquement) :
  php -S 127.0.0.1:8000 -t public

Troubleshooting rapide
----------------------
- "Access denied" / 1045 : vérifier que DATABASE_URL user/password/host sont corrects et que l'utilisateur MySQL existe.
- "Column not found FULL_COLLATION_NAME" : configurer server_version en `mariadb-<version>` dans doctrine.yaml ou DATABASE_URL.
- "The autoloader expected class X to be defined..." : vérifier namespace et emplacement du fichier (PSR‑4). Standard recommandé : namespace `App\Service` et dossier `src/Service` (pas `src\Services`).
- Doublon de classe "name is already in use" : supprimer/renommer les fichiers en double et réparer namespace/casse.

Livrables & Documentation attendue
----------------------------------
- Base de données (diagramme ER) — générée depuis entités (exporter diagramme).
- Code backend (entités, controllers, services, templates) — présent dans ce repo.
- Documentation : ce README + notes techniques (justification des relations et choix, instructions d'import).


