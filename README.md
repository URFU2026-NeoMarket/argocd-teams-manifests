# Демо-манифесты для проверки ApplicationSet `team-services`

Содержимое этой папки — **точная копия структуры**, которую нужно положить в
ветку `master` репозитория
[`argocd-teams-manifests`](https://github.com/URFU2026-NeoMarket/argocd-teams-manifests).

Перенесите **внутренности** (`teamdemo/`) в корень целевого репо.

## Схема доставки: общий Helm-чарт + per-component values

Компоненты деплоятся **не сырыми манифестами, а общим чартом**
[`simple`](https://github.com/URFU2026-NeoMarket/helm-charts) (тот же, что у `infratest-example`).
Каталог компонента содержит **один `values.yaml`** — чарт сам рендерит
`Deployment` + `Service` + `Ingress` + `Certificate`. Публичный адрес включается
флагом `ingress.enabled: true`, TLS выпускает cert-manager.

```
teamdemo/                         commandID = namespace = teamdemo (бездефисный)
├── svc-1/
│   ├── backend/   values.yaml   → release svc-1-backend  → teamdemo-svc-1-backend.2026...
│   └── frontend/  values.yaml   → release svc-1-frontend → teamdemo-svc-1-frontend.2026...
└── svc-2/
    └── admin/     values.yaml   → release svc-2-admin    → teamdemo-svc-2-admin.2026...
```

## Контракт схемы именования (важно)

Чарт `simple` диктует жёсткие правила, и схема репозитория обязана им соответствовать:

| Что | Правило чарта | Как мы кладём |
|---|---|---|
| `commandID` | `^[a-z][a-z0-9]{2,31}$` — **без дефисов**, и **обязан == namespace** | каталог 1-го уровня (`teamdemo`) — бездефисный токен команды |
| namespace | = `commandID` | один на команду (`teamdemo`); все её компоненты живут вместе |
| имя релиза | дефисы разрешены; задаёт имена ресурсов | `<service>-<component>` (`svc-1-frontend`) |
| host | `<commandID>-<release>.2026.tochka-urfu.tech` | `teamdemo-svc-1-frontend.2026...` |

> ⚠️ Каталог 1-го уровня (= `commandID` = namespace) **не может содержать дефис** —
> схема values чарта это отвергает. Поэтому `team-demo` → `teamdemo`.
> Дефисы допустимы только в именах `service`/`component` (они уходят в имя релиза).

## Что в `values.yaml`

Только per-component настройки чарта: образ, порт, `ingress.enabled`, ресурсы.
**`commandID` в файле НЕ указывается** — он обязан совпадать с namespace и
инъектируется `ApplicationSet`'ом из пути (`segment[0]`) через `helm.parameters`.
Это убирает дублирование namespace в каждом файле и исключает дрейф.

Обязательные по `values.schema.json` поля: `image.repository`, `image.tag`,
`resources.limits.{cpu,memory}` (плюс `commandID`, который придёт из ApplicationSet).

## Что проверяем

| Свойство | Как видно на этом демо |
|---|---|
| Множественная генерация | 3 каталога 3-го уровня → 3 Application |
| Именование | `teamdemo-svc-1-backend` и т.п. (нижний регистр) |
| Один namespace на команду | все 3 компонента → namespace `teamdemo` |
| Опциональность компонентов | `svc-2` имеет только `admin` — и это валидно |
| Публичный адрес | `ingress.enabled: true` → host + TLS автоматически |

## После наполнения репо

1. Применить ApplicationSet (`applicationset/team-services.yaml` в проекте `k3s-server`).
2. Проверить: `kubectl get applications -n argocd` → 3 приложения, `Synced`/`Healthy`.
3. Открыть `https://teamdemo-svc-1-frontend.2026.tochka-urfu.tech` (whoami отвечает, TLS валиден).
4. Прибраться: удалить `teamdemo/` из репо → ArgoCD `prune` все 3 приложения.
