# Module Colab Controller

**Status**: 🚧 En développement actif  
**Priorité**: ⭐⭐⭐ Haute (Abonnement Colab Pro disponible)  
**Dernière mise à jour**: 2025-11-12

> **Note de migration (2025-08-30)**: Ce module sera fusionné avec `modules/cloud-orchestrator` dans `ExecutionOrchestrator` (drivers: local, colab, cloud). Voir `ARCHITECTURE/ADR-2025-08-30-modular-restructuring-option-b.md`.
>
> **Mise à jour (2025-11-12)**: Priorisation élevée suite à disponibilité **Google Colab Pro** - GPU prioritaire, RAM étendue, sessions 24h. Voir `copilotage/knowledge/RESSOURCES_CLOUD_DISPONIBLES.md` pour stratégie d'utilisation.

## 🎯 Mission

Orchestrer et automatiser l'utilisation de **Google Colab Pro** pour maximiser l'efficacité des expérimentations de recherche sur le projet Panini.

## 🎁 Ressources Colab Pro Disponibles

### GPU Prioritaires
- **T4**: 16GB VRAM - Fine-tuning
- **P100**: 16GB VRAM - Training modèles moyens
- **V100**: 16GB VRAM - Haute performance
- **A100**: 40GB VRAM - Modèles larges (prioritaire mais rare)

### Capacités Étendues
- **RAM**: Jusqu'à 32GB+ (vs 12GB gratuit)
- **Session**: 24h continues (vs 12h gratuit)
- **Accès**: Prioritaire lors de forte demande

## 📋 Fonctionnalités Cibles

### Phase 1: Automation de Base ⭐ PRIORITAIRE
- [ ] **Lancement automatique** de notebooks depuis local
- [ ] **Gestion de queue** de jobs (séquentiel/parallèle)
- [ ] **Monitoring GPU**: détection type, utilisation VRAM
- [ ] **Récupération automatique** des résultats vers local/Drive
- [ ] **Logging centralisé** des exécutions

### Phase 2: Pipeline ML
- [ ] **Templates standardisés** par type d'expérimentation
- [ ] **Checkpointing automatique** toutes les N minutes
- [ ] **Gestion d'erreurs** et retry automatique
- [ ] **Snapshot état GPU** avant crash
- [ ] **Cost tracking** (temps GPU utilisé vs quota)

### Phase 3: Intégration Advanced
- [ ] **CI/CD**: Git push → Auto-test sur Colab
- [ ] **Dashboard temps réel**: Monitoring multi-notebooks
- [ ] **Optimization scheduler**: Allocation intelligente des ressources
- [ ] **Notebook versioning**: Sync avec Git
- [ ] **Collaborative mode**: Partage de sessions

## 🏗️ Architecture Proposée

```
modules/orchestration/colab/
├── src/
│   ├── launcher.py          # Lance notebooks sur Colab
│   ├── monitor.py           # Surveille exécutions
│   ├── queue_manager.py     # Gère file d'attente
│   ├── result_collector.py  # Récupère outputs
│   └── templates/           # Notebooks pré-configurés
│       ├── training_template.ipynb
│       ├── analysis_template.ipynb
│       └── validation_template.ipynb
├── config/
│   ├── colab_credentials.json  (git-ignored)
│   └── execution_config.yaml
├── logs/
│   └── execution_history.jsonl
├── tests/
│   └── test_launcher.py
└── docs/
    ├── SETUP.md
    └── API.md
```

## 🚀 Use Cases Panini

### 1. Optimisation Dictionnaire Panlang
```python
# Workflow automatisé pour hillclimbing 10000+ itérations
from colab_controller import launch_notebook, monitor

job = launch_notebook(
    notebook='panlang_hillclimbing.ipynb',
    params={'iterations': 10000, 'checkpoint_freq': 100},
    gpu_required='V100',
    timeout_hours=20
)

results = monitor(job, callback=save_checkpoints)
```

### 2. Analyse Corpus Multilingue
```python
# Traitement parallèle sur plusieurs notebooks
jobs = []
for corpus in ['fr', 'en', 'es', 'de', 'sanskrit', 'chinois']:
    job = launch_notebook(
        notebook='corpus_analysis.ipynb',
        params={'language': corpus, 'dataset': f'corpus_{corpus}.json'},
        gpu_required='T4'
    )
    jobs.append(job)

results = wait_all(jobs)
merge_results(results, output='multilingual_analysis.json')
```

### 3. Entraînement Embeddings Sémantiques
```python
# Pipeline complet avec backup automatique vers Google One
job = launch_notebook(
    notebook='train_embeddings.ipynb',
    params={
        'model': 'sentence-transformers',
        'corpus': 'gs://panini-datasets/semantic_corpus.json',
        'epochs': 50,
        'primitives': 'NSM'  # Natural Semantic Metalanguage
    },
    gpu_required='A100',  # Prioritaire si disponible
    auto_backup_to='gdrive:Panini/models/embeddings/',
    checkpoint_freq_minutes=30
)
```

## 📝 Template Notebook Standard

Chaque notebook Panini devrait inclure ce header:

```python
# === PANINI COLAB STANDARD HEADER ===
import os
import sys
from pathlib import Path
from google.colab import drive
import torch
import json
import pickle
from datetime import datetime

# Montage Drive (Google One)
drive.mount('/content/drive')
PANINI_ROOT = Path('/content/drive/MyDrive/Panini')
sys.path.insert(0, str(PANINI_ROOT))

# Vérification GPU
gpu_name = torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU'
vram_gb = torch.cuda.get_device_properties(0).total_memory / 1e9 if torch.cuda.is_available() else 0
print(f"🎯 GPU: {gpu_name}")
print(f"📊 VRAM: {vram_gb:.2f} GB")
print(f"💾 RAM: {psutil.virtual_memory().total / 1e9:.2f} GB")

# Logging automatique
log_dir = PANINI_ROOT / 'logs' / 'colab_executions'
log_dir.mkdir(parents=True, exist_ok=True)
log_file = log_dir / f"{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"

# Checkpoint automatique
def save_checkpoint(data, name='checkpoint'):
    checkpoint_dir = PANINI_ROOT / 'checkpoints' / name
    checkpoint_dir.mkdir(parents=True, exist_ok=True)
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    path = checkpoint_dir / f"{timestamp}.pkl"
    with open(path, 'wb') as f:
        pickle.dump(data, f)
    print(f"✅ Checkpoint sauvegardé: {path}")

# === VOTRE CODE ICI ===
```

## 🔧 Configuration Requise

### Installation
```bash
pip install google-colab-controller google-auth google-api-python-client
```

### Credentials
1. Créer OAuth 2.0 credentials sur Google Cloud Console
2. Activer Colab API
3. Stocker dans `config/colab_credentials.json` (git-ignored)

### Variables d'environnement
```bash
export GOOGLE_COLAB_CREDENTIALS=/path/to/credentials.json
export GOOGLE_DRIVE_ROOT=/content/drive/MyDrive/Panini
export PANINI_BACKUP_BUCKET=gs://panini-research-backup
```

## 🎯 Priorités de Développement

### Sprint 1 (Semaine 1-2): MVP ⭐ URGENT
1. ✅ Documentation des ressources disponibles
2. [ ] Script Python basique pour lancer un notebook via API
3. [ ] Monitoring simple (status: running/completed/failed)
4. [ ] Récupération résultats depuis Drive

### Sprint 2 (Semaine 3-4): Automation
5. [ ] Queue manager pour jobs multiples
6. [ ] Templates standardisés (3 types: training, analysis, validation)
7. [ ] Logging centralisé dans `copilotage/journal/`

### Sprint 3 (Mois 2): Production
8. [ ] Dashboard web monitoring
9. [ ] CI/CD integration
10. [ ] Cost optimization et tracking quota

## 📚 Références

- **Abonnements**: Voir `copilotage/knowledge/RESSOURCES_CLOUD_DISPONIBLES.md`
- [Colab Pro Features](https://colab.research.google.com/signup)
- [Google Colab API (unofficiel)](https://github.com/googlecolab/colabtools)
- [Best Practices](https://research.google.com/colaboratory/faq.html)
- **Architecture**: Voir `ARCHITECTURE_STANDARD.md`

## 🤝 Contribution

Pour contribuer à ce module:
1. Créer issue avec tag `colab-controller`
2. Branche: `feature/colab-{feature-name}`
3. Tests requis avant PR
4. Documentation à jour dans `docs/`

---

**Maintenu par**: Équipe Infrastructure Panini  
**Contact**: Voir `docs/PROJECT_OVERVIEW.md`
