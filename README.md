# Clash Mi — public profile + personal overwrite

В этом репозитории хранится публичный профиль **Clash Mi** `Default.yaml`.

Рекомендуемая схема:

| Компонент | Источник |
| --- | --- |
| Основной профиль `Default.yaml` | Публичный GitHub-репозиторий |
| `Personal.yaml` при первом подключении | Локально импортированная копия |
| `Personal.yaml` после подключения к Tailscale | Единый HTTPS-адрес Tailscale Service на нескольких VPS с `private-git-proxy` |
| Оригинал `Personal.yaml` | Закрытый GitHub-репозиторий `clash-mi-personal` |

Публичная часть содержит общую конфигурацию: DNS, правила, proxy groups и другие настройки. Персональная часть содержит приватные параметры, в первую очередь реальные proxy nodes.

## Приложения Clash Mi

- **iOS / iPadOS:** [Clash Mi в App Store](https://apps.apple.com/us/app/clash-mi/id6744321968)
- **Android:** [Clash Mi — последняя версия на GitHub Releases](https://github.com/KaringX/clashmi/releases/latest)
- **macOS:** [Clash Mi — последняя версия на GitHub Releases](https://github.com/KaringX/clashmi/releases/latest)

Официальная страница загрузки для всех платформ: [clashmi.app/download](https://clashmi.app/download).

## 1. Первое подключение: локальный Personal.yaml

Создайте отдельный **закрытый (Private) репозиторий GitHub**. Рекомендуемое название — **`clash-mi-personal`**. Сохраните в его корне `Personal.yaml` по примеру ниже, заменив адреса серверов, пароли, SNI и ключ Tailscale своими значениями.

**Для первого подключения необходимо импортировать копию `Personal.yaml` локально.** В этом файле находятся настройки подключения к Tailscale, а целевой сетевой источник профиля доступен только внутри tailnet. Поэтому сначала нужно получить доступ к Tailscale с помощью локальной копии.

1. Скачайте `Personal.yaml` из закрытого репозитория, войдя в GitHub-аккаунт с доступом к нему.
2. Откройте в Clash Mi **Core Settings → Overwrite (Override) → +**.
3. Импортируйте скачанный YAML-файл, задайте имя **Personal Local**, сохраните и выберите его как текущий overwrite.
4. Подключите публичный профиль по следующему разделу и запустите Clash Mi, чтобы установить соединение с Tailscale.

Локальная копия нужна для первоначального подключения; после него необходимо переключить overwrite на целевой сетевой источник, описанный в разделе 3.

## 2. Подключить публичный профиль Default

Откройте:

```text
My Profiles → +
```

В поле **Clash Profile Link** укажите:

```text
https://raw.githubusercontent.com/rsivanov-git/clash-mi/refs/heads/main/Default.yaml
```

Тип оставьте `yaml`.

Для профиля выберите **Core Overwrite → Current Selected (Personal Local)**. При необходимости задайте интервал обновления, например `1 d`.

После сохранения в **My Profiles** у профиля Default должно отображаться:

```text
Current Selected [Personal Local]
```

## 3. Подключить целевой источник через Tailscale

Целевой источник — **Tailscale Service, опубликованный на нескольких VPS**, на которых работает [`private-git-proxy`](https://github.com/rsivanov-git/private-git-proxy). Каждый экземпляр получает один и тот же `Personal.yaml` из закрытого репозитория `clash-mi-personal`. В Clash Mi используется единый адрес сервиса Tailscale.

### Настройка источника на VPS

Разверните `private-git-proxy` на каждом VPS по [инструкции проекта](https://github.com/rsivanov-git/private-git-proxy#docker-compose). В `.env` каждого экземпляра укажите:

```dotenv
GIT_URL=https://raw.githubusercontent.com/OWNER/clash-mi-personal/refs/heads/main/Personal.yaml
GIT_TOKEN=YOUR_FINE_GRAINED_PERSONAL_ACCESS_TOKEN
```

Замените `OWNER` своим GitHub-логином. Для токена достаточно доступа к репозиторию `clash-mi-personal` с разрешением **Contents: Read-only**. Токен хранится на VPS; в Clash Mi его указывать не требуется.

Опубликуйте экземпляры как хосты **одного и того же Tailscale Service**. Например, для заранее созданного сервиса `proxy-list` на каждом VPS:

```sh
sudo tailscale serve --bg --service=svc:proxy-list --https=443 http://127.0.0.1:8080
```

Сервис, одобрение его хостов и правила доступа для устройств Clash Mi должны быть настроены в Tailscale. Используйте приватный доступ через Serve.

### Переключение overwrite с локального файла на сервис

После успешного подключения Clash Mi к Tailscale:

1. Откройте **Core Settings → Overwrite (Override) → + → Add Profile Link**.
2. Укажите корневой HTTPS-адрес вашего Tailscale Service, например `https://proxy-list.YOUR-TAILNET.ts.net/`, и сохраните overwrite под именем **Personal**.
3. Дождитесь успешной загрузки YAML, затем выберите сетевой **Personal** как текущий overwrite.
4. У профиля `Default.yaml` оставьте **Core Overwrite → Current Selected** и убедитесь, что отображается **Current Selected [Personal]**.
5. Настройте интервал обновления сетевого overwrite при необходимости.

Адрес выше — пример: замените его фактическим HTTPS-адресом сервиса. **Не добавляйте `/Personal.yaml`**: `private-git-proxy` возвращает файл по запросу к `/`. Ссылка на закрытый GitHub-репозиторий задаётся в `GIT_URL` на VPS, а ссылка на сервис Tailscale — в Clash Mi.

При загрузке overwrite соединение с Tailscale должно быть активно. Если на новом устройстве ещё нет настроек Tailscale, повторите первоначальный локальный импорт из раздела 1.

## 4. Как разделены настройки

### Default.yaml

Публичный профиль содержит два proxy provider с заглушками: `Proxy-List` для внешних прокси и `Tailscale-Provider` для Tailscale. Группы используют их по имени. Например:

```yaml
proxy-providers:
  Proxy-List:
    type: inline
    payload:
      - name: "Private placeholder"
        type: reject

proxy-groups:
  - name: Proxy
    type: select
    use:
      - Proxy-List
```

Placeholder нужен, чтобы публичный профиль оставался валидным даже без персонального overwrite.

### Personal.yaml

Персональный overwrite заменяет содержимое `Proxy-List` и `Tailscale-Provider`. Пример `Personal.yaml` для размещения в закрытом репозитории:

```yaml
proxy-providers:
  Proxy-List:
    type: inline
    payload:
      - name: "London - 🐁 🇬🇧"
        type: anytls
        server: 192.0.2.10
        port: 9443
        password: "YOUR_ANYTLS_PASSWORD_9443"
        sni: vpn.example.com
        udp: true

      - name: "London - 🐀 🇬🇧"
        type: anytls
        server: 192.0.2.10
        port: 8443
        password: "YOUR_ANYTLS_PASSWORD_8443"
        sni: vpn.example.com
        udp: true

  Tailscale-Provider:
    type: inline
    payload:
      - name: Tailscale
        type: tailscale
        auth-key: "YOUR_TAILSCALE_AUTH_KEY"
        hostname: clash-mi
        accept-routes: true
        udp: true
```

Все адреса и секреты в примере — заглушки; перед использованием замените их. `sni` должен соответствовать вашему серверному TLS-сертификату. `auth-key` — ключ подключения устройства к вашему tailnet. `accept-routes: true` включает принятие маршрутов, объявленных subnet routers в Tailscale.

Сохраните имена providers `Proxy-List` и `Tailscale-Provider`: на них ссылаются группы публичного профиля. DNS и правила маршрутизации остаются в `Default.yaml`; копировать их в персональный override не требуется.

В результате `Default.yaml` не содержит адресов ваших прокси-серверов, их паролей и ключа подключения к Tailscale. При изменении общих правил достаточно обновить публичный `Default.yaml`; личные proxy nodes продолжают храниться отдельно в `Personal.yaml`.

## 5. Обновление

- `Default.yaml` обновляется по публичному Raw GitHub URL.
- `Personal.yaml` редактируется в закрытом репозитории `clash-mi-personal`.
- После первоначального локального импорта персональный overwrite загружается и обновляется через HTTPS-адрес Tailscale Service. Каждый запрос к `private-git-proxy` получает файл из настроенного закрытого репозитория.
- Для обновления сетевого overwrite требуется доступ к Tailscale Service.
- Clash Mi применяет выбранный Core Overwrite к активному профилю.
- Активным остаётся профиль `Default.yaml`; `Personal.yaml` подключается как **Core Settings override**.
