## aefi-acquisition

> <!-- start: Packmind standards -->

<!-- start: Packmind standards -->
# Packmind Standards

Before starting your work, make sure to review the coding standards relevant to your current task.

Always consult the sections that apply to the technology, framework, or type of contribution you are working on.

All rules and guidelines defined in these standards are mandatory and must be followed consistently.

Failure to follow these standards may lead to inconsistencies, errors, or rework. Treat them as the source of truth for how code should be written, structured, and maintained.

# Standard: SolidAI — Aggregate Domain : Racine, Entités et Value Objects

Standardize each SolidAI domain module as a single-aggregate structure with one dataclass aggregate root at the module root (Trio Atomique: intention.md + implementation + _tests/), entities in entities/, immutable identityless value objects in value_objects/, and immutable domain events in events/ (each with its own Trio Atomique) to make aggregate boundaries, invariants, and responsibilities immediately navigable and unambiguous. :
* Implémenter l'aggregate root comme dataclass portant les invariants métier et exposant les méthodes de mutation de l'agrégat — sans logique d'infrastructure ni import hors domain/
* Isoler chaque entité de l'agrégat dans domain/X/entities/ avec son propre Trio Atomique — une entité a une identité propre mais n'est pas la racine de l'agrégat
* Isoler chaque value object dans domain/X/value_objects/ avec son propre Trio Atomique — un value object est immuable, sans identité, défini uniquement par sa valeur
* Isoler les événements domain de l'agrégat dans domain/X/events/Y/ avec leur propre Trio Atomique — un événement domain est immuable, représente un fait passé, et ne dépend que du domain/
* Placer exactement un seul fichier de code à la racine du module domain — l'aggregate root — accompagné de son intention.md et de son dossier _tests/ ; ne jamais y placer d'entités ni de value objects

Full standard is available here for further request: [SolidAI — Aggregate Domain : Racine, Entités et Value Objects](.packmind/standards/solidai-aggregate-domain-racine-entites-et-value-objects.md)

# Standard: SolidAI — Aggregate Domain : Racine, Entités, Value Objects et Events

Standardiser l’anatomie des modules domain SolidAI en structurant chaque agrégat avec un unique aggregate root Python dataclass à la racine (Trio Atomique <module>_intention.md + implémentation + _tests/), des entités/value objects/events isolés dans entities/, value_objects/ et events/ avec leur propre Trio Atomique, et des interfaces dans repositories/ sans imports hors domain/ ni logique d’infrastructure afin de préserver les invariants, éviter les collisions de noms et améliorer la maintenabilité. :
* Créer un sous-dossier repositories/ à la racine du module domain pour y placer les interfaces de repository de l'agrégat — ces interfaces expriment le contrat de persistance du domain sans dépendance infrastructure
* Implémenter l'aggregate root comme dataclass Python portant les invariants métier et les méthodes de mutation — sans import hors domain/ et sans logique d'infrastructure
* Isoler chaque entité de l'agrégat dans domain/X/entities/Y/ avec son propre Trio Atomique (<entity>_intention.md, <entity>.py, _tests/) — une entité a une identité propre mais n'est pas la racine de l'agrégat
* Isoler chaque événement domain de l'agrégat dans domain/X/events/Y/ avec son propre Trio Atomique (<event>_intention.md, <event>.py, _tests/) — un événement est immuable et représente un fait passé dans le domain
* Isoler chaque value object dans domain/X/value_objects/Y/ avec son propre Trio Atomique (<vo>_intention.md, <vo>.py, _tests/) — un value object est immuable, sans identité, défini uniquement par sa valeur
* Placer exactement un seul fichier de code à la racine du module domain — l'aggregate root — avec son <module>_intention.md et son dossier _tests/ ; ne jamais y placer d'entités, value objects ou events

Full standard is available here for further request: [SolidAI — Aggregate Domain : Racine, Entités, Value Objects et Events](.packmind/standards/solidai-aggregate-domain-racine-entites-value-objects-et-events.md)

# Standard: SolidAI — Architecture Fractale Trio Atomique : Intention, Code, Tests Co-localisés

Enforce the SolidAI “Trio Atomique” fractal module layout—<module>_intention.md (Rationale/Responsibility/Design), <module>.py, and co-localized _tests/<module>_test.py at distance 1 (never .test.py)—to standardize DDD architecture, preserve Python import resolution, and make intent, implementation, and validation auditable while minimizing architectural debt. :
* Appliquer la structure du trio atomique (<module>_intention.md + <module>.py + _tests/) de manière uniforme à toutes les couches DDD sans exception
* Créer un fichier intention.md pour chaque module en utilisant le template Rationale / Responsibility / Design, même si le contenu est vide au départ
* Nommer les fichiers de test avec le suffixe _test.py (underscore) et jamais .test.py (point) — le point dans un nom de fichier Python casse la résolution des imports
* Placer les tests dans un sous-dossier _tests/ à l'intérieur du dossier du module — c'est le principe de distance minimale : chaque fichier est à exactement 1 niveau de son test
* Respecter l'ordre TDD lors de la création d'un atome : d'abord <module>_intention.md, puis <module>_test.py, puis l'implémentation <module>.py
* Structurer chaque atome avec <module>_intention.md et le code de production à la racine, et les tests dans _tests/ — les trois éléments restent dans le même arbre de dossiers

Full standard is available here for further request: [SolidAI — Architecture Fractale Trio Atomique : Intention, Code, Tests Co-localisés](.packmind/standards/solidai-architecture-fractale-trio-atomique-intention-code-tests-co-localises.md)

# Standard: SolidAI — Graph of Intentions (Promise↔DDD) Modeling

Appliquer une taxonomie stable et traçable (sources, tags, mappings) pour faire émerger des fonctionnalités atomiques et guider leur projection en DDD. :
* Appliquer la bijection Promise Theory ↔ DDD_Core (Agent↔Aggregate Root, +emit↔Domain Event, +expose/-accept↔Commands, Scope↔Bounded Context, Cooperation↔Use Case, Body↔Invariant, Assessment↔Tests).
* Définir une fonctionnalité atomique (FA) par la cohésion irréductible de l’intention et son ordre d’atomicité N (nombre d’Aggregate Roots impliqués), plutôt que par une frontière technique.
* Exiger une orchestration applicative explicite quand N>1 (Process Manager / Saga / Use Case multi-aggregates) et vérifier les invariants de chaque aggregate + invariants de liaison en fin d’opération.
* Maintenir la séparation en trois espaces `thoughts_interface/`, `promise_model/`, `code_interface/` et retarder la documentation d’infrastructure jusqu’à extraction d’une fonctionnalité atomique validée.
* Nommer et distinguer explicitement Commands (impératif) et Domain Events (passé) et ne pas les mélanger.
* Préserver l’indépendance d’infrastructure au niveau domaine (pas de dépendances infra dans Domain Model; entrées/sorties exprimées en objets du domaine).
* Tisser des liens minimaux mais suffisants: intention→(events)→(aggregate)→(strategies/use cases) et rejeter une FA candidate si la cohérence de bounded context ou l’atomicité échoue.
* Tracer la provenance de chaque note (S_user, S_retroing, S_doc) et conserver l’utilisateur comme terme source dominant (toute FA doit avoir une source S_user).

Full standard is available here for further request: [SolidAI — Graph of Intentions (Promise↔DDD) Modeling](.packmind/standards/solidai-graph-of-intentions-promiseddd-modeling.md)

# Standard: SolidAI — Graphe de Pensée : Ossature DDD-Crew et Règles de Traversal [EXPÉRIMENTAL]

Structurer les nœuds du graphe de pensée en trois couches de stabilité calquées sur le DDD Starter Modelling Process, et guider l'agent dans sa traversée du stable vers le volatile. :
* Mapper chaque nœud à exactement une couche de stabilité : #domain (étapes Understand/Discover), #application (étapes Decompose/Strategize/Connect), #infra (étapes Define/Code) — tout nœud sans tag de couche est invalide.
* Ne modifier un nœud #domain qu'après validation utilisateur explicite, en traçant la source de la modification (S_user, S_retroing ou S_doc) — la discovery est continue mais toute modification de couche stable est auditée.
* Orienter tous les liens du volatile vers le stable (#infra → #application → #domain), jamais dans le sens inverse.
* Respecter l'ordre d'apparition des couches : un nœud #application requiert au moins un nœud #domain parent dans le même bounded context ; un nœud #infra requiert au moins un nœud #application parent.

Full standard is available here for further request: [SolidAI — Graphe de Pensée : Ossature DDD-Crew et Règles de Traversal [EXPÉRIMENTAL]](.packmind/standards/solidai-graphe-de-pensee-ossature-ddd-crew-et-regles-de-traversal-experimental.md)

# Standard: SolidAI — Navigation Agents IA : Placement Déterministe Fake, Real, Interface, Atome

Définit le placement déterministe O(1) pour les agents IA : Fake → infrastructure/fake/, Real → infrastructure/real/, Interface → domain/repositories/, co-localisation dans _tests/, séparation par couche architecturale et non unit/integration. :
* Appliquer la règle de placement unique pour tout nouveau fichier : Interface (IXxx/Port) → domain/repositories/, Fake* → infrastructure/fake/, Real* → infrastructure/real/
* Séparer les tests par couche architecturale (domain/, application/, infrastructure/) et non par nature unit/integration — co-localiser chaque test dans le _tests/ de son module
* Utiliser le fichier intention.md comme pont de synchronisation entre le Meta-Domain (Markdown, Gherkin) et le code — le maintenir à jour quand l'implémentation diverge de la spécification

Full standard is available here for further request: [SolidAI — Navigation Agents IA : Placement Déterministe Fake, Real, Interface, Atome](.packmind/standards/solidai-navigation-agents-ia-placement-deterministe-fake-real-interface-atome.md)

# Standard: SolidAI — Nomenclature : snake_case, Préfixes I/fake_, Dossier _tests/, Suffixe _test.py

Standardize SolidAI file, folder, and class naming with snake_case, _tests/ for test directories, _test.py (never .test.py) for Python tests, I-prefixed Domain interfaces, fake_-prefixed Fakes, and an invariant intention.md to enable deterministic O(1) navigation and avoid Python import-resolution issues. :
* Nommer le dossier de tests _tests/ (avec underscore préfixe) pour le distinguer visuellement du code de production et le faire apparaître en premier dans le tri alphabétique
* Nommer les fichiers de test avec le suffixe _test.py (underscore) et jamais .test.py (point) — le point dans un nom de fichier Python casse la résolution des imports de modules
* Nommer tous les fichiers et dossiers Python en snake_case — jamais de camelCase, PascalCase ou tirets dans les noms de fichiers et dossiers
* Préfixer les classes et fichiers Fake avec fake_ pour identifier immédiatement qu'il s'agit d'une implémentation d'infrastructure pour le référentiel Test
* Préfixer les interfaces de Domain avec I (majuscule) pour les distinguer des implémentations et les placer dans domain/repositories/ ou application/X_service/
* Toujours nommer le fichier de spécification intention.md en minuscules sans variation — c'est le nom canonique du pont entre Meta-Domain et Code

Full standard is available here for further request: [SolidAI — Nomenclature : snake_case, Préfixes I/fake_, Dossier _tests/, Suffixe _test.py](.packmind/standards/solidai-nomenclature-snakecase-prefixes-ifake-dossier-tests-suffixe-testpy.md)

# Standard: SolidAI — Règles d'Import DDD : Isolation Domain, Application, Infrastructure

Enforce strict DDD import rules where domain/ never imports infrastructure/, application/ imports only Domain and uses infrastructure/<adapter>/fake/ in tests (never the Real adapter), and Infrastructure implements Domain interfaces with Real and Fake co-located to preserve layer isolation, independent testability, and parallel development. :
* Dans les tests de la couche application/, importer uniquement les Fakes depuis infrastructure/<adapter>/fake/ et jamais l'implémentation Real depuis infrastructure/<adapter>/<adapter>.py
* Implémenter les interfaces du Domain dans la couche Infrastructure (Real à la racine de l'adapter, Fake co-localisé dans <adapter>/fake/) sans créer de dépendances inverses vers l'Application
* Ne jamais importer de code infrastructure dans la couche domain/ — le Domain est pur et doit pouvoir être testé sans aucune dépendance externe

Full standard is available here for further request: [SolidAI — Règles d'Import DDD : Isolation Domain, Application, Infrastructure](.packmind/standards/solidai-regles-dimport-ddd-isolation-domain-application-infrastructure.md)

# Standard: SolidAI — Services Applicatifs : Structure, Interfaces et Ports Infrastructure

Définit la structure obligatoire d'un service applicatif : interface API inbound (i_api_), ports outbound dans application/X_service/ports/ (i_), DTOs, et co-localisation en trio atomique. :
* Créer les DTOs Request/Response systématiquement dans dtos/, même vides, avec @dataclass(frozen=True) — jamais de primitives éclatées en signature de méthode
* Déclarer les ports outbound vers l'Infrastructure dans application/X_service/ports/ — ces ports expriment un besoin technique de l'Application, pas un concept du Domain
* Distinguer explicitement le sens des interfaces : i_api_ est inbound (Adapters → Application), i_ dans ports/ est outbound (Application → Infrastructure)
* Préfixer l'interface inbound du service avec i_api_ pour la distinguer sans ambiguïté des ports outbound et des interfaces Domain
* Structurer chaque service applicatif avec le trio atomique obligatoire : intention.md, i_api_X_service.py, dtos/, X_service.py, et _tests/ — sans exception

Full standard is available here for further request: [SolidAI — Services Applicatifs : Structure, Interfaces et Ports Infrastructure](.packmind/standards/solidai-services-applicatifs-structure-interfaces-et-ports-infrastructure-1.md)

# Standard: SolidAI — Test Doubles DDD : Fake In-Memory vs Mock, Placement Infrastructure

Définit la distinction Fake (implémentation in-memory) vs Mock (vérification d'interactions), impose leur placement dans infrastructure/fake/ avec le trio atomique complet, et exige que chaque interface Domain ait un Fake correspondant testé. :
* Placer les Fakes dans infrastructure/fake/[catégorie]/[module]/ avec le trio atomique complet (intention.md + implémentation + _tests/)
* S'assurer que pour chaque interface Domain (IXxx), il existe un Fake correspondant dans infrastructure/fake/ — sans Fake, les tests Application nécessitent l'infrastructure réelle et le développement devient séquentiel
* Tester le Fake lui-même pour garantir qu'il respecte le contrat de l'interface Domain — cette validation est le garant de la propagation des tests vers l'infrastructure réelle
* Utiliser des Fakes (implémentation in-memory fonctionnelle) plutôt que des Mocks (vérification d'interactions) pour tester les use cases applicatifs — le Fake permet la vérification d'état, le Mock vérifie les appels

Full standard is available here for further request: [SolidAI — Test Doubles DDD : Fake In-Memory vs Mock, Placement Infrastructure](.packmind/standards/solidai-test-doubles-ddd-fake-in-memory-vs-mock-placement-infrastructure.md)
<!-- end: Packmind standards -->

---
> Source: [Luciosmic/AEFI_Acquisition](https://github.com/Luciosmic/AEFI_Acquisition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
