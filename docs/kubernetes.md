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
```text
k8s/portfolio/
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── job-migrations.yaml
├── frontend/
│   ├── deployment.yaml
│   └── service.yaml
├── postgres/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── pvc.yaml
└── traefik/
    └── (opcional) CRDs / IngressRoute / Middleware si se usan
```

> Nota: el objetivo de esta estructura es separar el trabajo propio del contenido upstream y mantener una evolución por fases.

---

## 3) Namespace y recursos

Namespace:
- `portfolio`

Comandos útiles:
```bash
kubectl get ns
kubectl -n portfolio get pods,svc,ingress,ingressroute,middleware,job
kubectl -n portfolio describe deploy backend
```

## 4) Estructura de manifiestos
Imágenes y versionado (tags inmutables)

Las imágenes se publican en GHCR.
Para despliegues “reproducibles”, los deployments usan tags inmutables (ej: sha-...) en lugar de latest, para evitar latest en producción.

Ver imagen actual:
```bash
kubectl -n portfolio get deploy backend -o=jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
kubectl -n portfolio get deploy frontend -o=jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

## 5) Variables de entorno (ConfigMap/Secret)

El backend toma configuración desde:

backend-config (ConfigMap)

backend-secret (Secret)

Se inyectan vía envFrom en el deployment.

Ver qué hay definido:
```bash
kubectl -n portfolio get cm backend-config -o yaml
kubectl -n portfolio get secret backend-secret -o yaml
```

Importante: no versionar secretos reales. En el repo se deja un ejemplo o placeholders si corresponde.

## 6) Migraciones de base de datos (Job)

Problema real resuelto: al iniciar, el backend fallaba por tablas inexistentes (relation "user" does not exist).
Se resolvió ejecutando migraciones Alembic + carga de datos iniciales.

Job:
k8s/portfolio/backend/job-migrations.yaml

Ejecutar migraciones (manual / on-demand):
```bash
kubectl -n portfolio delete job backend-migrations --ignore-not-found
kubectl -n portfolio apply -f k8s/portfolio/backend/job-migrations.yaml
kubectl -n portfolio get jobs
POD=$(kubectl -n portfolio get pod -l job-name=backend-migrations -o jsonpath='{.items[0].metadata.name}')
kubectl -n portfolio logs "$POD"

Salida esperada:
✅ migrations + initial_data OK
```

## 7) Exposición pública (Traefik / Ingress)
Este cluster usa Traefik (incluido en k3s) como Ingress Controller.

```text
Reglas de ruteo:

/ → frontend (service:80)

/api → backend (service:8000)

/docs → manejado por backend (FastAPI)
```
Ver recursos de entrada:

```bash
kubectl -n portfolio get ingress
kubectl -n portfolio get ingressroute,middleware
```
Nota > Si se usa IngressRoute/Middleware (CRDs de Traefik), asegurarse de que existan los CRDs instalados:
```bash
kubectl get crd | grep -i traefik
```
## 8) Operación y troubleshooting
Rollouts
```bash
kubectl -n portfolio rollout status deploy/backend
kubectl -n portfolio rollout status deploy/frontend
```
Logs
```bash
kubectl -n portfolio logs deploy/backend --tail=200
kubectl -n portfolio logs deploy/frontend --tail=200
```
ImagePullBackOff / ErrImagePull

Checklist:

- [x] el tag existe en GHCR.

- [x] el owner en ghcr.io/<owner>/... está en minúsculas

- [x] el deployment apunta al tag correcto (no duplicado/concatenado)

- [x] si el package es privado: configurar imagePullSecre

