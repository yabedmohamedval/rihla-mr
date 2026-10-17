# 02 — Modèle de domaine et invariants

Statut : modèle documentaire, aucun schéma SQL ou modèle Java définitif. Sources : [PDF](reference/Rihla_MR_Conception_Fonctionnelle_Technique.pdf) p. 5–8 et 11–14 ; source B pour les invariants et l'ordre de développement. Les compléments proposés sont explicitement signalés.

## 1. Relations et entités

Vue conceptuelle du PDF p. 6 : Company regroupe Agency, AppUser, Vehicle et Route ; Vehicle utilise SeatLayout composé de Seat ; Route porte des Trip ; Trip possède ses TripSeat ; Booking rattache les passagers à un trajet et à ses sièges ; Ticket sert leur contrôle. Payment et Refund tracent les règlements ; AuditEvent et WebhookEvent tracent les actions et événements.

Cette vue ne fixe pas les cardinalités physiques. Le dessin du PDF place Payment près de TripSeat sans définir sa clé de rattachement : le rattachement précis au Booking, le nombre de tentatives et les paiements partiels restent à définir (DEC-08). City/Station forment le référentiel géographique ; les relations exactes avec Route, Agency et Trip ne sont pas entièrement décrites.

Dans le tableau, les attributs sont ceux réellement cités p. 6–7 (complément QR p. 10). « Par relation » décrit un périmètre métier attendu, pas une colonne inventée. Le PDF exige `company_id` explicite sur les éléments sensibles, sans lister toutes les tables concernées ; DEC-09 doit fixer ce périmètre.

| Entité | Responsabilité et attributs confirmés | Relations principales | Tenant et contraintes |
|---|---|---|---|
| Company | Identité, licence, statut PENDING/APPROVED/SUSPENDED | Agences, comptes, véhicules, routes | Racine du tenant ; effets des statuts à définir |
| Agency | Compagnie, ville, adresse, géolocalisation, téléphone | Company, City ; agences d'un Trip | Appartient à Company ; périmètre des agents ouvert |
| AppUser | Compte, `company_id` optionnel, rôle, statut | Company optionnelle | Comptes plateforme possibles ; multi-rôles/multi-compagnies non définis ; voyageur sans compte autorisé |
| City | Référentiel géographique ; autres champs : À définir | Agency ; recherche et origine/destination à préciser | Référentiel potentiellement partagé, portée à valider |
| Station | Référentiel géographique ; champs : À définir | Lien à City/Agency/Route : À définir | Portée tenant : À définir |
| Route | Origine, destination, durée estimée, distance | Company, Trip ; type des points géographiques ouvert | Compagnie dans le diagramme ; intégrité origine/destination à préciser |
| Vehicle | Immatriculation, type, plan de sièges, statut | Company, SeatLayout, Trip | Compagnie ; unicité immatriculation/portée non spécifiée |
| SeatLayout | Configuration physique des sièges | Vehicle, Seat | Portée tenant/réutilisation/versionnement : À définir |
| Seat | Élément de configuration physique ; libellé/position non spécifiés | SeatLayout ; correspondance TripSeat à préciser | Portée par plan à préciser ; identité du siège logique à définir |
| Trip | Route, véhicule, agences, horaires, prix, statut | Route, Vehicle, Agency, TripSeat, Booking | Compagnie par relations ; toutes les ressources privées doivent être cohérentes avec le tenant |
| TripSeat | Inventaire d'un trajet : état, hold, expiration | Trip ; Seat et affectation Booking à préciser | Compagnie par Trip ; attribution active exclusive |
| Booking | Référence, trajet, contact, statuts, montants | Trip, BookingPassenger ; Payment à préciser | Compagnie du trajet ; référence unique recommandée ; confirmation conditionnée |
| BookingPassenger | Passager, siège, tarif associé | Booking ; siège ; Ticket dans le diagramme | Compagnie du Booking ; données personnelles minimales |
| Payment | Fournisseur, méthode, montant, statuts, identifiants externes (groupe payment/refund) | Booking/transaction à préciser ; Refund | Compagnie de l'opération ; succès vérifié côté serveur ; identité fournisseur à préciser |
| Refund | Fournisseur, méthode, montant, statuts, identifiants externes (groupe payment/refund) | Payment | Compagnie de l'opération ; politique d'éligibilité et remboursements partiels ouverts |
| Ticket | Numéro, QR, statuts check-in/boarding | BookingPassenger/Booking, Trip selon QR | Compagnie du trajet ; référence unique recommandée ; pas de seconde utilisation après embarquement |
| AuditEvent | Action, acteur, entité, métadonnées, horodatage | Acteur et entité concernés | Portée compagnie ou plateforme à définir ; pas de secrets ni données excessives |
| WebhookEvent | `provider_event_id` unique, type, statut de traitement | Fournisseur ; Payment/rattachement exact à définir | Résolution fiable du tenant côté serveur ; unicité et traitement idempotent |

`Fare` apparaît seulement dans l'ancienne roadmap p. 13 ; son existence comme entité autonome n'est pas définie. `Passenger` y est aussi un raccourci, tandis que p. 6 définit BookingPassenger : aucun nouveau profil passager persistant n'est imposé (DEC-17).

## 2. Modules et frontières

Le PDF p. 7 impose un monolithe modulaire sous `mr.rihla`, chaque module structuré en `api / application / domain / infrastructure`. Spring Modulith reste l'orientation pour les frontières et événements internes, sans ajout de dépendance en Phase 0. Les noms ci-dessous sont confirmés ; la répartition de responsabilité est une lecture fonctionnelle initiale à affiner, sans couplage persistant intermodule implicite.

| Module | Responsabilité initiale |
|---|---|
| identity | Comptes, authentification, rôles ; contrôle tenant appliqué aussi dans les modules propriétaires |
| company | Cycle de compagnie et administration de son périmètre |
| agency | Agences et organisation du guichet |
| geography | City, Station et référentiel géographique |
| route | Liaisons origine/destination |
| vehicle | Véhicules et plans physiques de sièges |
| trip | Programmation, horaires, prix et cycle des trajets |
| inventory | TripSeat, disponibilité, hold, expiration, attribution exclusive |
| booking | Réservation, passagers, annulation, orchestration des invariants |
| payment | Paiements, remboursements, PaymentProvider et événements fournisseur |
| ticketing | Émission du billet confirmé, référence et QR signé |
| boarding | Manifeste, check-in, embarquement et protection contre réutilisation |
| notification | NotificationProvider et messages ; activation selon besoin/Phase 9 |
| reporting | Dashboard et agrégats autorisés par tenant |
| audit | Traçabilité des actions sensibles |
| shared | Primitives transversales limitées ; ne pas y déplacer les règles métier par commodité |

Les appels entre booking, inventory, payment et ticketing doivent respecter leurs responsabilités et permettre l'atomicité nécessaire. APIs internes, dépendances autorisées et propriété exacte de WebhookEvent restent à spécifier avant réalisation. Ni microservices ni bus externe ne sont requis pour ces échanges.

## 3. Cycles de vie (PDF p. 5–6)

| Agrégat | Chaîne principale explicitement indiquée | Branches indiquées | Ambiguïtés à lever |
|---|---|---|---|
| Trip | DRAFT → SCHEDULED → BOARDING → DEPARTED → ARRIVED → COMPLETED | DELAYED, CANCELLED autour de SCHEDULED | Origines exactes des branches, reprise après retard, effets d'annulation, permissions et horaires limites |
| TripSeat | AVAILABLE → HELD → BOOKED → CHECKED_IN → BOARDED | Expiration → AVAILABLE ; BLOCKED = indisponibilité administrative | Expiration s'applique au hold ; blocage/déblocage, annulation et libération à préciser ; aucun retour automatique d'un siège vendu |
| Booking | PENDING → HELD → AWAITING_PAYMENT → CONFIRMED → CHECKED_IN → BOARDED → COMPLETED | PAYMENT_FAILED depuis attente de paiement ; HELD → EXPIRED ; CONFIRMED → CANCELLED | Expiration en AWAITING_PAYMENT, nouvelles tentatives, annulation avant confirmation et agrégation multi-passagers |
| Payment | CREATED → PENDING → SUCCEEDED | PENDING → FAILED ; SUCCEEDED → REFUND_PENDING → REFUNDED | Échecs/remboursements partiels, tentatives, rapprochement et événements désordonnés |

Les chaînes reproduisent les états du PDF ; elles ne constituent pas une table exhaustive de transitions autorisées. Les gardes, acteurs, compensations et synchronisations Booking/TripSeat/Ticket doivent être définis (DEC-06). Un état en échec ne justifie jamais à lui seul une libération qui écraserait une attribution valide.

## 4. Invariants

| ID | Exigence obligatoire | Preuve future attendue |
|---|---|---|
| INV-01 | PostgreSQL est la source de vérité transactionnelle ; l'UI ne décide pas de la disponibilité | Vérification serveur lors de l'attribution, même après un affichage périmé |
| INV-02 | Un siège d'un trajet n'appartient jamais simultanément à plusieurs réservations actives | 100 tentatives distinctes du dernier siège : 1 succès, 99 `SEAT_UNAVAILABLE` |
| INV-03 | Ventes web et guichet partagent exactement le même inventaire | Concurrence web/guichet et deux agences : une seule vente |
| INV-04 | Les données privées de chaque compagnie sont isolées | Tests négatifs lecture/écriture entre tenants |
| INV-05 | Autorisation = RBAC ET contrôle de l'appartenance au tenant | Un rôle valide ne permet pas d'utiliser la ressource d'une autre compagnie |
| INV-06 | Réservations et paiements critiques sont idempotents selon leurs contrats | Rejeu sans deuxième réservation, encaissement ou confirmation ; portée/durée des clés à décider |
| INV-07 | Webhooks : signature, timestamp, identité d'événement et anti-replay vérifiés | Événements invalides rejetés ; cinq livraisons valides ne produisent qu'un effet |
| INV-08 | QR minimal signé, sans données personnelles directes | Contenu limité à la référence technique (ticketId, tripId, nonce/signature) ; falsification rejetée |
| INV-09 | Booking n'est CONFIRMED qu'après preuve de paiement ou validation cash autorisée | Pas de confirmation par le frontend ; pas de billet valide avant confirmation |
| INV-10 | Les règles métier sensibles sont testées sur PostgreSQL réel | Tests d'intégration/concurrence Testcontainers ; mocks ou base en mémoire ne suffisent pas |

Autres règles confirmées : hold automatiquement expirant après une durée configurable, annulation selon politique compagnie avec remboursement éventuel, billet embarqué non réutilisable (p. 5–6). La définition précise de « réservation active » doit être alignée sur les états et les holds, sans réduire INV-02 (DEC-06).

## 5. Isolation, transactions et concurrence

Le tenant d'une opération privée doit être dérivé d'un contexte authentifié autorisé, puis comparé aux ressources demandées ; un `company_id` fourni par le client ne constitue pas une preuve. Pour les ventes publiques, le trajet détermine la compagnie, mais ne donne aucun accès aux autres données privées. Les actions plateforme/support exigent un périmètre explicite et un audit ; l'absence de company_id sur AppUser n'est pas un passe-droit.

Exigences transactionnelles : acquisition des sièges/hold et création de réservation cohérentes ; arbitrage atomique entre expiration, vente et confirmation ; confirmation de paiement et de réservation sans effet partiel incohérent ; autorisation et état recontrôlés à l'écriture. Les frontières exactes de transaction et les reprises après crash restent à concevoir. Ne pas maintenir une transaction DB pendant un appel réseau fournisseur sans analyse.

Le PDF demande contraintes et verrouillage transactionnel, mais ne choisit **aucune stratégie précise**. Options à comparer en Phase 5 : verrouillage pessimiste, versionnement optimiste, mises à jour conditionnelles et contraintes adaptées, éventuellement combinées. Évaluer contention, deadlocks, ordre d'acquisition de plusieurs sièges, retries et expiration concurrente. Aucun choix n'est arrêté ici (DEC-07).

Un paiement réussi après expiration ne peut reprendre un siège attribué à autrui. Remboursement, réaffectation ou autre traitement commercial : **À définir** (DEC-05). Une réservation multi-sièges doit faire l'objet d'une décision explicite sur l'atomicité et la gestion partielle (DEC-06).

## 6. Intégrité et unicité

Recommandations du PDF p. 7 à traduire lors de la conception physique : unicité des références Booking et Ticket ; unicité `provider_event_id` de WebhookEvent ; contraintes sur TripSeat empêchant la double vente ; clés étrangères strictes et suppressions contrôlées ; index sur origine/destination/départ/statut/company_id. La composition des index et des clés n'est pas définie.

Points techniques à spécifier : identité unique du siège dans un trajet ; cohérence du tenant des relations ; portée globale ou par fournisseur de l'identifiant d'événement (ne pas affaiblir la déduplication) ; format/devise/précision des montants ; fuseau et stockage des horaires ; modification d'un plan après ventes. L'unicité des immatriculations, le nombre de billets par passager ou la propriété globale de Station ne sont pas décidés par le PDF.

Pour les événements externes futurs, Transactional Outbox doit associer changement métier et événement dans la même transaction PostgreSQL ; l'activation du bus n'arrive qu'en Phase 9 justifiée. L'audit nécessaire à la sécurité existe avant tout éventuel audit asynchrone.

## 7. Décisions restantes

Consulter le [registre unique DEC](04_IMPLEMENTATION_PLAN.md#4-registre-des-décisions-ouvertes) : hold/annulation (01–02), cash et paiement tardif (05), transitions (06), locking (07), paiements (08), multi-tenancy/géographie (09), statuts compagnie/trajet (10), temps/montants (11), plans (12), QR (13), contrats (14), conformité (15), indicateurs (16), tarifs/passagers (17). Toute propriété absente du PDF reste une décision de phase, pas un attribut confirmé.
