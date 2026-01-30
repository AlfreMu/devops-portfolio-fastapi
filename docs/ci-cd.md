# CI/CD (GitHub Actions + GHCR + k3s en EC2)

Este repo usa GitHub Actions para:
- **CI:** validar que `backend` y `frontend` buildéan.
- **CD:** build + push a GHCR y deploy automático en k3s (EC2).

---

## CI — `.github/workflows/ci.yml`
Se ejecuta en:
- `pull_request` hacia `master`
- `push` a `master`

Hace:
- Build de imágenes Docker (backend/frontend).
- No publica imágenes.

Objetivo:
- que un PR no rompa el build.

---

## CD — `.github/workflows/cd.yml`
Se ejecuta en:
- `push` a `master` (tambien cuando se mergea a master).

Flujo:
1. **Build + push** de imágenes a GHCR con tags inmutables:
   - `ghcr.io/<owner-lc>/devops-portfolio-fastapi-backend:sha-<commit>`
   - `ghcr.io/<owner-lc>/devops-portfolio-fastapi-frontend:sha-<commit>`
2. **Deploy** desde un runner self-hosted en la EC2 usando `kubectl`:
   - actualiza imágenes con `kubectl set image`
   - corre migraciones (Job) y espera que terminen
   - verifica rollout (`kubectl rollout status`)

---

## Cómo llega a “producción” (EC2)
- GitHub Actions publica imágenes en **GHCR**.
- k3s en la EC2 hace **pull** de esas imágenes y recrea los pods.
- Traefik expone:
  - `/` → frontend
  - `/api` → backend

URLs:
- `http://54.227.12.89/login`
- `http://54.227.12.89/docs`
