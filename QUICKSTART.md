# Quick Start Guide - Email Finder v6

> Guide ultra-rapide pour un Claude qui arrive sans contexte.
> **Version actuelle: v6** (janvier 2026)

---

## 🚀 En 30 secondes

**C'est quoi ?** Outil de vérification d'emails professionnels via SMTP direct. Pas d'API externe, pas de crédits - juste du SMTP pur.

**Où ?**
- **Frontend** : https://email.auraia.ch (Basic Auth)
- **API** : http://192.3.81.106:8000
- **VPS** : `ssh root@192.3.81.106` (clé SSH configurée)

**Comment utiliser ?**
```bash
# Recherche par domaine + nom (méthode principale)
curl -X POST "http://192.3.81.106:8000/api/find-email" \
  -H "Content-Type: application/json" \
  -d '{"domain":"company.com","fullName":"John Doe"}'

# Vérifier un email spécifique (v6 NEW)
curl -X POST "http://192.3.81.106:8000/api/check-email" \
  -H "Content-Type: application/json" \
  -d '{"email":"john@company.com"}'

# Health check (v6 NEW)
curl http://192.3.81.106:8000/health
```

---

## 📚 Documents à lire (dans l'ordre)

1. **CLAUDE.md** ← **COMMENCE ICI** (guide pour Claude Code)
2. **README.md** - Vue d'ensemble complète
3. **docs/API_USAGE.md** - Comment utiliser l'API
4. **docs/ARCHITECTURE.md** - Détails techniques
5. **docs/ROADMAP.md** - Features et état du projet
6. **docs/MAINTENANCE.md** - Opérations quotidiennes

---

## ✨ Features v6 (Janvier 2026)

### 1. Check Email Endpoint
Valide un email spécifique directement, avec fallback automatique vers recherche domaine:
```bash
curl -X POST "http://192.3.81.106:8000/api/check-email" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@company.com","fullName":"Test User"}'
```

### 2. Bulk Search JSON
Batch processing via JSON (paste-from-spreadsheet):
```bash
curl -X POST "http://192.3.81.106:8000/api/bulk-search-json" \
  -H "Content-Type: application/json" \
  -d '{
    "searches": [
      {"domain":"company1.com","fullName":"John Doe"},
      {"domain":"company2.com","fullName":"Jane Smith"}
    ]
  }'
```

### 3. Health Endpoint
Monitoring avec config + cache stats + version:
```bash
curl http://192.3.81.106:8000/health
# Returns: status, version, config, cache metrics, system info
```

### 4. API Robustness
- **Retry Logic**: 3 tentatives avec exponential backoff (1s → 2s → 4s)
- **MX Fallback**: Essaie jusqu'à 2 MX servers si premier unreachable
- **Centralized Config**: `backend/config.py` pour toutes les env vars

---

## 🛠️ Commandes essentielles

### SSH VPS
```bash
ssh root@192.3.81.106
```

### Vérifier que tout marche
```bash
# Status service
ssh root@192.3.81.106 "systemctl status email-finder"

# Test API
curl http://192.3.81.106:8000/api/cache/stats

# Test recherche
curl -X POST "http://192.3.81.106:8000/api/find-email" \
  -H "Content-Type: application/json" \
  -d '{"domain":"auraia.ch","fullName":"Adrian Turion"}'
```

### Redémarrer le service
```bash
ssh root@192.3.81.106 "systemctl restart email-finder"
```

### Voir les logs
```bash
ssh root@192.3.81.106 "tail -f /root/logs/email_finder.log"
```

### Déployer une modification
```powershell
.\scripts\deploy.ps1
```

---

## 📁 Structure projet v6

```
vps-email-finder/
├── CLAUDE.md              ← **LIS EN PREMIER** (guide pour Claude)
├── README.md              ← Vue d'ensemble complète
├── QUICKSTART.md          ← Ce fichier
├── DEPLOYMENT_NOTES.md    ← Notes de déploiement v6
├── .gitignore
│
├── backend/               ← API Python FastAPI
│   ├── config.py          ← **v6 NEW** Config centralisée
│   ├── core/
│   │   ├── email_finder.py   ← Logique principale (retry, MX fallback)
│   │   ├── mx_cache.py       ← Cache MX records (1h TTL)
│   │   └── logger.py         ← Logging structuré
│   ├── tests/             ← 37 tests (91.9% pass)
│   │   ├── test_api.py
│   │   └── test_email_finder.py
│   ├── main.py            ← API endpoints (find, check, bulk, health)
│   ├── models.py          ← Pydantic models (v6: +3 nouveaux)
│   ├── database.py        ← SQLite ORM
│   └── requirements.txt
│
├── frontend/              ← React + shadcn/ui
│   ├── components.json    ← shadcn config (New York style)
│   ├── src/
│   │   ├── components/
│   │   │   ├── SearchForm.tsx    ← Mode switch v6
│   │   │   ├── BulkSearch.tsx    ← Paste-from-sheet v6
│   │   │   ├── HistoryList.tsx   ← Copy buttons v6
│   │   │   └── ui/              ← shadcn components
│   │   └── App.tsx
│   └── UI_REDESIGN_GUIDE.md
│
├── docs/                  ← Documentation
│   ├── API_USAGE.md
│   ├── ARCHITECTURE.md
│   ├── MAINTENANCE.md
│   ├── ROADMAP.md         ← Features roadmap (ALL DONE ✅)
│   └── SESSION_NOTES.md
│
├── scripts/
│   └── deploy.ps1
│
├── deploy_v6.ps1          ← **Script déploiement v6 automatique**
│
└── bulk-search-success.png ← Screenshot test UI
```

**Fichiers critiques v6**:
- `backend/config.py` - Config retry/MX/cache
- `backend/main.py` - +3 endpoints (health, check-email, bulk-json)
- `backend/core/email_finder.py` - Retry + MX fallback
- `CLAUDE.md` - Guide complet pour futures instances Claude

---

## ⚡ Actions rapides

### Problème fréquent #1 : Service down
```bash
ssh root@192.3.81.106 "systemctl restart email-finder"
```

### Problème fréquent #2 : API retourne 500
```bash
# Voir les logs
ssh root@192.3.81.106 "tail -50 /root/logs/email_finder.log | grep ERROR"
```

### Problème fréquent #3 : Cache ne fonctionne pas
```bash
# Vérifier le fichier
ssh root@192.3.81.106 "ls -la /root/vps-email-finder/backend/core/mx_cache.py"

# Redéployer si manquant
scp backend/core/mx_cache.py root@192.3.81.106:/root/vps-email-finder/backend/core/
ssh root@192.3.81.106 "systemctl restart email-finder"
```

---

## 🧪 Test complet (2 minutes) - v6

```bash
# 1. Service up ?
ssh root@192.3.81.106 "systemctl is-active email-finder"
# Doit retourner : active

# 2. Health check (v6 NEW)
curl http://192.3.81.106:8000/health
# Doit retourner : {"status":"healthy","version":"v6",...}

# 3. API répond ?
curl http://192.3.81.106:8000/docs
# Doit retourner : HTML

# 4. Cache fonctionne ?
curl http://192.3.81.106:8000/api/cache/stats
# Doit retourner : JSON avec hits/misses/hit_rate

# 5. Find email marche ?
curl -X POST "http://192.3.81.106:8000/api/find-email" \
  -H "Content-Type: application/json" \
  -d '{"domain":"auraia.ch","fullName":"Adrian Turion"}'
# Doit retourner : status: "valid", email: "adrian.turion@auraia.ch"

# 6. Check email marche ? (v6 NEW)
curl -X POST "http://192.3.81.106:8000/api/check-email" \
  -H "Content-Type: application/json" \
  -d '{"email":"adrian.turion@auraia.ch"}'
# Doit retourner : status: "valid"

# 7. Bulk search JSON marche ? (v6 NEW)
curl -X POST "http://192.3.81.106:8000/api/bulk-search-json" \
  -H "Content-Type: application/json" \
  -d '{"searches":[{"domain":"google.com","fullName":"Test User"}]}'
# Doit retourner : total: 1, results: [{status: "catch_all",...}]
```

Si tout marche ✅ → Système opérationnel v6

**Smoke test rapide (v6)**:
```bash
curl -s http://192.3.81.106:8000/health | grep '"status":"healthy"' && echo "✅ OK" || echo "❌ FAIL"
```

---

## 📊 Métriques importantes (v6)

```bash
# Health check complet (v6)
curl -s http://192.3.81.106:8000/health | python -m json.tool
# Retourne: status, version, config, cache, system

# Cache hit rate (> 50% = bon)
curl -s http://192.3.81.106:8000/api/cache/stats
# Retourne: {"hit_rate":"66.7%","cached_domains":5,...}

# Config runtime (v6)
curl -s http://192.3.81.106:8000/health | grep -o '"max_retries":[0-9]*'
# Doit être: "max_retries":3

# Historique recherches
curl -s http://192.3.81.106:8000/api/history?limit=1 | grep -o '"id":[0-9]*'
# Nombre = total recherches depuis dernier wipe DB

# Espace disque (> 2 GB libre = bon)
ssh root@192.3.81.106 "df -h /"

# Version déployée
curl -s http://192.3.81.106:8000/health | grep -o '"version":"v[0-9]*"'
# Doit être: "version":"v6"
```

---

## 🎯 Modifications courantes (v6)

### Modifier la logique de recherche
```
Fichier : backend/core/email_finder.py
Fonction : find_email() ou check_email()
Après : git commit + git push + .\deploy_v6.ps1
```

### Ajouter un endpoint API
```
Fichier : backend/main.py
Modèle : backend/models.py (si nouveau request model)
Après : git commit + git push + .\deploy_v6.ps1
```

### Changer les patterns d'email
```
Fichier : backend/core/email_finder.py
Fonction : generate_patterns() (ligne ~70)
Après : git commit + git push + .\deploy_v6.ps1
```

### Modifier retry/MX config (v6 NEW)
```
Fichier : backend/config.py
Variables : SMTP_MAX_RETRIES, MAX_MX_SERVERS, etc.
OU via env vars dans backend/.env sur VPS
Après : redémarrer service (pas besoin rebuild)
```

### Modifier le cache TTL
```
Fichier : backend/config.py
Variable : MX_CACHE_TTL (default: 3600)
OU via env var : MX_CACHE_TTL=7200
Après : systemctl restart email-finder
```

### Déployer rapidement (v6)
```powershell
# Script automatique avec tests
.\deploy_v6.ps1

# OU manuel
git add -A
git commit -m "feat: description"
git push
ssh root@192.3.81.106 "cd /root/vps-email-finder && git pull && systemctl restart email-finder"
```

---

## 🔑 Credentials

### SSH VPS
- **Host** : 192.3.81.106
- **User** : root
- **Auth** : Clé SSH (~/.ssh/id_ed25519)
- **Pas de mot de passe** ✅

### Basic Auth Frontend
- **URL** : https://email.auraia.ch
- **User** : admin
- **Password** : Demander à Adrian

### API Backend
- **Pas d'authentification** (usage interne)

---

## 🆘 En cas de panique

**Service complètement cassé ?**
```bash
# 1. Redémarrer
ssh root@192.3.81.106 "systemctl restart email-finder"

# 2. Vérifier les logs
ssh root@192.3.81.106 "journalctl -u email-finder -n 100"

# 3. Si toujours cassé, contacter Adrian
```

**VPS inaccessible ?**
1. Panel RackNerd : https://my.racknerd.com
2. Console VNC
3. Reboot

**Database corrompue ?**
```bash
# Restaurer le dernier backup
ssh root@192.3.81.106 "ls -lht /root/backup_*.tar.gz | head -1"
# Extraire et restaurer (voir MAINTENANCE.md)
```

---

## 📞 Contacts

**Projet maintenu par** : Adrian Turion
**Email** : adrian.turion@auraia.ch
**VPS Hébergeur** : RackNerd (panel: https://my.racknerd.com)

---

**Tu as lu ce fichier ? Maintenant lis README.md pour la vue complète ! 📖**
