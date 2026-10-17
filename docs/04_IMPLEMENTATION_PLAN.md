# 04 — Plan d'implémentation et décisions

Statut : Phase 0 documentaire préparée, **validation utilisateur en attente**. Aucune application disponible. Sources : [PDF](reference/Rihla_MR_Conception_Fonctionnelle_Technique.pdf), 14 pages, et roadmap actualisée fournie le 17 octobre 2026 (B). Le passage à la Phase 1 exige une validation explicite.

## 1. Arbitrage des sources et architecture cible

| Sujet | PDF | Plan applicable (B) / conséquence |
|---|---|---|
| Phase 0 | Découverte terrain (p. 13) | Documentation/workspace ; entretiens conservés comme activité non réalisée, non validée |
| Fondations/référentiels | Company/Agency/City/Vehicle/Route en ancienne Phase 1 | Fondations en 1 ; ressources opérateur en 2 ; référentiel géographique nécessaire avant routes/recherche |
| Inventaire/booking/recherche | Anciennes phases 2–3 mêlent hold, recherche, MockPayment et Ticket | Inventaire 3, recherche 4, réservation/hold 5 ; billets 7 après confirmation ; PaymentProvider/mock 8 |
| Opérateur | Ancienne Phase 4 | Référentiels en 2, programmation en 3, guichet en 6, manifeste/boarding en 7 ; dashboard proposé en 6 |
| Paiement réel | MoosylPaymentProvider annoncé « Phase 2 » p. 9, paiements réels en Phase 6 p. 13 | Incohérence interne du PDF ; Phase 8 actualisée, fournisseur réel seulement si contrat/API disponibles ; aucun engagement fournisseur |
| Kafka | « Phase 2 » p. 10, ancienne Phase 7 p. 13 | Incohérence interne ; Phase 9 conditionnelle avec Outbox, jamais prérequis du MVP transactionnel |
| Notifications | Interface simulable au MVP (p. 10) | Besoins conservés, implémentation planifiée en 9 conditionnelle ; arbitrer le minimum MVP sans bus (DEC-18) |
| Production/avancé | Anciennes phases 8–9 | Hardening/CI/CD/cloud en 10 ; fonctionnalités avancées hors périmètre initial |

Architecture prévue : Java 21, Spring Boot, monolithe modulaire, Spring Security, Spring Data JPA/Hibernate, PostgreSQL, Flyway, Spring Validation, Actuator, Testcontainers, Docker. Conserver l'orientation Spring Modulith sans dépendance à ce stade. Frontend Angular standalone (PDF : 20+ ; version compatible à choisir en Phase 1), Angular Material, Tailwind CSS, mobile-first, FR/AR, RTL et PWA ; espaces public, opérateur et administration dans un même frontend. Modules et frontières : [document 02](02_DOMAIN_MODEL.md#2-modules-et-frontières).

Cible future : GitHub Actions, OIDC/WIF, Artifact Registry, Cloud Run, Firebase Hosting, PostgreSQL et Secret Manager. Le PDF p. 13 mentionne Porkbun/DNS et www.rihla.mr/api.rihla.mr : intentions à vérifier avant activation, pas des ressources actuellement disponibles. Pas de Kafka/Redpanda, Redis, Elasticsearch/OpenSearch, Kubernetes, microservices, GPS live ou infrastructure cloud active au démarrage.

## 2. Plan Phase 0 à Phase 10

### Phase 0 — Documentation et préparation du workspace

- **Objectif** : transformer les sources en documents exploitables et identifier les décisions manquantes sans implémenter.
- **Périmètre** : inspection préalable, lecture intégrale du PDF, quatre documents, README, instructions persistantes, exclusions locales et conservation des dossiers applicatifs vides.
- **Livrables** : `01_PRODUCT_REQUIREMENTS.md` (produit/stories), `02_DOMAIN_MODEL.md` (modèle/invariants), `03_API_CONTRACT.md` (routes/contrats ouverts), présent plan ; README, AGENTS.md, .gitignore ; PDF conservé.
- **Prérequis** : PDF accessible intégralement ; sources A/B identifiées ; état du workspace et instructions existantes inspectés avant écriture.
- **Critères d'acceptation techniques** : tous les livrables présents/non vides ; sept rôles, entités, modules et vingt endpoints explicites couverts ; roadmap 0–10 ; divergences et décisions tracées ; aucun fichier applicatif, secret, dépendance, migration, service ou pipeline ajouté ; aucun fichier préexistant supprimé/écrasé.
- **Critères d'acceptation humains** : validation des exigences et du plan par le demandeur ; arbitrages nécessaires au prochain périmètre assignés ; autorisation explicite de Phase 1. Ces critères ne sont pas acquis par la seule rédaction.
- **Vérifications** : arborescence, comparaison des routes/rôles/entités avec le PDF, relecture croisée, liens locaux, marqueurs de secrets, intégrité du PDF, état Git si disponible. Aucun test backend/frontend, aucune installation nécessaire.
- **Risques** : attributs et règles non définis pris pour acquis ; faux sentiment de validation métier ; ancienne roadmap réutilisée ; dépendances cash/billets/paiement ignorées.
- **Découverte conservée** : 3–5 compagnies, 10–20 voyageurs et agents à interviewer (nombre d'agents non spécifié) ; collecter ventes, annulations, tarifs, sièges et paiements. Activité non réalisée ; résoudre les questions avant les phases qui en dépendent.

### Phase 1 — Fondations backend/frontend

- **Objectif/périmètre** : socle Java/Spring Boot, PostgreSQL, Flyway, sécurité de base et frontières modulaires ; Angular standalone, Material/Tailwind, structure FR/AR/RTL et préparation PWA.
- **Livrables** : projets reproductibles, configuration locale sans secrets, première structure modulaire, base d'authentification/autorisation, navigation des trois espaces. Aucun bus ni cloud actif.
- **Prérequis** : validation explicite de Phase 0 ; décisions de sécurité/tenant nécessaires et versions compatibles choisies, stratégie de dépôts décidée sans dépôt imbriqué.
- **Acceptation** : démarrage local documenté, migration de socle reproductible, authentification de base et refus d'accès non autorisé prouvés ; écrans de socle FR/AR/RTL utilisables.
- **Tests attendus** : build, sécurité, intégration PostgreSQL/Testcontainers/Flyway, limites de modules, tests frontend de navigation/langue. Pas de tests métier prématurés.
- **Risques** : socle trop large, choix de versions incompatibles, autorisation tenant tardive ; durcissement ne doit pas attendre Phase 10 pour les contrôles fondamentaux.

### Phase 2 — Company / Agency / Vehicle / SeatLayout / Route

- **Objectif/périmètre** : référentiel opérateur, ressources et plans ; City/Station selon besoins clarifiés ; administration de compagnie.
- **Livrables** : capacités et interfaces de gestion, contrats finalisés, persistance tenant ; politiques de droits approuvées.
- **Prérequis** : Phase 1 ; DEC-04/09/10/12/17 selon périmètre.
- **Acceptation** : gestion des ressources de sa compagnie ; références étrangères incohérentes refusées ; approbation/suspension auditées avec effets décidés.
- **Tests attendus** : intégrité PostgreSQL, validation, RBAC/tenant négatif, formulaires et parcours opérateur.
- **Risques** : partage non maîtrisé des plans, confusion Station/Agency, effets commerciaux de suspension non définis.

### Phase 3 — Trip / TripSeat / Inventory

- **Objectif/périmètre** : programmer les départs et construire leur inventaire propre ; disponibilité et blocage administratif selon décisions.
- **Livrables** : trajets, inventaire et transitions approuvées ; contrat de consultation de disponibilité.
- **Prérequis** : Phase 2 ; cohérence horaires/montants, plan et transitions clarifiée.
- **Acceptation** : inventaire sans siège logique dupliqué ; relations appartenant au bon tenant ; changements de trajet/plan contrôlés. Aucune vente sans Phase 5.
- **Tests attendus** : intégrité et transitions sur PostgreSQL, génération inventaire, isolation ; tests d'interface de programmation.
- **Risques** : modification du véhicule/plan après ventes, gestion du retard/annulation, fuseaux.

### Phase 4 — Recherche publique

- **Objectif/périmètre** : recherche, comparaison, détails et disponibilité en FR/AR/RTL.
- **Livrables** : quatre GET publics de villes/trajets/sièges, écrans mobile-first correspondants.
- **Prérequis** : Phase 3 ; règles de publication et critères de recherche définis.
- **Acceptation** : critères origine/destination/date/passagers appliqués, données privées absentes, état vide explicite, disponibilité issue du serveur sans garantie de réservation.
- **Tests attendus** : filtres et publication, réponses/validation, interfaces FR/AR/RTL et parcours de recherche ; mesure de requêtes sur jeu représentatif.
- **Risques** : disponibilité périmée, recherche multi-passagers ambiguë, index mal adaptés ; aucun moteur externe par défaut.

### Phase 5 — Booking / Seat Hold / Expiration / Concurrence

- **Objectif/périmètre** : réservation sans compte, hold/expiration, consultation sécurisée et annulation selon politique ; protection transactionnelle PostgreSQL.
- **Livrables** : contrats de réservation/idempotence finalisés, stratégie de concurrence justifiée et testée, politique d'accès sans compte ; transitions documentées.
- **Prérequis** : Phase 4 ; DEC-01/02/03/05/06/07/14 résolues pour ce périmètre.
- **Acceptation** : 100 tentatives distinctes du dernier siège donnent 1 succès et 99 `SEAT_UNAVAILABLE` ; expiration cohérente ; rejeux sans réservation supplémentaire ; aucune confirmation non prouvée, aucun billet valide.
- **Tests attendus** : transactions et courses sur PostgreSQL réel, expiration/reprise après interruption, accès non autorisé, idempotence concurrente et réservation multi-sièges.
- **Risques** : deadlocks, hold jamais libéré, perte de siège à la confirmation, exposition via référence. Aucun choix de locking préimposé en Phase 0.

### Phase 6 — Vente au guichet

- **Objectif/périmètre** : ventes opérateur sur inventaire commun, confirmation cash autorisée ; dashboard simple proposé ici, métriques à valider.
- **Livrables** : parcours guichet, trace de validation cash et preuve d'autorisation, contrat de répétition ; tableaux de bord selon périmètre retenu.
- **Prérequis** : Phase 5 ; politique cash et encaissement, autorisations d'agence et agrégats définis. Clarifier le support minimal de preuve cash avant PaymentProvider complet (section 3).
- **Acceptation** : concurrence web/guichet/deux agences sans double vente ; réservation confirmée seulement par acteur habilité et preuve cash traçable ; répétition sans double encaissement.
- **Tests attendus** : concurrence PostgreSQL, audit, habilitations négatives, idempotence cash, calcul d'agrégats sur données connues.
- **Risques** : annoncer une confirmation à la simple intention cash, périmètre agence trop large, anticiper toute la Phase 8.

### Phase 7 — Ticket QR / Check-in / Boarding

- **Objectif/périmètre** : émission après confirmation, accès au billet, manifeste, validation QR, check-in et boarding.
- **Livrables** : billet signé minimal, parcours de scan et contrôle des doublons ; gestion de clés spécifiée sans secrets en dépôt.
- **Prérequis** : Phase 6 fournit un chemin cash confirmé ; DEC-06/13 et contrats de scan finalisés.
- **Acceptation** : aucun billet valide pour réservation non confirmée ; QR sans données personnelles ; mauvais trajet/falsification refusés ; deuxième utilisation après boarding refusée.
- **Tests attendus** : émission conditionnelle, signature/états, scans concurrents sur PostgreSQL, permissions, E2E cash → confirmation → billet → embarquement.
- **Risques** : divergence des états passager/réservation/billet, fuite de clé ; offline boarding reste différé.

### Phase 8 — Paiements

- **Objectif/périmètre** : PaymentProvider, MockPaymentProvider, fournisseur réel disponible, webhooks, idempotence et remboursements selon politique.
- **Livrables** : contrats fournisseurs et états, rapprochement avec preuve cash existante, déduplication durable, traitement des paiements tardifs/échecs.
- **Prérequis** : phases 5–7 ; DEC-02/05/08/14 ; contrats/API fournisseur et exigences réglementaires avant toute exploitation réelle.
- **Acceptation** : seul le serveur vérifie le succès ; cinq webhooks identiques ne confirment qu'une fois ; signature/timestamp/identité/anti-replay contrôlés ; succès tardif sans double vente ; mock identifié comme simulé.
- **Tests attendus** : intégration PostgreSQL, webhooks répétés/concurrents/désordonnés, signatures invalides, paiement après expiration, remboursements ; E2E mock.
- **Risques** : API indisponible, replay, rapprochement incomplet, compensation commerciale indéfinie. Moosyl est une possibilité du PDF, pas une dépendance garantie.

### Phase 9 — Transactional Outbox / Kafka / Notifications

- **Objectif/périmètre** : traiter des besoins asynchrones **démontrés** ; sinon phase non activée avec décision documentée.
- **Livrables** : justification du besoin, Outbox transactionnelle, transport Kafka/Redpanda seulement si nécessaire, consommateurs idempotents ; NotificationProvider/canaux approuvés. Audit sensible déjà présent avant cette phase.
- **Prérequis** : parcours transactionnels fiables ; DEC-18 ; exigences de reprise/délais et coûts établies.
- **Acceptation** : changement métier et événement durable atomiques ; publication reprenable après panne ; répétitions sans effet métier supplémentaire ; aucun événement privé livré au mauvais tenant.
- **Tests attendus** : panne après commit/avant publication, rejeu, déduplication, ordre/retard, livraison des notifications retenues et isolation.
- **Risques** : complexité sans besoin, notifications doubles, données personnelles dans événements ; aucun topic du PDF n'est imposé sans contrat.

### Phase 10 — Production hardening / CI/CD / Cloud

- **Objectif/périmètre** : sécurité, observabilité, charge, sauvegarde/restauration et déploiement contrôlé ; compléter les mesures déjà nécessaires aux phases antérieures.
- **Livrables** : CI GitHub Actions, images Docker traçables par SHA, Artifact Registry, Cloud Run, Firebase Hosting, Secret Manager ; logs structurés/requestId, health/readiness, métriques ; procédures backup/restore/retour arrière.
- **Prérequis** : périmètre MVP retenu validé ; Phase 9 réalisée ou explicitement non activée ; conformité locale validée avant commercialisation ; autorisation explicite de déploiement.
- **Acceptation** : CI verte, aucun secret versionné, audit dépendances, charge selon objectifs approuvés, restauration prouvée ; production manuelle/contrôlée (`workflow_dispatch` envisagé), pas de credentials production dans les PR.
- **Tests attendus** : JUnit/Mockito, PostgreSQL/Testcontainers/Flyway, frontend, E2E Playwright, charge k6, sécurité et restauration selon périmètre ; budgets de performance à définir.
- **Risques** : coûts, données personnelles dans logs, pool DB, absence de rollback, autorisations juridiques manquantes. Lag Kafka seulement si Kafka est activé.

## 3. Dépendance Booking → cash → billet → PaymentProvider

1. **Phase 5** : créer et maintenir une réservation/hold sans prétendre qu'un paiement a réussi. Définir la garde de confirmation, mais aucun fournisseur mock complet ni billet valide.
2. **Avant Phase 6** : arrêter le contrat minimal de preuve cash (acteur habilité, opération, montant et traçabilité nécessaires à définir). Le PDF ne tranche pas son stockage : enregistrement Payment minimal ou preuve cash intégrable au module payment sont des options, pas des choix actés. L'arbitrage DEC-05 doit permettre une preuve durable et une confirmation idempotente sans déplacer toute la Phase 8.
3. **Phase 6** : le cash autorisé constitue le premier chemin de confirmation exploitable. Sans preuve et validation autorisée, Booking reste en attente et le chemin billet est bloqué.
4. **Phase 7** : émettre le billet uniquement pour Booking confirmé, initialement par ce chemin cash. Les tests peuvent construire un état confirmé avec preuve contrôlée ; cela n'autorise pas une confirmation arbitraire en production.
5. **Phase 8** : introduire PaymentProvider/mock puis fournisseur disponible et réutiliser la garde métier ; intégrer le cash sans confirmation en double. Le scénario voyageur complet avec paiement simulé n'est donc livré qu'à cette phase.

Ainsi la Phase 7 n'attend pas une intégration bancaire, mais dépend réellement d'une confirmation autorisée en Phase 6. Si le support cash ne peut être défini dans cette limite, signaler le blocage et demander un arbitrage de roadmap avant de coder ; ne jamais contourner INV-09.

## 4. Registre des décisions ouvertes

Toutes les lignes sont **ouvertes** : aucune politique commerciale ni validation terrain n'est acquise. Une « phase limite » signifie avant l'implémentation concernée ; les décisions peuvent être prises plus tôt.

| ID | Question | Impact | Phase limite de décision | Statut |
|---|---|---|---|---|
| DEC-01 | Durée configurable du hold, début du délai, renouvellement et expiration en attente de paiement ? | Disponibilité et concurrence ; 10 min n'est qu'un exemple | 5 | Ouvert |
| DEC-02 | Politique compagnie d'annulation, délais, frais, remboursements partiels, réservation/trajet annulé ? | Droits voyageur, restitution siège, comptabilité | 5 pour annulation ; 8 pour intégration refund | Ouvert |
| DEC-03 | Quelle preuve supplémentaire pour consulter/annuler sans compte, et quelle récupération ? | Protection des données et anti-énumération | 5 | Ouvert |
| DEC-04 | Matrice RBAC/tenant, support, affectations agence/trajet, auth/rotation/MFA et comptes multi-compagnies ? | Accès privés et opérations cash/boarding | Socle 1 ; compléter avant chaque phase concernée | Ouvert |
| DEC-05 | Preuve et autorité cash, stockage minimal, confirmation/expiration concurrentes, paiement tardif après siège réattribué ? | Chaîne 5→6→7→8 ; pas de billet sans confirmation | Garde/course en 5 ; cash avant 6 ; détail fournisseur avant 8 | Ouvert |
| DEC-06 | Transitions complètes, réservation active, plusieurs sièges/passagers, atomicité, état Booking agrégé ? | Intégrité et cohérence des cycles | Trip en 3 ; Booking/TripSeat en 5 ; boarding en 7 | Ouvert |
| DEC-07 | Quelle stratégie de verrouillage/contraintes, isolation, ordre d'acquisition et retries ? | Preuve de non-double vente PostgreSQL | 5 | Ouvert |
| DEC-08 | Rattachement et tentatives Payment, identifiants fournisseur, unicité événement, refunds/rapprochement ? | Idempotence et comptabilité | Sous-ensemble cash en 6 ; complet en 8 | Ouvert |
| DEC-09 | Portée des company_id, Station/City/Agency, plans partagés et cohérence des relations ? | Modèle tenant et géographie | Socle tenant en 1 ; référentiels en 2 | Ouvert |
| DEC-10 | Effets d'une approbation/suspension compagnie, publication, retard et annulation de trajet ? | Ventes, réservations, communications | Compagnie 2 ; trajet 3 ; propagation avant 5 | Ouvert |
| DEC-11 | Fuseaux, conventions horaires, devise, précision/arrondis et affichage prix ? | Recherche, montants et calculs | 3 ; affiner avant 8 | Ouvert |
| DEC-12 | Identité/versionnement du plan, changement de véhicule/sièges après réservation ? | Stabilité de l'inventaire | Plan 2 ; effets trajet 3 | Ouvert |
| DEC-13 | Signature QR, clés/rotation, récupération billet, anti-réutilisation et retries de scan ? | Sécurité ticket ; offline différé | 7 | Ouvert |
| DEC-14 | DTO, erreurs/HTTP, pagination, idempotence (portée/durée/conflits), endpoints manquants ? | Contrats testables | Avant chaque route ; réservations 5, paiement 8 | Ouvert |
| DEC-15 | Licences transport, intermédiation, paiements, responsabilité et règles locales de données/rétention ? | Autorisation de commercialiser et conservation | Avant tout pilote commercial/paiement réel ; au plus tard 10 | Ouvert ; validation locale requise |
| DEC-16 | Définitions CA/ventes/occupation, périodes, cash/remboursements, droits sur rapports ? | Dashboard fiable | 6 si cible retenue | Ouvert |
| DEC-17 | Fare entité autonome ? Tarifs, informations passager minimales, passager réutilisable, vente téléphone ? | Modèle et périmètre MVP | Tarifs/référentiels 2–3 ; passagers 5 | Ouvert |
| DEC-18 | Notifications minimum MVP, canaux, consentements/délais, besoin asynchrone réel et activation de 9 ? | Écart notifications MVP PDF / ordre actualisé ; coût du bus | Avant recette MVP ; avant activation 9 | Ouvert |
| DEC-19 | Résultats des entretiens terrain et validation des exigences par les parties prenantes ? | Risque de produit fondé sur hypothèses | Organiser dès cadrage ; résultats avant décisions métier concernées | Non réalisé / non validé |
| DEC-20 | Versions compatibles, conventions de build et dépôts Git futurs (racine ou séparés ultérieurement) ? | Fondations reproductibles ; pas de dépôt imbriqué | 1 | Ouvert |
| DEC-21 | Rétention audit, objectifs charge/disponibilité, sauvegardes, RPO/RTO et coûts cloud ? | Exploitation et recette production | Audit minimal 1 ; objectifs avant 10 | Ouvert |

## 5. Vérification et progression

Chaque phase exige des contrats assez précis pour ses livrables, des tests proportionnés et un rapport factuel. Les scénarios PDF p. 12 sont conservés : dernier siège à 100 concurrents, cinq webhooks, deux agences, paiement après expiration, QR utilisé deux fois, annulation de trajet et propagation selon politique. Les règles sensibles sont vérifiées sur PostgreSQL réel, y compris isolation négative entre compagnies.

La Phase 0 ne lance pas ces tests applicatifs : elle en établit les obligations futures. Une vérification documentaire réussie ne vaut ni validation métier ni permission d'implémentation. Après remise du rapport Phase 0, attendre la validation explicite du demandeur avant Phase 1.
