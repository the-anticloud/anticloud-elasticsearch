# 17_HOW_TO_UPDATE — ONLINE_RETAIL/ELASTICSEARCH

**Project:** ONLINE_RETAIL/ELASTICSEARCH
**License:** Unknown
**Files:** 0
**LOC:** 0

## Update Guide

### Checking for Updates

```bash
git fetch origin
git log HEAD..origin/main
```

### Applying Updates

```bash
git pull origin main
pip install -e .
```

### Verifying

Run the full test suite after updating:
```bash
python -m pytest
```

**Anticloud FZ LLE · 0-1.gg**
