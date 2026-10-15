# UzCloud CLI

[🇬🇧 English](README.md) | [🇷🇺 Русский](README.ru.md) | [🇺🇿 O'zbekcha](README.uz.md)

`uzc` это консольная утилита для [UzCloud](https://uzcloud.uz/). Управляет виртуальными машинами, дисками, сетями, кластерами Kubernetes, балансировщиками, DNS и биллингом. Работает через тот же API и тот же API-клиент, что и [Terraform-провайдер](https://registry.terraform.io/providers/telecom-cloud-uzcloud/uzcloud/latest).

## Установка

Скачайте архив для своей платформы со страницы [Releases](https://github.com/Telecom-Cloud-UzCloud/uzcloud-cli/releases) и положите `uzc` в каталог из `PATH`.

macOS и Linux:

```bash
VERSION=1.0.3
OS=darwin      # или linux
ARCH=arm64     # или amd64
curl -LO https://github.com/Telecom-Cloud-UzCloud/uzcloud-cli/releases/download/v$VERSION/uzcloud_cli_${VERSION}_${OS}_${ARCH}.tar.gz
tar -xzf uzcloud_cli_${VERSION}_${OS}_${ARCH}.tar.gz uzc
sudo mv uzc /usr/local/bin/
uzc version
```

Windows: скачайте `uzcloud_cli_<версия>_windows_amd64.zip`, распакуйте и добавьте папку с `uzc.exe` в `PATH`.

К каждому релизу также приложены пакеты для Debian/Ubuntu, RHEL/Fedora и Alpine (`.deb`, `.rpm`, `.apk`).

Проверьте загрузку по файлу `uzcloud_cli_<версия>_SHA256SUMS` из того же релиза:

```bash
shasum -a 256 -c uzcloud_cli_${VERSION}_SHA256SUMS --ignore-missing
```

На macOS бинарник пока не нотаризован. Если Gatekeeper его блокирует, выполните `xattr -d com.apple.quarantine /usr/local/bin/uzc`.

## Быстрый старт

Выпустите API-токен в портале UzCloud (**Профиль → API**), затем:

```bash
uzc init                       # сохранит токен и проект/регион/провайдер/биллинг-цикл по умолчанию
uzc region list
uzc plan list                  # тарифы ВМ в регионе по умолчанию
uzc template list
uzc ssh-key create laptop      # загрузит ~/.ssh/id_ed25519.pub
uzc vm create web-1 --template ubuntu-22-04 --plan box2cm2 --ssh-key laptop --wait
uzc vm list
```

## Настройки

Приоритет: **флаг → переменная окружения → профиль**.

| Параметр        | Флаг               | Переменная           |
|-----------------|--------------------|----------------------|
| API-токен       | `--token`          | `UZC_API_TOKEN`      |
| Адрес API       | `--endpoint`       | `UZC_API_ENDPOINT`   |
| Проект          | `--project`        | `UZC_PROJECT`        |
| Регион          | `--region`         | `UZC_REGION`         |
| Облачный провайдер | `--cloud-provider` | `UZC_CLOUD_PROVIDER` |
| Биллинг-цикл    | `--billing-cycle`  | `UZC_BILLING_CYCLE`  |
| Профиль         | `--profile`, `-p`  | `UZC_PROFILE`        |
| Файл конфига    | `--config`         | `UZC_CONFIG`         |

Имена переменных совпадают с Terraform-провайдером, поэтому оба инструмента могут работать от одного окружения. Профили хранятся в `<user config dir>/uzcloud/config.yaml` с правами `0600`.

```bash
uzc config set region tashkent
uzc config view
uzc init --profile staging
uzc config use staging
uzc -p prod vm list
```

## Формат вывода

Все команды поддерживают `-o table` (по умолчанию), `-o json` и `-o yaml`. Статусные сообщения пишутся в stderr, поэтому stdout можно отдавать в `jq`:

```bash
uzc vm list -o json | jq -r '.[] | select(.state=="Running") | .slug'
uzc vm list -q | xargs uzc vm stop   # -q: только слаги
```

## Команды

Полный список: `uzc --help`, подробности по каждой: `uzc <команда> --help`. Асинхронные операции принимают `--wait`. Разрушающие операции спрашивают подтверждение, если не передан `--yes`.

Для эндпоинтов без отдельной команды есть `uzc api`, с той же авторизацией, ретраями и обработкой ошибок:

```bash
uzc api GET /account/balance
uzc api POST /account/coupons -F coupon_code=FREE100
uzc api POST /autoscale -d @autoscale.json
```

Автодополнение: `uzc completion zsh|bash|fish|powershell`. Слаги ресурсов тоже дополняются: `uzc vm stop <TAB>`.
