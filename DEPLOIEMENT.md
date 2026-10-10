# Déployer Ticket d'Or — guide pas à pas

## Important avant la mise en ligne
Cette version contient le catalogue, l'administration, la création/modification/publication d'événements et l'enregistrement de demandes de réservation dans PostgreSQL. Le paiement Orange Money/MTN Money/Wave n'est PAS connecté : les commandes restent en attente, aucun débit n'est lancé et aucun billet QR valide n'est émis. Il faut un compte marchand/prestataire et ses identifiants API pour terminer cette partie. Ne publiez pas la billetterie comme si les paiements étaient actifs.

## 1. Créer la base de données Supabase
1. Ouvre https://supabase.com/ et crée un compte/projet.
2. Crée un projet PostgreSQL et conserve le mot de passe de base de données en lieu sûr.
3. Dans les paramètres de connexion, récupère une chaîne PostgreSQL compatible avec Prisma. Pour le premier schéma, utilise la connexion directe/session indiquée par Supabase, pas une URL de navigateur.
4. Ne partage jamais cette chaîne : elle contient un secret.

## 2. Préparer le code
1. Télécharge et décompresse `ticket-dor-v5.zip`.
2. Installe Node.js 20 ou plus récent depuis https://nodejs.org/.
3. Ouvre un terminal dans le dossier `ticket-dor-v4`.
4. Exécute `npm install`.
5. Copie `.env.example` vers `.env` et remplace `DATABASE_URL` par la vraie chaîne de connexion.
6. Exécute `npx prisma db push` pour créer les tables dans la base choisie.
7. Exécute `npm run setup:admin` et choisis un nouveau mot de passe unique (ne réutilise pas le mot de passe partagé dans la conversation).
8. Lance `npm run dev` et ouvre http://localhost:3000. Administration : http://localhost:3000/admin/login.

## 3. Mettre le code sur GitHub
1. Ouvre https://github.com/ et crée un dépôt privé nommé `ticket-dor`.
2. Depuis le terminal du projet, exécute :
   - `git init`
   - `git add .`
   - `git commit -m "Préparer Ticket d'Or au déploiement"`
   - `git branch -M main`
   - `git remote add origin URL_DU_DEPOT_GITHUB`
   - `git push -u origin main`
3. Vérifie que `.env` n'est PAS dans GitHub. Le fichier est ignoré par `.gitignore`. Ne force jamais son ajout.

## 4. Publier sur Vercel
1. Ouvre https://vercel.com/ et connecte ton dépôt GitHub.
2. Choisis le dépôt `ticket-dor` et le framework Next.js.
3. Dans les paramètres du projet, ajoute ces variables pour les environnements Production, Preview si nécessaire, et Development si tu les utilises :
   - `DATABASE_URL` : chaîne PostgreSQL de Supabase
   - `APP_URL` : URL Vercel de ton site, sans slash final
   - `ADMIN_EMAIL` : `boniakelicoty@gmail.com`
   - `ADMIN_PASSWORD_HASH` : valeur du hash généré dans le fichier `.env` par `npm run setup:admin`
   - `SESSION_SECRET` : secret généré dans `.env` (garde-le confidentiel)
4. Commande de build : `npm run build` (elle exécute Prisma Generate puis Next.js Build).
5. Lance le déploiement. Si la compilation échoue, ouvre les logs Vercel et corrige l'erreur avant d'utiliser le site.
6. Une fois la base configurée, vérifie la page d'accueil, `/admin/login`, la création d'un événement test, sa publication et une réservation test.

## 5. Préparer les paiements réels
Avant d'activer les paiements, il faut choisir un prestataire qui prend en charge les moyens voulus en Côte d'Ivoire et créer/valider le compte marchand. Demande au prestataire ses clés API de test, sa documentation et la procédure webhook. Après intégration, ajoute les clés uniquement dans les variables d'environnement Vercel (`PAYMENT_PROVIDER`, `PAYMENT_API_KEY`, `PAYMENT_WEBHOOK_SECRET`) et teste les paiements en environnement sandbox. Ne collecte jamais le code secret Mobile Money du client. La confirmation doit venir du serveur via un webhook vérifié, pas de la page de retour du navigateur.

## 6. Billets QR et domaine
Le QR code doit contenir un jeton aléatoire unique et être créé seulement après confirmation serveur du paiement. Le scan à l'entrée devra vérifier la base de données et marquer le billet utilisé pour empêcher une seconde entrée. Cette fonction n'est pas encore activée dans cette version.

Quand le site et les paiements sont testés, achète un domaine chez un registrar et ajoute-le dans les paramètres Domains de Vercel. `ticketdor.ci` est seulement une suggestion : sa disponibilité n'a pas été vérifiée. Configure ensuite `APP_URL` avec le domaine final.

## Problèmes courants
- `PrismaClientInitializationError` : vérifie `DATABASE_URL` et que la base accepte les connexions.
- Connexion admin refusée : vérifie `ADMIN_EMAIL`, `ADMIN_PASSWORD_HASH` et `SESSION_SECRET` dans Vercel, puis redéploie.
- Événements absents : vérifie que les tables sont créées et que les événements ont le statut publié et une date future.
- Ne mets jamais un secret dans un fichier HTML ou une variable préfixée `NEXT_PUBLIC_`.
