# helm-store

This repo teaches you helm from scratch by building an hands on project. Happy Helming!!!H

## HelmStore 🛍️

cloud-native e-commerce platform built with helm.

## Services

- product-service (FastAPI + PostgreSQL)
- cart-service (FastAPI + Redis)
- frontend (Nginx)

## Deploy

``` bash
helm install product-service ./charts/product-service --values ./values/dev.yml
```
