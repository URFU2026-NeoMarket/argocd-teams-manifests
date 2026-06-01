# Демо-манифесты для проверки ApplicationSet `team-services`

Содержимое этой папки — **точная копия структуры**, которую нужно положить в
ветку `master` репозитория
[`argocd-teams-manifests`](https://github.com/URFU2026-NeoMarket/argocd-teams-manifests).

Перенесите **внутренности** (`team-demo/`) в корень целевого репо — не саму папку
`demo-teams-manifests/`, а то, что в ней.

## Структура

```
team-demo/
├── svc-1/
│   ├── backend/   deployment.yaml   → Application team-demo-svc-1-backend
│   └── frontend/  deployment.yaml   → Application team-demo-svc-1-frontend
└── svc-2/
    └── admin/     deployment.yaml   → Application team-demo-svc-2-admin
```

## Что проверяем

| Свойство | Как видно на этом демо |
|---|---|
| Множественная генерация | 3 каталога 3-го уровня → 3 Application |
| Именование | `team-demo-svc-1-backend` и т.п. (нижний регистр) |
| Namespace `<team>-<service>` | `team-demo-svc-1` (backend+frontend вместе), `team-demo-svc-2` (admin) |
| Опциональность компонентов | `svc-2` имеет только `admin`, без backend/frontend — и это валидно |
| Авто-создание namespace | `CreateNamespace=true` в syncPolicy |

## Детали манифестов

- Образ — `traefik/whoami:v1.10.4` (лёгкая заглушка, отвечает на HTTP).
- **Namespace в манифестах намеренно не указан** — ArgoCD деплоит в `destination.namespace`
  из Application (`<team>-<service>`). Хардкодить namespace нельзя, иначе конфликт со схемой генерации.
- Заданы `resources` requests/limits — чтобы заглушки не съели ресурсы ноды.

## После наполнения репо

1. Применить ApplicationSet (`applicationset/team-services.yaml`) — задача 2.2 в `todo.md`.
2. Проверить: `kubectl get applications -n argocd` → 3 приложения, `Synced`/`Healthy`.
3. Прибраться: удалить `team-demo/` из репо → ArgoCD должен `prune` все 3 приложения.
