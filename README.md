# 🧰 Ultimate Updater avec envoi Discord

Ce script met à jour automatiquement ton système et envoie un rapport complet sur Discord, y compris en pièce jointe, tout en respectant la limite des 2000 caractères pour le message.

---

## 📦 Installation

### 1️⃣ Cloner ou télécharger le script

```bash
mkdir -p /home/scripts
cd /home/scripts
wget https://raw.githubusercontent.com/Ssyleric/ultimate-updater/main/update.sh
chmod +x update.sh
```

### 2️⃣ Installer les dépendances

```bash
apt update -y && apt install -y jq curl
```

---

## 🔧 Configuration de l'envoi Discord

Dans ton script `update.sh`, ajoute ou modifie la section suivante **à la fin** pour utiliser la méthode standard **jq + pièce jointe** :

```bash
# Envoi du log complet vers Discord (méthode standard jq + pièce jointe)
WEBHOOK="https://discord.com/api/webhooks/TON_WEBHOOK_ICI"
LOG_PATH="/var/log/ultimate-updater.log"

HEADER="🧰 Rapport update.sh – $(hostname) — $(date '+%Y-%m-%d %H:%M')"
MAXLEN=2000

# Bloc de code Markdown (vraies nouvelles lignes)
PREFIX="$HEADER

\`\`\`"
SUFFIX="\`\`\`"

# On lit un extrait raisonnable du log (évite les énormes fichiers)
RAW_LOG="$(tail -n 500 "$LOG_PATH" 2>/dev/null || true)"

ENCODED=""
FINAL_ENCODED=""
while IFS= read -r line || [[ -n "${line:-}" ]]; do
  ENCODED="${FINAL_ENCODED}${line}"$'\n'
  CHAR_COUNT=$(printf "%s\n%s\n%s\n" "$PREFIX" "$ENCODED" "$SUFFIX" | wc -m)
  [[ $CHAR_COUNT -gt $MAXLEN ]] && break
  FINAL_ENCODED="$ENCODED"
done <<< "$RAW_LOG"

FINAL_MESSAGE=$(printf "%s\n%s\n%s\n" "$PREFIX" "$FINAL_ENCODED" "$SUFFIX")
PAYLOAD_JSON="$(printf "%s" "$FINAL_MESSAGE" | jq -Rs '{content: .}')"

# Envoi : contenu + fichier joint
curl -s -X POST "$WEBHOOK"   --form-string "payload_json=${PAYLOAD_JSON}"   -F "file=@${LOG_PATH};type=text/plain" >/dev/null
```

---

## 🚀 Utilisation

### Exécuter manuellement

```bash
bash /home/scripts/update.sh
```

### Automatiser avec `cron`

```bash
crontab -e
```

Ajoute la ligne suivante pour exécuter chaque jour à 4h du matin :

```bash
0 4 * * * /home/scripts/update.sh
```

---

## 📜 Notes importantes

- ✅ Méthode **100% compatible Discord** (respecte la limite de **2000 caractères** par message)
- 📎 Envoie **un fichier joint** avec le rapport complet
- 📝 Utilise `jq -Rs` pour conserver les retours à la ligne
- 📌 Dépendances : `jq` et `curl`
- 📂 Log complet enregistré dans `/var/log/ultimate-updater.log`

---

## 📖 Exemple de sortie sur Discord

```
🧰 Rapport update.sh – pve-server — 2025-08-15 11:45

[Info] Mise à jour lancée...
[Info] Mise à jour terminée avec succès.
```

Pièce jointe : **ultimate-updater.log** (log complet)
