# Module 11 — Monitoring

Мониторинг в двух частях: на VM (systemd) и в Kubernetes (через ArgoCD).

## Часть 1 — bare-metal

Стек: Prometheus, Alertmanager, Node Exporter, Grafana. Каждый — отдельный systemd-сервис.

Установка:

1. Создать пользователей: `prometheus`, `alertmanager`, `node_exporter`.
2. Скопировать конфиги в `/etc/prometheus/` и `/etc/alertmanager/`.
3. Скопировать systemd-юниты в `/etc/systemd/system/`.
4. Запустить: `systemctl daemon-reload && systemctl enable --now prometheus alertmanager node_exporter`.

Проверка:
- Prometheus: `http://<host>:9090` (вкладка Targets — все UP)
- Alertmanager: `http://<host>:9093`
- Node Exporter: `http://<host>:9100/metrics`

Grafana: импортировать дашборд `part1-bare-metal/grafana/my-host.json`.

## Часть 2 — Kubernetes

Стек: kube-prometheus-stack (Prometheus + Alertmanager + Grafana + node-exporter + kube-state-metrics), Loki + Promtail (логи), Tempo (трейсы, бонус). Всё через ArgoCD.

Установка: `kubectl apply -f bootstrap/root.yaml`. ArgoCD подхватит Application'ы из `apps/` и развернёт стек в namespace `monitoring`.

Проверка:
- `kubectl -n argo get applications`
- `kubectl -n monitoring get pods`

Values лежат в GitOps-репе `engorohov/labs`, путь `module-10/values/monitoring/`:
- `values.yaml` — kube-prometheus-stack
- `loki-values.yaml` — Loki
- `promtail-values.yaml` — Promtail
- `tempo-values.yaml` — Tempo (бонус)
- `blackbox-values.yaml` — Blackbox Exporter (бонус)

## Алерты в Telegram

Настроены 4 алерта:
1. `KubePodCrashLooping` — под крашится
2. `PodinfoHighErrorRate` — много 5xx у Podinfo
3. `NodeFilesystemAlmostOutOfSpace` — мало места на диске
4. `PodinfoDown` — Podinfo не отвечает

Токен бота и chat_id лежат в Secret `alertmanager-telegram-secret`.

## Логи и трейсы Podinfo

- Логи: Grafana → Explore → Loki → `{namespace="podinfo-dev"}`.
- Трейсы: Grafana → Explore → Tempo → Service Name = `podinfo-dev`.

## Манифесты Podinfo

Лежат в `apps/podinfo-monitoring-app.yaml` и `manifests/podinfo-monitoring/`
