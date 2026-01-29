# Kubernetes (k3s en AWS) — Documentación Técnica

Este documento describe cómo está desplegado el proyecto en Kubernetes usando **k3s** sobre una **EC2** en AWS, y cómo operar los componentes principales.

> Objetivo: que cualquier perfil técnico pueda entender el setup en 5–10 minutos y reproducir el deploy de forma controlada.

---

## 1) Overview

Componentes en el namespace `portfolio`:

- **backend** (FastAPI) → Deployment + Service (ClusterIP)
- **frontend** (web) → Deployment + Service (ClusterIP)
- **postgres** (DB) → Deployment + PVC + Service (ClusterIP)
- **ingress** (Traefik) → expone tráfico público:
  - `/` → frontend
  - `/api` → backend
  - `/docs` → backend swagger

Además:
- **Job `backend-migrations`**: ejecuta `alembic upgrade head` y `app.initial_data` (migraciones + datos iniciales)

---

## 2) Estructura de manifiestos

Los manifests del portfolio viven bajo:
k8s/portfolio/
├── backend/
│ ├── deployment.yaml
│ ├── service.yaml
│ ├── configmap.yaml
│ ├── secret.yaml
│ └── job-migrations.yaml
├── frontend/
│ ├── deployment.yaml
│ └── service.yaml
├── postgres/
│ ├── deployment.yaml
│ ├── service.yaml
│ └── pvc.yaml
└── traefik/
└── (opcional) CRDs / IngressRoute / Middleware si se usan

> Nota: el objetivo de esta estructura es separar el trabajo propio del contenido upstream y mantener una evolución por fases.

---

## 3) Namespace y recursos

Namespace:
- `portfolio`

Comandos útiles:
kubectl get ns
kubectl -n portfolio get pods,svc,ingress,ingressroute,middleware,job
kubectl -n portfolio describe deploy backend

## 4) Estructura de manifiestos
Imágenes y versionado (tags inmutables)

Las imágenes se publican en GHCR.
Para despliegues “reproducibles”, los deployments usan tags inmutables (ej: sha-...) en lugar de latest, para evitar latest en producción.

Ver imagen actual:

kubectl -n portfolio get deploy backend -o=jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
kubectl -n portfolio get deploy frontend -o=jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

## 5) Variables de entorno (ConfigMap/Secret)

El backend toma configuración desde:

backend-config (ConfigMap)

backend-secret (Secret)

Se inyectan vía envFrom en el deployment.

Ver qué hay definido:

kubectl -n portfolio get cm backend-config -o yaml
kubectl -n portfolio get secret backend-secret -o yaml

Importante: no versionar secretos reales. En el repo se deja un ejemplo o placeholders si corresponde.

## 6) Migraciones de base de datos (Job)

Problema real resuelto: al iniciar, el backend fallaba por tablas inexistentes (relation "user" does not exist).
Se resolvió ejecutando migraciones Alembic + carga de datos iniciales.

Job:
k8s/portfolio/backend/job-migrations.yaml

Ejecutar migraciones (manual / on-demand):

kubectl -n portfolio delete job backend-migrations --ignore-not-found
kubectl -n portfolio apply -f k8s/portfolio/backend/job-migrations.yaml
kubectl -n portfolio get jobs
POD=$(kubectl -n portfolio get pod -l job-name=backend-migrations -o jsonpath='{.items[0].metadata.name}')
kubectl -n portfolio logs "$POD"

Salida esperada:
✅ migrations + initial_data OK

## 7) Exposición pública (Traefik / Ingress)
Este cluster usa Traefik (incluido en k3s) como Ingress Controller.

Reglas de ruteo:

/ → frontend (service:80)

/api → backend (service:8000)

/docs → manejado por backend (FastAPI)

Ver recursos de entrada:

kubectl -n portfolio get ingress
kubectl -n portfolio get ingressroute,middleware

Si se usa IngressRoute/Middleware (CRDs de Traefik), asegurarse de que existan los CRDs instalados:

kubectl get crd | grep -i traefik

## 8) Operación y troubleshooting
Rollouts
kubectl -n portfolio rollout status deploy/backend
kubectl -n portfolio rollout status deploy/frontend

Logs
kubectl -n portfolio logs deploy/backend --tail=200
kubectl -n portfolio logs deploy/frontend --tail=200

ImagePullBackOff / ErrImagePull

Checklist:

- el tag existe en GHCR.

- el owner en ghcr.io/<owner>/... está en minúsculas

el deployment apunta al tag correcto (no duplicado/concatenado)

si el package es privado: configurar imagePullSecret

9) Próximas mejoras (Kubernetes)

Healthchecks (liveness/readiness) para backend/frontend

Requests/limits

HPA (si aplica)

TLS (HTTPS) con certificados

Automatizar CD (deploy automático) y/o GitOps


---

## Paso 2 — Actualizar los README chicos (para que no mientan)

### A) `k8s/portfolio/README.md`
Tu texto está bien como “principios”. Yo le agregaría 3 líneas para guiar:

**Pegá esto abajo** del texto que ya tenés:

```md
Estructura:
- `backend/`, `frontend/`, `postgres/`: recursos por servicio (Deployment/Service/PVC/Job)
- `traefik/`: recursos de entrada (Ingress/IngressRoute/Middleware si aplica)

Aplicación:
```bash
kubectl -n portfolio apply -f k8s/portfolio/postgres
kubectl -n portfolio apply -f k8s/portfolio/backend
kubectl -n portfolio apply -f k8s/portfolio/frontend
kubectl -n portfolio apply -f k8s/portfolio/traefik   # si existe


### B) `.github/workflows/README.md`
Ahí hay una inconsistencia: tu README dice “CD al mergear a main”, pero vos venís trabajando en `master`. En tu README principal también se menciona CD como “próximo paso”.

Propongo dejarlo así, neutro y cierto:

```md
# Workflows (portfolio)

- `ci.yml`: build y validaciones en PRs y/o pushes (checks obligatorios con branch protection).
- `cd.yml` (pendiente): deploy automático cuando se mergea a la rama protegida (master/main).

> Nota: la rama protegida puede ser `master` o `main` según la configuración del repo.

