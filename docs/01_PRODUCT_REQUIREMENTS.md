# 01 — Exigences produit

Statut : cadrage Phase 0 rédigé, soumis à validation humaine. Aucune fonctionnalité implémentée.

Sources : [PDF de conception v1.0, octobre 2026](reference/Rihla_MR_Conception_Fonctionnelle_Technique.pdf), lu intégralement (14 pages), et instructions de développement Phase 0 du 17 octobre 2026 (« source B »). Les pages citées sont celles du PDF. Les règles métier du PDF sont conservées ; l'ordre des phases suit B. Une mention « À définir » n'autorise pas à inventer une politique.

## 1. Vision et parties prenantes

Rihla MR relie voyageurs, compagnies, agences et agents d'embarquement en Mauritanie. Ce produit B2B2C fournit aux voyageurs la comparaison et la réservation, et aux compagnies un outil quotidien de programmation, vente et embarquement (PDF p. 3–4).

Les voyageurs peinent à comparer horaires, tarifs, disponibilité et points de départ. Les ventes dispersées entre web, téléphone et guichet exposent les compagnies à la double vente ; les agents manquent d'un manifeste et d'un suivi centralisés. La proposition de valeur est un inventaire unique, fiable et partagé entre tous les canaux, avec isolation entre compagnies.

Parties prenantes : voyageurs, propriétaires et responsables de compagnie, agences, agents de vente et d'embarquement, administration centrale et support ; fournisseurs de paiement et autorités/conseil local pour les intégrations et la commercialisation. Le téléphone est un canal cité par le PDF, mais son parcours opérationnel et ses droits restent à définir.

## 2. Rôles (PDF p. 3)

| Rôle | Responsabilités confirmées | Limites à préciser |
|---|---|---|
| PASSENGER | Recherche, réservation, paiement, consultation, annulation | Réservation sans compte ; mécanisme de consultation sécurisé ouvert |
| COMPANY_OWNER | Administration complète de sa compagnie | MFA recommandé ; ne confère pas d'accès aux autres compagnies |
| COMPANY_MANAGER | Routes, trajets, véhicules, tarifs, personnel, rapports | Matrice précise des permissions à définir |
| AGENCY_AGENT | Vente guichet, encaissement cash, gestion des réservations | Périmètre agence et délégation cash à définir |
| BOARDING_AGENT | Manifeste, scan QR, check-in, embarquement | Affectations aux agences/trajets à définir |
| PLATFORM_ADMIN | Validation compagnies, supervision, sécurité, audit | Accès transversal explicitement autorisé et audité |
| SUPPORT_AGENT | Support limité aux voyageurs et opérateurs | Périmètre, masquage et actions autorisées à définir ; aucun droit global implicite |

RBAC **et** contrôle de l'appartenance au tenant sont requis pour chaque ressource opérateur. Les permissions détaillées ne se déduisent pas du seul nom du rôle.

## 3. Périmètre MVP et différé

| Domaine | MVP attendu | Phases actualisées |
|---|---|---|
| Voyageur | FR/AR, RTL, mobile-first/PWA ; recherche origine, destination, date, passagers ; comparaison horaire, prix, compagnie, disponibilité ; détail et plan de sièges ; réservation sans compte avec téléphone ; cash ou paiement simulé ; consultation sécurisée, annulation configurable ; billet QR | 1, 4, 5, 6, 7, 8 |
| Compagnie/agence | Authentification employés ; agences, véhicules, plans, routes, tarifs, trajets ; ventes guichet sur inventaire commun ; manifeste, scan et embarquement ; dashboard départs, ventes, remplissage, CA | 1 à 3, 6, 7 ; dashboard proposé en 6 |
| Administration | Liste, approbation et suspension des compagnies ; supervision et audit ; support limité | Contrôles dès 1 ; compagnie en 2 ; supervision complétée jusqu'à 10 |
| Notifications | Besoins du PDF p. 10 : confirmation, rappels 24 h/2 h, retard/changement de départ, annulation, remboursement ; fournisseurs simulables derrière NotificationProvider | Planification actualisée en 9, conditionnelle ; couverture MVP à arbitrer (DEC-18) |

Les écrans de référence sont accueil, résultats/filtres, détail (agence, véhicule, équipements, politique, prix), plan (disponible/sélectionné/occupé/bloqué), checkout, billet, dashboard et boarding (PDF p. 4–5). Une sélection visuelle n'est pas une garantie de disponibilité.

Différé/exclu initialement : microservices, Kubernetes, Redis, Elasticsearch/OpenSearch, Kafka/Redpanda, GPS live/ETA temps réel, IA, tarification dynamique, fidélité, chat, avis et wallet interne (p. 13–14 et B). Offline boarding, signature asymétrique offline, push/WhatsApp et séparation des PWA sont des possibilités futures, pas des engagements MVP. Aucun cloud actif avant la phase autorisée. Kafka/Outbox reste conditionné à un besoin asynchrone démontré.

## 4. Parcours et résultats observables

1. **Recherche/comparaison** : saisir origine, destination, date, nombre de passagers ; consulter compagnie, heure, prix, durée et places restantes ; accéder au détail. L'affichage ne remplace pas la vérification transactionnelle à la réservation.
2. **Sièges/réservation sans compte** : consulter le plan, choisir un ou plusieurs sièges, saisir contact et informations minimales des passagers ; demander la réservation avec idempotence. Le serveur arbitre les conflits et applique le hold ; un siège perdu entre affichage et validation produit `SEAT_UNAVAILABLE`.
3. **Cash/simulation** : consulter le récapitulatif, choisir le moyen disponible ; une intention cash reste en attente jusqu'à validation autorisée. Le mock de Phase 8 fournit un résultat vérifié côté serveur. Ni le navigateur ni la simple sélection d'un moyen ne confirment la réservation.
4. **Consultation/annulation** : retrouver une réservation par référence avec preuve d'accès supplémentaire à définir ; afficher uniquement les données autorisées ; appliquer la politique de la compagnie avant annulation et éventuel remboursement. La référence seule ne donne aucun droit sur les données personnelles.
5. **Guichet** : un agent authentifié et habilité choisit un trajet de sa compagnie, réserve dans le même inventaire que le web et valide l'encaissement cash si autorisé. La confirmation est traçable ; une vente concurrente web/agence ne peut pas doubler le siège.
6. **QR/embarquement** : délivrer un billet uniquement après confirmation ; l'agent habilité vérifie signature, trajet, billet et état ; réalise check-in puis boarding selon transitions à préciser. Un billet déjà embarqué ne permet jamais un nouvel embarquement.

## 5. Epics et User Stories

Priorités : P0 = invariant ou prérequis indispensable ; P1 = capacité MVP. Les phases indiquent une cible de réalisation, pas une autorisation de démarrage. Les critères dépendant d'une décision sont bloqués par celle-ci ; voir [registre DEC](04_IMPLEMENTATION_PLAN.md#4-registre-des-décisions-ouvertes).

| Epic / ID | Rôle et objectif | Priorité / phase | Critères d'acceptation observables |
|---|---|---|---|
| Accès / US-01 | Voyageur : utiliser le service en français ou arabe sur mobile | P1 / 1, compléments futurs | Changement de langue sur les écrans livrés ; arabe en RTL ; formulaire et navigation utilisables au viewport mobile défini pour recette |
| Accès / US-02 | Employé : s'authentifier pour accéder à ses fonctions | P0 / 1 | Accès anonyme aux fonctions protégées refusé ; jeton expiré refusé ; refresh en cookie HttpOnly avec rotation ; rôle insuffisant refusé |
| Accès / US-03 | Compagnie : conserver la confidentialité de ses données | P0 / 1 puis chaque phase | Un utilisateur de A ne peut lire/modifier une ressource privée de B, même avec son identifiant ; tests négatifs sur chaque ressource livrée |
| Référentiel / US-04 | Administrateur : approuver ou suspendre une compagnie | P1 / 2 | Action réservée aux droits administratifs et auditée ; statut obtenu APPROVED ou SUSPENDED ; effets commerciaux soumis à DEC-10 |
| Référentiel / US-05 | Propriétaire/responsable : gérer agences, véhicules, plans et routes | P1 / 2 | Données enregistrées retrouvables dans sa compagnie ; références à une ressource privée d'un autre tenant rejetées ; champs obligatoires validés selon contrat approuvé |
| Départs / US-06 | Responsable : programmer un trajet et son inventaire | P0 / 3 | Trajet associé à route, véhicule, agences, horaires et prix ; inventaire propre au trajet ; aucune duplication de siège logique ; transitions non autorisées refusées |
| Recherche / US-07 | Voyageur : comparer les départs pertinents | P1 / 4 | Les critères origine/destination/date/passagers sont pris en compte ; résultat affiche compagnie, horaire, prix et disponibilité ; absence de résultat explicitée |
| Recherche / US-08 | Voyageur : consulter le détail et choisir ses sièges | P1 / 4–5 | Plan distingue indisponibilités ; détail présente les informations disponibles ; état serveur revérifié à la réservation, conflit retourné si nécessaire |
| Réservation / US-09 | Voyageur : réserver sans créer de compte | P0 / 5 | Réservation possible avec téléphone de contact et données minimales approuvées ; un rejeu identique respecte le contrat d'idempotence ; aucun billet valide avant confirmation |
| Réservation / US-10 | Voyageur/agent : ne pas subir une double vente | P0 / 5–6 | Sur PostgreSQL réel : 100 tentatives distinctes sur le dernier siège, 1 succès et 99 `SEAT_UNAVAILABLE` ; concurrence web/guichet et entre deux agences : une seule attribution |
| Réservation / US-11 | Voyageur : récupérer la disponibilité après expiration | P0 / 5 | Après durée configurée, hold expiré libéré sans libérer un siège déjà confirmé ; course expiration/confirmation testée ; valeur commerciale fixée par DEC-01 |
| Réservation / US-12 | Voyageur : consulter et annuler sa réservation | P1 / 5 | Référence seule insuffisante pour exposer des données personnelles ; demande non autorisée refusée ; annulation suit la politique approuvée et produit un résultat traçable ; DEC-02/03 préalables |
| Guichet / US-13 | Agent d'agence : vendre et encaisser du cash | P0 / 6 | Inventaire partagé ; agent sans droit cash refusé ; encaissement autorisé confirmé une seule fois ; identité de l'acteur et action auditées |
| Pilotage / US-14 | Responsable : consulter ses départs, ventes, occupation et CA | P1 / 6 (proposé) | Jeu de données connu produit les agrégats attendus selon DEC-16 ; données d'une autre compagnie exclues |
| Billets / US-15 | Voyageur : obtenir un billet QR après confirmation | P0 / 7 | Aucune émission valide pour Booking non CONFIRMED ; QR signé sans données personnelles directes ; billet lié au trajet et passager concernés |
| Embarquement / US-16 | Agent d'embarquement : consulter le manifeste et valider un billet | P0 / 7 | Manifeste limité au périmètre autorisé ; QR falsifié/mauvais trajet refusé ; check-in/boarding tracés ; deuxième utilisation après boarding refusée, y compris scans concurrents |
| Paiement / US-17 | Voyageur : payer par fournisseur simulé puis fournisseur disponible | P0 / 8 | Un succès déclaré par le frontend ne confirme rien ; succès vérifié côté serveur confirme une seule fois ; échec ne délivre pas de billet ; paiement tardif traité selon DEC-05 sans double vente |
| Paiement / US-18 | Plateforme : traiter les notifications fournisseur sûrement | P0 / 8 | Signature/timestamp invalides et replay non autorisé rejetés ; événement valide reçu cinq fois : une seule confirmation, réponse 2xx aux doublons déjà traités |
| Support / US-19 | Support : assister avec des droits limités | P1 / phase à arbitrer | Matrice DEC-04 approuvée avant livraison ; opérations hors périmètre refusées ; consultations/actions sensibles auditées sans données excessives |
| Supervision / US-20 | Administrateur : auditer les actions sensibles | P0 / dès 1, complété 10 | Recherche d'audit accessible seulement aux droits approuvés ; action, acteur, entité et date retrouvables ; aucun secret journalisé |
| Notifications / US-21 | Voyageur : être informé des changements pertinents | P1 / 9 conditionnelle | Après DEC-18, événements retenus déclenchent le message attendu ; absence de diffusion croisée entre compagnies ; doublons gérés selon contrat à définir |

## 6. Règles impératives et décisions métier

Les invariants INV-01 à INV-10 du [modèle de domaine](02_DOMAIN_MODEL.md#4-invariants) sont normatifs : source de vérité PostgreSQL, exclusivité du siège, stock web/guichet commun, isolation, RBAC + tenant, idempotence, webhooks sécurisés, QR sans données personnelles, preuve avant confirmation et tests PostgreSQL réels.

Restent notamment ouverts : durée du hold (10 minutes est un **exemple**, p. 5), annulation/remboursement et tarifs, validation cash, paiement tardif, accès sécurisé sans compte, limites du support, effets d'une suspension ou annulation de trajet, évolution du plan de sièges, réservation multi-passagers et capacité de groupe. Ne déduire aucun taux, délai ou frais commercial.

La découverte terrain du PDF p. 13 reste **non réalisée/non validée** : interviewer 3–5 compagnies, 10–20 voyageurs et des agents (nombre d'agents non précisé), documenter ventes, annulations, tarifs, règles de sièges et paiements. Le registre DEC fixe les échéances avant implémentation concernée. Les obligations de licences, intermédiation, paiement et responsabilité doivent être validées localement avant commercialisation (p. 12).
