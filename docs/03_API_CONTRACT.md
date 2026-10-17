# 03 — Contrat API documentaire

Statut : inventaire fidèle des routes, pas un contrat de payload final ni un fichier OpenAPI. Source : [PDF](reference/Rihla_MR_Conception_Fonctionnelle_Technique.pdf) p. 8–11 ; compléments de sécurité imposés par B. Les noms de paramètres ci-dessous restent ceux du PDF.

## 1. Conventions transversales

- API REST avec les méthodes et chemins recensés ; aucun préfixe `/v1` ajouté. Préparer le versionnement si des clients mobiles tiers apparaissent.
- DTO explicites, aucune entité JPA directement exposée. Propriétés exactes, types, champs obligatoires, exemples et schémas : **À définir** pour tous les endpoints.
- Erreurs structurées avec code métier, message localisable et `requestId` ; noms exacts des autres propriétés, statut HTTP par cas et format : **À définir**. `SEAT_UNAVAILABLE` est le seul code métier explicitement nommé par le PDF. Aucun catalogue supplémentaire n'est présenté comme acquis.
- Corrélation par `requestId` entre réponse d'erreur et logs ; génération, propagation et éventuel header : **À définir**. Journaux sans secrets ni données personnelles complètes.
- `Idempotency-Key` explicitement requis pour POST public bookings et, selon p. 9, pour les écritures critiques. Portée tenant/opération/client, durée, stockage, rejeu d'un résultat, comportement sur payload différent et requêtes simultanées : **À définir** avant chaque écriture critique. Déduplication du webhook par événement fournisseur distincte de cette clé client.
- Pagination standardisée des listes opérateur/admin ; paramètres, limites, tri et enveloppe : **À définir**. Pagination de recherche publique et du manifeste à arbitrer.
- Authentification opérateur : access token court, refresh token HttpOnly avec rotation ; Argon2 ou BCrypt paramétré ; MFA recommandé pour COMPANY_OWNER et PLATFORM_ADMIN (p. 11). Durées, transports, protection CSRF/CORS et choix de hash : **À définir** en Phase 1.
- Autorisation privée : RBAC **et** appartenance de la ressource au tenant ; ne jamais faire confiance à un company_id client. Matrice exhaustive par rôle/action, y compris support et administration transversale : **À définir**.
- Endpoints publics sensibles : protection contre énumération/abus, rate limiting par IP/téléphone/compte selon contexte, minimisation des données et erreurs non bavardes. **Une référence de réservation seule ne prouve jamais le droit d'accéder aux données personnelles ni d'annuler.** Mécanisme de preuve supplémentaire : **À définir**, sans choix imposé ici.
- Validation serveur des entrées, identifiants, formats, références, bornes, permissions et transitions ; Spring Validation prévu. Formats/dates/fuseaux/devise/limites : **À définir**. Les données publiques sur une compagnie ne rendent pas ses données opérationnelles publiques.

Profils utilisés dans les tableaux : **PUB** = lecture publique limitée aux données publiables ; **RES** = protection d'accès à une réservation sans compte à définir ; **OP** = authentification + permission métier + tenant + audit des actions sensibles ; **ADM** = permission plateforme explicite + audit, étendue à définir. Ces profils sont des raccourcis documentaires, pas des rôles supplémentaires.

## 2. API publique — PDF p. 8

Pour chaque ligne, requête/réponse exactes et codes HTTP sont **À définir** ; les informations connues sont des besoins sémantiques, pas des noms de propriétés inventés.

| Méthode et route | Responsabilité / informations connues | Sécurité et invariants | Détails restant à spécifier |
|---|---|---|---|
| GET `/api/public/cities` | Référentiel des villes | PUB ; aucune donnée privée | Champs ville, localisation FR/AR, tri, filtres : À définir |
| GET `/api/public/trips/search` | Recherche origine, destination, date, passagers ; comparaison des départs | PUB ; validation critères ; disponibilité issue du serveur | Noms/types des paramètres, filtres, pagination, statuts publiables et capacité de groupe : À définir |
| GET `/api/public/trips/{tripId}` | Détail d'un départ ; trajet, agence, véhicule, équipements, politique, prix attendus par les écrans | PUB ; filtrage des informations publiables | DTO détail, champs optionnels et comportement sur trajet non publié : À définir |
| GET `/api/public/trips/{tripId}/seats` | Plan/disponibilité des sièges du trajet | PUB ; aucune identité de passager ; état affiché non garanti à l'achat | Représentation du plan, états exposés, fraîcheur : À définir |
| POST `/api/public/bookings` | Réservation sans compte, contact téléphone, passagers et choix de sièges ; `Idempotency-Key` explicite | Validation et anti-abus ; INV-01/02/06/09 ; `SEAT_UNAVAILABLE` en conflit | DTO, réponse de hold, preuve d'accès délivrée, règles multi-sièges, idempotence et HTTP de conflit : À définir |
| GET `/api/public/bookings/{reference}` | Consultation sécurisée | RES ; référence insuffisante ; anti-énumération/rate limiting ; données minimales | Preuve d'accès, récupération, contenu exposé, erreurs indiscernables si retenues : À définir |
| POST `/api/public/bookings/{reference}/cancel` | Annulation selon politique compagnie, remboursement éventuel | RES ; vérification état/politique ; écriture critique idempotente à contractualiser | DTO, politique, délais/frais, réponse et coordination remboursement : À définir |

## 3. API opérateur — PDF p. 8–9

Les rôles cités indiquent les responsabilités de p. 3 ; leur permission exacte n'est pas un contrat d'autorisation déjà validé. Tous les payloads et codes HTTP sont **À définir**.

| Méthode et route | Responsabilité / informations connues | Sécurité et invariants | Détails restant à spécifier |
|---|---|---|---|
| POST `/api/operator/routes` | Créer une route ; origine, destination, durée, distance dans le domaine | OP ; propriétaire/responsable selon matrice ; tenant des relations | DTO, validations géographiques, doublons : À définir |
| POST `/api/operator/trips` | Créer un départ ; route, véhicule, agences, horaires, prix, statut | OP ; cohérence tenant ; inventaire cohérent | DTO, état initial, création inventaire et idempotence : À définir |
| PATCH `/api/operator/trips/{id}` | Modifier un trajet | OP ; transitions autorisées ; protéger réservations existantes | Format PATCH, champs modifiables, version/conflits et effets retard/annulation : À définir |
| GET `/api/operator/trips/{id}/manifest` | Manifeste passagers | OP ; agent d'embarquement dans son périmètre ; données personnelles minimales | Champs, filtres, pagination, affectation de l'agent : À définir |
| POST `/api/operator/bookings/counter` | Vente guichet, cash de première classe | OP ; agent habilité ; inventaire partagé ; idempotence ; confirmation cash autorisée | DTO, point de validation cash, rattachement agence/acteur et réponse : À définir |
| POST `/api/operator/tickets/{ticketId}/check-in` | Enregistrer le check-in | OP ; signature, billet, trajet, statut vérifiés ; billet issu d'une réservation confirmée | Données de scan, transitions et contrat de répétition : À définir |
| POST `/api/operator/tickets/{ticketId}/board` | Enregistrer l'embarquement | OP ; contrôle tenant/trajet/statut ; deuxième utilisation interdite, même concurrente | DTO, gardes, idempotence d'une retransmission versus refus d'une réutilisation : À définir |
| GET `/api/operator/dashboard` | Départs, ventes, remplissage, CA | OP ; agrégats limités au tenant et aux droits | Périodes, fuseau, calculs cash/remboursements, filtres et DTO : À définir |

## 4. API administration — PDF p. 9

| Méthode et route | Responsabilité / informations connues | Sécurité | Détails restant à spécifier |
|---|---|---|---|
| GET `/api/admin/companies` | Lister les compagnies | ADM ; étendue d'accès explicite | DTO, pagination, filtres/statuts/tri : À définir |
| POST `/api/admin/companies/{id}/approve` | Approuver une compagnie, statut APPROVED | ADM ; action sensible auditée | Prérequis licence, transition, répétition et réponse : À définir |
| POST `/api/admin/companies/{id}/suspend` | Suspendre une compagnie, statut SUSPENDED | ADM ; action sensible auditée | Motif, effets sur ventes/trajets/réservations, idempotence : À définir |
| GET `/api/admin/audit` | Consulter l'audit | ADM ; accès restreint, minimisation des métadonnées | DTO, pagination, filtres, rétention et restrictions support : À définir |

Les schémas de requête/réponse et codes HTTP de ces quatre routes restent **À définir**.

## 5. Webhook explicitement défini — PDF p. 10

| Méthode et route | Responsabilité / informations connues | Sécurité | Détails restant à spécifier |
|---|---|---|---|
| POST `/api/webhooks/payments/{provider}` | Vérifier l'événement, mettre à jour Payment, confirmer Booking si applicable ; réponse 2xx même si événement déjà traité | Signature/HMAC selon fournisseur, timestamp, identité unique `providerEventId`, anti-replay ; résolution sûre de l'opération/tenant ; traitement idempotent | Payload, headers, signature/canonicalisation, fenêtre temporelle, secrets/rotation, erreurs, événements désordonnés/tardifs, portée de l'identité : À définir en Phase 8 |

Cette route fait partie de l'inventaire du PDF même si elle n'apparaît pas dans ses tableaux de la section 6. Le 2xx de déduplication ne dispense jamais d'authentifier l'événement. L'articulation entre livraison répétée légitime et fenêtre anti-replay doit être contractualisée avec le fournisseur.

## 6. Capacités sans endpoints définis dans le PDF

Aucun chemin supplémentaire n'est fixé ici. À concevoir dans la phase concernée : connexion/refresh/déconnexion et gestion des employés ; onboarding compagnie ; gestion/liste/détail des agences, véhicules, plans et sièges ; maintenance City/Station ; liste et gestion complémentaire routes/trajets ; blocage/déblocage inventaire ; éventuelle confirmation cash séparée de la vente ; accès au billet/QR ; initiation et vérification du paiement, remboursement ; accès support ; gestion des politiques et tarifs ; notifications si retenues. Les endpoints CRUD partiels du PDF ne suffisent pas à couvrir tous ces besoins.

L'expiration des holds peut être interne et ne requiert pas nécessairement un endpoint public. Health/readiness Actuator relève de l'exploitation ; exposition et sécurité à définir, sans route ajoutée au présent contrat métier.

## 7. Conditions de finalisation par phase

Avant implémentation d'une route : approuver les DTO et statuts HTTP, permissions et périmètres, règles de validation et erreurs, contrats d'idempotence si critique, puis tests positifs/négatifs. Ne pas bloquer Phase 1 sur des payloads de Phase 8 ; résoudre chaque décision avant sa phase limite ([DEC-03/04/14](04_IMPLEMENTATION_PLAN.md#4-registre-des-décisions-ouvertes)). Aucun contrôleur, client frontend ou OpenAPI n'est produit en Phase 0.
