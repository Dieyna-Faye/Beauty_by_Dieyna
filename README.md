# Maison Naya — boutique en ligne

Boutique à vendeur unique construite avec Laravel 12, React 19, TypeScript et MySQL. Le parcours client est en français et adapté aux téléphones.

## Fonctionnalités

- Catalogue public filtrable par catégorie, fiches produits, photos, options et stock.
- Panier conservé dans le navigateur, modification des quantités et suppression d’articles.
- Commande invitée avec coordonnées et adresse de livraison.
- Frais déterminés depuis la zone sélectionnée et calculés à nouveau par le serveur.
- Réservation concurrente du stock, montants issus du catalogue serveur, suivi privé par jeton aléatoire et protection idempotente contre les doubles envois.
- Espace vendeuse protégé : produits et photos, catégories, zones, commandes, adresse de livraison, suivi des statuts et historique.
- Paiement Wave : création de session Checkout, redirection, vérification HMAC des webhooks, idempotence des événements et rapprochement du montant, de la devise et de la commande.
- Orange Money apparaît dans le choix de paiement, mais son API Web Payment n’est pas encore intégrée : elle est réservée aux marchands agréés et son accès/API Sénégal doit être fourni par Orange.

Aucun produit ni tarif de livraison n’est préchargé. Ils se configurent dans l’espace vendeuse pour éviter d’afficher des prix ou zones inventés. Une commande ne peut pas être créée sans zone de livraison active.

## Installation locale (Laragon / Windows)

1. Dans MySQL ou HeidiSQL, créez la base :

   ```sql
   CREATE DATABASE e_commerce CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

2. Configurez Laravel :

   ```powershell
   cd C:\laragon\www\e-commerce\backend
   Copy-Item .env.example .env
   ```

   Modifiez `backend/.env` : renseignez `DB_DATABASE`, `DB_USERNAME` et `DB_PASSWORD`, puis définissez un `ADMIN_PASSWORD` robuste. L’administratrice est créée par `db:seed` uniquement si cette variable est renseignée.

3. Installez et initialisez le backend :

   ```powershell
   & 'C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe' 'C:\laragon\bin\composer\composer.phar' install
   & 'C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe' artisan key:generate
   & 'C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe' artisan migrate --seed
   & 'C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe' artisan storage:link
   & 'C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe' artisan serve --host=127.0.0.1 --port=8000
   ```

4. Dans un second terminal, lancez React :

   ```powershell
   cd C:\laragon\www\e-commerce\frontend
   npm install
   npm run dev
   ```

   Boutique : `http://localhost:5173` · administration : `http://localhost:5173/admin`.

## Configuration

Les modèles se trouvent dans `backend/.env.example`. Les valeurs sensibles restent dans `backend/.env`, exclu du contrôle de version.

- `DB_*` : connexion MySQL.
- `ADMIN_NAME`, `ADMIN_EMAIL`, `ADMIN_PASSWORD` : compte vendeuse initial. Après modification de ces valeurs, relancez `php artisan db:seed`.
- `APP_URL` : adresse publique HTTPS du backend en production.
- `APP_FRONTEND_URL` : adresse publique HTTPS du frontend ; utilisée pour les retours Wave.
- `WAVE_API_KEY` : clé API Wave Business Checkout, gardée côté serveur.
- `WAVE_WEBHOOK_SECRET` : secret HMAC associé au webhook enregistré dans Wave Business Portal. Obligatoire pour démarrer Wave dans cette version.
- `WAVE_REQUEST_SIGNING_SECRET` : facultatif, si la signature des requêtes API est activée sur la clé Wave.
- `ORANGE_MONEY_CLIENT_ID`, `ORANGE_MONEY_CLIENT_SECRET`, `ORANGE_MONEY_WEBPAY_URL` : réservés à une future intégration. Ils ne rendent pas Orange Money opérationnel actuellement.
- `VITE_API_URL` et `VITE_STORAGE_URL` dans `frontend/.env` : facultatifs ; leurs valeurs locales par défaut pointent vers Laravel.

Créez les zones avec leurs vrais tarifs et activez-les dans **Administration → Livraison**. Ajoutez ensuite catégories et produits dans l’espace vendeuse.

## Paiements marchands

### Wave

L’intégration appelle l’API Checkout officielle côté serveur. Il faut un compte Wave Business, une clé API autorisée à créer les sessions Checkout, un secret de webhook, une adresse HTTPS publique et enregistrer `https://<domaine-backend>/api/v1/webhooks/wave` dans le portail Wave pour les événements Checkout. Le site ne marque une commande payée qu’après réception d’un webhook signé qui correspond à la session, au numéro de commande, au montant et à `XOF`.

La configuration de compte marchand et les identifiants restent à fournir. Sans clé/secret, les commandes restent en attente et aucun paiement n’est simulé.

### Orange Money

Orange indique que le Web Payment est destiné aux marchands Orange Money agréés, disponibles au Sénégal, et que l’intégration/API est fournie après souscription. Le portail expose une documentation restreinte. Il reste à obtenir l’approbation marchande, les identifiants et la référence technique officielle Sénégal (initialisation, retour, vérification et notification) avant d’activer le connecteur. Le site échoue volontairement de manière sûre et ne prétend pas qu’un paiement Orange a été lancé.

Références officielles : [Wave Checkout API](https://docs.wave.com/checkout), [Wave Webhooks](https://docs.wave.com/webhook), [Orange Money Web Payment](https://developer.orange.com/apis/om-webpay), [FAQ Orange Money](https://developer.orange.com/apis/om-webpay/faq).

## Tests et vérification

```powershell
cd C:\laragon\www\e-commerce\backend
& 'C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.exe' artisan test

cd C:\laragon\www\e-commerce\frontend
npm run build
```

Les tests Laravel couvrent le recalcul serveur des montants, panier vide, nom et téléphone invalides, adresse incomplète, produit désactivé, stock insuffisant, double envoi, webhook Wave invalide, confirmation signée et échec de paiement. Les tests utilisent SQLite en mémoire et ne déclenchent aucun paiement réel. Le build frontend vérifie TypeScript et la compilation Vite.

MySQL doit être démarré dans Laragon et la base créée avant d’exécuter les migrations locales. Pour le paiement réel, déployez sur HTTPS et configurez les comptes marchands avant toute commande commerciale.
