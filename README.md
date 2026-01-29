# DevOps Portfolio — FastAPI Template (Docker + Kubernetes + CI/CD + AWS)

![CI](https://github.com/AlfreMu/devops-portfolio-fastapi/actions/workflows/ci.yml/badge.svg)

Proyecto de **portfolio DevOps** que demuestra el despliegue completo de una aplicación
containerizada utilizando **Docker**, **Kubernetes (k3s)**, **CI/CD con GitHub Actions**
y **despliegue real en AWS**.

El foco del proyecto no está en el desarrollo de la aplicación, sino en la **infraestructura,
automatización, despliegue y buenas prácticas DevOps**.

---

## Descripción del Proyecto

Este repositorio implementa una aplicación backend en **FastAPI** con frontend web,
orquestada en **Kubernetes**, con base de datos **PostgreSQL** persistente.

El objetivo es demostrar, a nivel **DevOps Jr**, cómo llevar una aplicación desde
contenedores locales hasta un **cluster Kubernetes en la nube**, incluyendo:

- Contenerización correcta
- CI funcional
- Publicación de imágenes
- Deploy real en AWS
- Exposición pública mediante Ingress
- Persistencia de datos
- Decisiones técnicas explicables en entrevistas

---

## 🏗️ Arquitectura General

La arquitectura del proyecto es la siguiente:

- **AWS EC2** como host de infraestructura
- **k3s** como distribución ligera de Kubernetes
- **Traefik** como Ingress Controller
- **Backend** FastAPI desplegado como Deployment
- **Frontend** web desplegado como Deployment
- **PostgreSQL** con volumen persistente (PVC)
- **Job de Kubernetes** para ejecutar migraciones y datos iniciales
- **GitHub Container Registry (GHCR)** como registry de imágenes

Todo el tráfico externo ingresa por Traefik y se enruta de la siguiente forma:

- `/` → Frontend
- `/api` → Backend
- `/docs` → Swagger (FastAPI)

---

## 🧰 Stack Tecnológico

- **Lenguaje / Framework**
  - Python (FastAPI)
  - Frontend web (imagen preexistente del template)

- **Contenedores**
  - Docker
  - Docker Compose (entorno local)

- **Orquestación**
  - Kubernetes (k3s)
  - Deployments, Services, Jobs, Ingress

- **CI/CD**
  - GitHub Actions
  - Build de imágenes
  - Publicación en GHCR
  - Branch protection y checks

- **Cloud**
  - AWS EC2 (Ubuntu 22.04)

- **Networking**
  - Traefik Ingress Controller

---

## ☁️ Despliegue en AWS

El proyecto está desplegado en una instancia **EC2** que ejecuta un cluster
Kubernetes local mediante **k3s**.

Características del despliegue:

- Cluster Kubernetes funcional en AWS
- Imágenes descargadas desde GHCR
- Servicios internos expuestos vía Ingress
- Datos persistentes incluso tras reinicios de la instancia

---

## 🌐 Accesos Públicos

Con la instancia en ejecución, la aplicación queda accesible vía la IP pública de EC2:

- **Frontend:**  
  `http://<IP_PUBLICA>/`

- **Backend (API):**  
  `http://<IP_PUBLICA>/api`

- **Swagger UI:**  
  `http://<IP_PUBLICA>/docs`

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
.
├── k8s/ # Manifiestos de Kubernetes
│ └── portfolio/
│ ├── backend/
│ ├── frontend/
│ ├── postgres/
│ └── traefik/
├── docs/ # Documentación técnica
├── .github/workflows/ # Pipelines CI/CD
├── docker-compose.* # Entorno local
└── README.md

## 🚧 Estado del Proyecto y Próximos Pasos

Estado actual:
- ✅ Backend y frontend funcionando en Kubernetes
- ✅ Persistencia de datos validada
- ✅ CI y publicación de imágenes
- ✅ Exposición pública mediante Ingress

Próximas mejoras:
- Infraestructura como Código (Terraform).
- HTTPS con certificados TLS.
- CD, Automatización completa del deploy.


📌 Autor: AlfreMu
