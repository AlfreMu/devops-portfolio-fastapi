![CI](https://github.com/AlfreMu/devops-portfolio-fastapi/actions/workflows/ci.yml/badge.svg) -
![CD](https://github.com/AlfreMu/devops-portfolio-fastapi/actions/workflows/cd.yml/badge.svg)
---
# DevOps Portfolio — FastAPI App (Docker + Kubernetes + CI/CD + AWS)

Proyecto de **portfolio DevOps** que demuestra el despliegue completo de una aplicación
containerizada utilizando **Docker**, **Kubernetes (k3s)**, **CI/CD con GitHub Actions**
y **despliegue real en AWS (EC2)**.

El foco del proyecto no está en el desarrollo de la aplicación, sino en la **infraestructura,
automatización, despliegue y prácticas DevOps**.

---
## Descripción del Proyecto

Este repositorio implementa una aplicación backend en **FastAPI** con frontend web,
orquestada en **Kubernetes**, con base de datos **PostgreSQL** persistente.

El objetivo es demostrar, a nivel **DevOps Jr**, cómo llevar una aplicación desde
contenedores locales hasta un **cluster Kubernetes en la nube**, incluyendo:

- Contenerización con Docker
- CI funcional
- Publicación de imágenes en registry
- Deploy real en AWS (EC2)
- Exposición pública mediante Ingress (Traefik)
- Persistencia de datos (PostgreSQL)

---
## 📚 Documentación

- Kubernetes (k3s en AWS EC2): [docs/kubernetes.md](docs/kubernetes.md)
- CI/CD (GitHub Actions + GHCR + deploy a k3s): [docs/ci-cd.md](docs/ci-cd.md)
---
## 🏗️ Arquitectura General

La arquitectura del proyecto es la siguiente:

- **AWS EC2** como host de infraestructura
- **k3s** como distribución ligera de Kubernetes
- **Traefik** como Ingress Controller
- **Backend** FastAPI desplegado como Deployment
- **Frontend** web desplegado como Deployment
- **PostgreSQL** con volumen persistente (PVC)
- **Job de Kubernetes** para migraciones y datos iniciales
- **GitHub Container Registry (GHCR)** como registry de imágenes Docker

Todo el tráfico externo ingresa por Traefik y se enruta de la siguiente forma:

- `/` → Frontend web (login)
- `/docs` → Swagger UI (FastAPI)
- `/api/v1/utils/health-check` → Backend (health-check)

---

## 🧰 Stack Tecnológico

- **Backend:** Python · FastAPI  
- **Frontend:** Web (template base)
- **Contenedores:** Docker · Docker Compose (local)
- **Orquestación:** Kubernetes (k3s)
- **CI/CD:** GitHub Actions · GHCR
- **Infraestructura:** AWS EC2 (Ubuntu 22.04)
- **Networking:** Traefik (Ingress Controller)

---

## ☁️ Despliegue en AWS

El proyecto está desplegado en una instancia **EC2** que ejecuta un cluster
Kubernetes local mediante **k3s**.

Características del despliegue:

- Cluster Kubernetes (k3s) funcional en AWS
- Imágenes Docker descargadas desde GHCR
- Servicios internos expuestos vía Ingress (Traefik)
- Datos persistentes incluso tras reinicios de la instancia

---

## 🌐 Accesos Públicos
Con la instancia en ejecución, la aplicación queda accesible vía la IP pública de EC2:

- **Frontend:**
  - `http://54.227.12.89/` (redirige al login)

- **Swagger UI (Backend):**
  - `http://54.227.12.89/docs`

- **Health-check (Backend):**
  - `http://54.227.12.89/api/v1/utils/health-check`
---

## 🎯 Objetivos del proyecto: 
Demostrar de forma practica:

- Uso correcto de **Docker** y contenedores
- Orquestación real con **Kubernetes**
- Separación de responsabilidades entre servicios
- Ejecución de **migraciones mediante Jobs**
- Uso de **tags inmutables** para imágenes
- CI funcional con **GitHub Actions**
- Despliegue real en **AWS**
- Diagnóstico y resolución de problemas reales (ImagePull, migrations, Ingress)

---

## 📁 Estructura del Repositorio
```text
├── k8s/                      # Manifiestos de Kubernetes
│   └── portfolio/
│       ├── backend/
│       ├── frontend/
│       ├── postgres/
│       └── traefik/
├── docs/                     # Documentación técnica
├── .github/workflows/        # Pipelines CI/CD
├── docker-compose.*          # Entorno local
└── README.md
```

📌 Autor: AlfreMu
