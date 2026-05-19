# Instructions Copilot - Panini-CoLabController

📍 **CONTEXTE LOCAL :** Tu te trouves actuellement dans le sous-module `modules/orchestration/colab`.
**Mission stricte :** Contrôle des sessions Google Colab et synchronisation du code sans interruption.

⚠️ **RÈGLES D'ANTI-DÉBORDEMENT :**
- Gère le driver Colab uniquement (connexion, upload, sync GitHub).
- Ne recrée jamais la logique des drivers local ou cloud (ExecutionOrchestrator).
- Ce module est en cours de fusion avec ExecutionOrchestrator — voir ADR-2025-08-30.

🗺️ **CARTOGRAPHIE DE L'ÉCOSYSTÈME PANINI :**
1. **Hub/Orchestrateur** (Racine) : Lien entre les modules. Ne contient que l'orchestration (`src/panini_colabmcp`).
2. **Panini-FS** (`modules/core/filesystem`) : Stockage FUSE3.
3. **Panini-SemanticCore** (`modules/core/semantic`) : Extraction dhātu.
4. **OntoWave** (`modules/ontowave`) : UX et UI.
5. **Panini-AttributionRegistry** (`modules/data/attribution`) : Traçabilité et provenance.
6. **Panini-AutonomousMissions** (`modules/missions/autonomous`) : Workflows IA.
7. **Panini-PublicationEngine** (`modules/publication/engine`) : Formatage/Export.
8. **Panini-UltraReactive** (`modules/reactive/ultra-reactive`) : Streaming temps réel.
9. **Panini-CloudOrchestrator** (`modules/orchestration/cloud`) : Infra et Déploiement.
10. **Panini-CoLabController** (`modules/orchestration/colab`) : Driver Colab.
11. **ExecutionOrchestrator** (`modules/orchestration/execution`) : Orchestration multi-driver.
12. **Panini-Research** (`research`) : Brouillons et laboratoire.

🔗 **RÈGLES GLOBALES :**
Le submodule `copilotage/` est présent dans ce dépôt et contient les directives partagées de l'écosystème Panini.
- **Journal de bord :** ce dépôt tient son propre journal dans `docs/journal-de-bord/YYYY-MM-DD.md`. Consulter `copilotage/regles/REGLES_JOURNAL_v1.md` pour les règles complètes.
- **Règles d'autonomie et de copilotage :** `copilotage/regles/REGLES_COPILOTAGE_v0.0.2.md`
- **Avant tout commit :** créer/mettre à jour `docs/journal-de-bord/$(date +%Y-%m-%d).md` puis stager le fichier.
