# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Repositorio de práctica GitOps para desplegar Keycloak, Grafana y Prometheus en Kubernetes usando ArgoCD, Kustomize y Helm.

## Comandos principales

Este proyecto no tiene build/test tradicional — es IaC puro.

```bash
# Validar manifiestos con Kustomize (sin aplicar)
kubectl kustomize manifests/apps/__overlays

# Aplicar todos los manifiestos directamente
kubectl apply -k manifests/apps/__overlays

# Aplicar el ArgoCD Application raíz (GitOps sync)
kubectl apply -f revissions/primera-revision.yaml

# Ver el estado de los recursos en el namespace monitoring
kubectl get all -n monitoring
```

## Arquitectura

**Patrón base/overlay con Kustomize:**
- `manifests/apps/__overlays/` — Kustomization raíz que combina las tres apps
- `manifests/apps/<app>/base/` — HelmRelease base de cada aplicación
- `manifests/apps/<app>/lab/` — Overlay de entorno lab que referencia la base

**Valores de Helm:**
- `resources/apps/<app>/values.yaml` — Valores personalizados por aplicación

**ArgoCD Application:**
- `revissions/primera-revision.yaml` — App raíz que apunta a `manifests/apps/__overlays` con auto-sync, prune y self-heal

## Aplicaciones desplegadas

| App | Chart | Namespace | Helm values |
|-----|-------|-----------|------------|
| Keycloak | bitnami/keycloak (latest) | monitoring | resources/apps/keycloak/values.yaml |
| Grafana | grafana (latest) | monitoring | resources/apps/grafana/values.yaml |
| Prometheus | prometheus (latest) | monitoring | resources/apps/prometheus/values.yaml |

## Configuración de Keycloak

- Realm: `mi-realm`, Client: `mi-app` (público, wildcards)
- Roles: `app-admin`, `app-user`, `app-viewer`
- Scope personalizado: `mi-scope`
- Base de datos: PostgreSQL 17.6.0 (en clúster, credenciales en values.yaml)
- La configuración del realm se aplica automáticamente vía keycloak-config-cli al iniciar

## Notas importantes

- Las credenciales en los values files son para entorno de laboratorio únicamente
- `allowInsecureImages: false` — se usan imágenes `bitnami/` estándar
- Grafana tiene Prometheus configurado como datasource interno
- Prometheus tiene AlertManager y PushGateway deshabilitados
