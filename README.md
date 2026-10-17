# Rihla MR

Plateforme B2B2C de transport interurbain en Mauritanie : voyageurs, compagnies, agences et embarquement autour d'un inventaire de sièges partagé entre web et guichet.

**État réel : Phase 0 documentaire préparée, validation utilisateur en attente.** Aucun backend, frontend, service ou déploiement n'est implémenté ; aucune application n'est actuellement exécutable.

## Architecture prévue

Java 21 / Spring Boot en monolithe modulaire, orientation Spring Modulith, Security, JPA/Hibernate, PostgreSQL, Flyway, Validation, Actuator et Testcontainers. Angular standalone, Material/Tailwind, FR/AR, RTL, mobile-first et PWA ; espaces public, opérateur et administration. Docker puis CI/CD/cloud dans les phases dédiées. PostgreSQL reste la source de vérité ; aucun bus ou cache externe n'est requis au démarrage.

## Workspace

```text
rihla-mr/
├── rihla-mr-api/          # vide, backend futur
├── rihla-mr-front/        # vide, frontend futur
├── docs/
│   ├── 01_PRODUCT_REQUIREMENTS.md
│   ├── 02_DOMAIN_MODEL.md
│   ├── 03_API_CONTRACT.md
│   ├── 04_IMPLEMENTATION_PLAN.md
│   └── reference/
│       └── Rihla_MR_Conception_Fonctionnelle_Technique.pdf
├── README.md
├── AGENTS.md
└── .gitignore
```

Les dossiers techniques préexistants `.git`, `.agents` et `.codex` ont été conservés. À l'inspection initiale, ils étaient vides ; `.git` ne constituait pas un dépôt Git utilisable. Aucun dépôt n'a été initialisé. Les dossiers applicatifs pourront devenir deux dépôts indépendants ultérieurement, sans dépôt imbriqué.

## Documentation de travail

1. [Exigences produit](docs/01_PRODUCT_REQUIREMENTS.md) : rôles, MVP, parcours, stories et critères observables.
2. [Modèle de domaine](docs/02_DOMAIN_MODEL.md) : entités, modules, cycles, invariants et décisions techniques ouvertes.
3. [Contrat API](docs/03_API_CONTRACT.md) : routes explicites, sécurité et payloads restant à définir.
4. [Plan d'implémentation](docs/04_IMPLEMENTATION_PLAN.md) : phases 0–10, dépendances et registre des décisions.

Le [PDF de conception v1.0](docs/reference/Rihla_MR_Conception_Fonctionnelle_Technique.pdf), lu sur ses 14 pages, est la source métier. La roadmap actualisée fournie par le demandeur détermine l'ordre d'implémentation ; ses divergences avec le PDF sont tracées dans le plan. Les entretiens de découverte restent non réalisés/non validés.

Lire les documents applicables avant chaque phase, résoudre les décisions bloquantes, respecter le périmètre, tester les changements et produire un rapport. Ne pas présenter les éléments « À définir » comme approuvés. **Attendre la validation explicite de Phase 0 avant toute implémentation de Phase 1.** Aucun commit, push ou déploiement sans autorisation. Instructions persistantes : [AGENTS.md](AGENTS.md).
