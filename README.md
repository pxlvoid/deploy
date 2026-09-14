# deploy

Общий GitHub Actions workflow выкатки docker compose проектов на свой сервер.

Сам деплой делает сервер: git fetch нужного коммита → `docker compose build` → `up --wait` →
откат на прежние образы, если сервис не стал healthy. Workflow только подключается по SSH
ключом проекта. На сервере этот ключ привязан к выкатке одного проекта через forced command,
шелла по нему нет. Адрес сервера и его ключ хоста лежат в секретах, в репозитории их нет.

## Подключение

```yaml
# .github/workflows/deploy.yml в репозитории проекта
name: Deploy
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
jobs:
  deploy:
    uses: pxlvoid/deploy/.github/workflows/deploy.yml@v1
    with:
      project: my-project
      url: https://example.com   # необязательно: проверка снаружи и ссылка в Environments
    secrets:
      DEPLOY_SSH_KEY: ${{ secrets.DEPLOY_SSH_KEY }}
      DEPLOY_SERVER: ${{ secrets.DEPLOY_SERVER }}
```

Секреты репозитория:

| Секрет | Что |
|---|---|
| `DEPLOY_SSH_KEY` | Приватный ключ деплоя проекта |
| `DEPLOY_SERVER` | Строка 1 — `user@host` или `user@host:port`, строка 2 — ключ хоста `ssh-ed25519 AAAA...` |

Секреты передаются явно: `secrets: inherit` между разными владельцами (репозиторий не у
pxlvoid) молча не передаёт ничего.

С не-default ветки workflow не выкатывает.

## Версии

Проекты ссылаются на `@v1`. Совместимые правки — перенести тег `v1`
(`git tag -f v1 && git push -f origin v1`), несовместимые — новый тег `v2`.
