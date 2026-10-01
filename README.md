# Module 11 — Monitoring

Мониторинг в двух частях: на VM (systemd) и в Kubernetes (через ArgoCD).

## Структура
module-11/
├── part1-bare-metal/ # часть 1: Prometheus + Alertmanager + Node Exporter + Grafana на VM
├── apps/ # ArgoCD Application для мониторинга Podinfo
└── manifests/ # ServiceMonitor и PrometheusRule для Podinfo

text

## Часть 1 — bare-metal

Стек: Prometheus, Alertmanager, Node Exporter, Grafana. Каждый — отдельный systemd-сервис.

### Как поставить

1. Создать пользователей:
useradd --no-create-home --shell /bin/false prometheus
useradd --no-create-home --shell /bin/false alertmanager
useradd --no-create-home --shell /bin/false node_exporter

text

2. Скопировать конфиги:
cp part1-bare-metal/prometheus/prometheus.yml /etc/prometheus/
cp -r part1-bare-metal/prometheus/rules /etc/prometheus/
cp part1-bare-metal/alertmanadger/alertmanager.yml /etc/alertmanager/

text

3. Скопировать systemd-юниты:
cp part1-bare-metal/prometheus/prometheus.service /etc/systemd/system/
cp part1-bare-metal/alertmanadger/alertmanager.service /etc/systemd/system/
cp part1-bare-metal/node-exporter/node_exporter.service /etc/systemd/system/

text

4. Запустить:
systemctl daemon-reload
systemctl enable --now prometheus alertmanager node_exporter

text

Проверка: `http://<host>:9090` (Prometheus, вкладка Targets), `http://<host>:9093` (Alertmanager), `http://<host>:9100/metrics` (Node Exporter).

Grafana-дашборд: импортировать `part1-bare-metal/grafana/my-host.json`.

## Часть 2 — Kubernetes

Стек: kube-prometheus-stack (Prometheus + Alertmanager + Grafana + node-exporter + kube-state-metrics), Loki + Promtail (логи), Tempo (трейсы, бонус). Всё через ArgoCD.

### Как поставить
kubectl apply -f bootstrap/root.yaml

text

ArgoCD подхватит Application'ы из `apps/` и развернёт стек в namespace `monitoring`.

Проверка:
kubectl -n argo get applications
kubectl -n monitoring get pods

text

### Values

Лежат в GitOps-репе `engorohov/labs`, путь `module-10/values/monitoring/`:
- `values.yaml` — kube-prometheus-stack
- `loki-values.yaml` — Loki
- `promtail-values.yaml` — Promtail
- `tempo-values.yaml` — Tempo (бонус)
- `blackbox-values.yaml` — Blackbox Exporter (бонус)

### Алерты в Telegram

Настроены 4 алерта:
1. `KubePodCrashLooping` — под крашится
2. `PodinfoHighErrorRate` — много 5xx у Podinfo
3. `NodeFilesystemAlmostOutOfSpace` — мало места на диске
4. `PodinfoDown` — Podinfo не отвечает

Токен бота и chat_id лежат в Secret `alertmanager-telegram-secret`, монтируются в Alertmanager как файлы.

### Логи Podinfo

Grafana → Explore → Loki → `{namespace="podinfo-dev"}`.

### Трейсы Podinfo (бонус)

Grafana → Explore → Tempo → Service Name = `podinfo-dev`. Podinfo шлёт трейсы по OTLP на `tempo.monitoring.svc.cluster.local:4317`.

### ServiceMonitor и PrometheusRule для Podinfo

Лежат в `apps/podinfo-monitoring-app.yaml` и `manifests/podinfo-monitoring/`
