<div align="center">

# 📬 AI Law Daily Digest

**An automated daily pipeline: fetch AI × law papers & news → distill with an LLM → email it to you**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-3776ab.svg?logo=python&logoColor=white)](https://python.org)
[![Zero Dependencies](https://img.shields.io/badge/dependencies-0-success.svg)](#)
[![LLM](https://img.shields.io/badge/LLM-GLM--5.3--flash-722ed1.svg)](https://open.bigmodel.cn)

**arXiv papers · Hacker News · Solidot · GitHub repos · (optional Twitter RSS)**

中文 · [English](README.en.md)

</div>

---

## How it works

```
Every day at 08:00 (systemd timer)
   ├─ arXiv API        papers: computational law / legal AI / AI governance
   ├─ Hacker News      community discussions (Algolia API, ranked by points)
   ├─ Solidot RSS      tech news (keyword-filtered)
   ├─ GitHub Search    new repos in legal-AI topics
   └─ Twitter RSS      optional — just set TWITTER_RSS_URLS
        ↓
   dedupe (90-day sliding window) → LLM writes "today's highlights + per-item commentary"
        ↓
   HTML email + local Markdown archive
```

The LLM is an enhancement layer — if it fails, the digest falls back to raw abstracts; a failing source never breaks the run. The whole thing is **pure Python standard library, zero third-party dependencies**.

## Quick start

```bash
cp aidigest.env.example aidigest.env   # SMTP credentials + LLM API key
python3 aidigest.py --dry-run          # trial: fetch + distill + archive, no email
python3 aidigest.py                    # full run: fetch + distill + send
python3 aidigest.py --no-llm           # skip the LLM, use raw abstracts
```

Config (`aidigest.env`): recipient, SMTP (465/587 auto-selected), LLM key, lookback window (`LOOKBACK_HOURS`), item cap (`MAX_ITEMS`), optional Twitter RSS URLs.

## Server deployment (systemd)

```ini
# /etc/systemd/system/aidigest.service
[Unit]
Description=AI Law Daily Digest
After=network-online.target

[Service]
Type=oneshot
WorkingDirectory=/opt/aidigest
ExecStart=/usr/bin/python3 /opt/aidigest/aidigest.py
```

```ini
# /etc/systemd/system/aidigest.timer
[Timer]
OnCalendar=*-*-* 08:00:00
Persistent=true
RandomizedDelaySec=300
```

```bash
sudo systemctl enable --now aidigest.timer   # runs daily at 08:00
systemctl list-timers aidigest*              # next run
journalctl -u aidigest -n 50                 # logs
```

## Notes

- Email goes over SMTP 465/587 (works on clouds that block port 25); any provider with an app password works
- Dedup state lives in `state.json` (auto-pruned to 90 days); digests are archived under `digests/`
- From a mainland-China server Twitter is unreachable, hence HN/Solidot cover the overseas layer; plug in an RSSHub/Nitter URL anytime via config

## License

[MIT](LICENSE)
