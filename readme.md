# Module 10 - ArgoCD & GitOps

Репозиторий содержит конфигурацию для управления инфраструктурой кластера через ArgoCD (App-of-Apps, ApplicationSet, AppProject, Sync Waves и ignoreDifferences).

## Структура проекта
- `bootstrap/root.yaml` — корневое приложение (App-of-Apps).
- `apps/` — манифесты приложений (`podinfo-dev`, `podinfo-prod`, `hello`).
- `appsets/` — генераторы ApplicationSet.
- `install/` — конфигурация для установки ArgoCD.
- `releases/` — файлы параметров (values) для окружений.

## Инструкция по развертыванию

1. Добавляем официальный Helm-репозиторий проекта Argo:
```bash
helm repo add argocd https://github.io
helm repo update
```

2. Устанавливаем ArgoCD одной командой, используя конфигурационный файл из этого репозитория:
```bash
helm install argocd argocd/argo-helm -n argo --create-namespace -f install/argocd-values.yaml
```

3. Запускаем GitOps-конвейер (App-of-Apps). Для этого примените корневой манифест из папки репозитория:
```bash
kubectl apply -f bootstrap/root.yaml
```


## Мониторинг (Module 11)

Стек разворачивается в namespace `monitoring` через ArgoCD:
- `apps/monitoring.yaml` — kube-prometheus-stack (Prometheus + Alertmanager + Grafana + node-exporter + kube-state-metrics)
- `apps/loki.yaml` — Loki
- `apps/promtail.yaml` — Promtail
- `apps/tempo.yaml` — Tempo (трейсы, бонус)

### Values

Лежат в `module-10/values/monitoring/`:
- `values.yaml` — kube-prometheus-stack
- `loki-values.yaml` — Loki
- `promtail-values.yaml` — Promtail
- `tempo-values.yaml` — Tempo
- `blackbox-values.yaml` — Blackbox Exporter

### Алерты в Telegram

Токен бота и chat_id — в Secret `alertmanager-telegram-secret`, монтируются в Alertmanager как файлы (`/etc/alertmanager/secrets/`).

Настроены алерты:
- `KubePodCrashLooping` — под крашится
- `PodinfoHighErrorRate` — много 5xx у Podinfo
- `NodeFilesystemAlmostOutOfSpace` — мало места на диске ноды
- `PodinfoDown` — Podinfo не отвечает

### Логи и трейсы Podinfo

- Логи: Grafana → Explore → Loki → `{namespace="podinfo-dev"}`
- Трейсы: Grafana → Explore → Tempo → Service Name = `podinfo-dev`

### ServiceMonitor и PrometheusRule для Podinfo

Лежат в репе `module-11` (`apps/podinfo-monitoring-app.yaml`, `manifests/podinfo-monitoring/`)
