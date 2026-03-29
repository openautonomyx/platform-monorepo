# OpenAutonomyX Platform

Self-hosted open-source platform stack running on `openautonomyx.com`.

## Services

| Service | Directory | URL | Port |
|---------|-----------|-----|------|
| **Lago** (Billing) | `billing/` | billing.openautonomyx.com | 3100/3101 |
| **GlitchTip** (Error Tracking) | `error-tracking/` | sentry.openautonomyx.com | 4300 |
| **Novu** (Notifications) | `notifications/` | notify.openautonomyx.com | 3340-3342 |
| **Cal.com** (Scheduling) | `scheduling/` | cal.openautonomyx.com | 3350 |
| **Postiz** (Social) | `social/` | social.openautonomyx.com | 3360 |
| **OnlyOffice** (Docs) | `docs/` | docs.openautonomyx.com | 3650 |
| **Stalwart** (Mail) | `mail/` | mail.openautonomyx.com | 25/465/993/4500 |
| **Uptime Kuma** (Monitoring) | `monitoring/` | status.openautonomyx.com | 3500 |

## Other services (managed separately)

| Service | URL | Port |
|---------|-----|------|
| Liferay Portal | www.openautonomyx.com | 8080 |
| n8n | api.openautonomyx.com | 3800 |
| ERPNext | erp.openautonomyx.com | 3200 |
| Nextcloud | next.openautonomyx.com | 3600 |
| Mautic | marketing.openautonomyx.com | 4100 |
| Matrix Synapse | chat.openautonomyx.com | 3700 |
| Matomo | analytics.openautonomyx.com | 4200 |
| Evolution API | evo.openautonomyx.com | 4000 |
| FreePBX | pbx.openautonomyx.com | 3400 |
| Restreamer | stream.openautonomyx.com | 3300 |

## Quick Start

```bash
cp .env.example .env
# Edit .env with your values
cd billing && docker compose up -d
cd ../error-tracking && docker compose up -d
# ... etc
```

## Nginx

All Nginx reverse proxy configs are in `nginx/`. Copy to `/etc/nginx/sites-enabled/` and run `certbot --nginx -d <domain>`.

## SMTP

All services use Stalwart Mail Server at `mail.openautonomyx.com:5870` (STARTTLS).
