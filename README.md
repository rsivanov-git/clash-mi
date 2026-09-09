# Clash Mi — public profile + personal overwrite

В этом репозитории хранится публичный профиль **Clash Mi** `Default.yaml`.

Рекомендуемая схема:

```text
Default.yaml (public, GitHub)
          +
Personal.yaml (Core Settings override, private GitHub repository)
          ↓
      Clash Mi
```

Публичная часть содержит общую конфигурацию: DNS, правила, proxy groups и другие настройки. Персональная часть содержит приватные параметры, в первую очередь реальные proxy nodes.

## Приложения Clash Mi

- **iOS / iPadOS:** [Clash Mi в App Store](https://apps.apple.com/us/app/clash-mi/id6744321968)
- **Android:** [Clash Mi — последняя версия на GitHub Releases](https://github.com/KaringX/clashmi/releases/latest)
- **macOS:** [Clash Mi — последняя версия на GitHub Releases](https://github.com/KaringX/clashmi/releases/latest)

Официальная страница загрузки для всех платформ: [clashmi.app/download](https://clashmi.app/download).

## 1. Добавить персональный overwrite

Создайте отдельный **закрытый (Private) репозиторий GitHub** и сохраните в нём `Personal.yaml` по примеру ниже. Замените адреса серверов, пароли, SNI и ключ Tailscale своими значениями. Реальный `Personal.yaml` с секретами должен храниться только в закрытом репозитории.

Скачайте `Personal.yaml` из GitHub, войдя в аккаунт с доступом к этому репозиторию. Затем импортируйте скачанный YAML-файл в раздел **Core Settings → Overwrite (Override)** приложения Clash Mi.

> Обычная Raw-ссылка на файл закрытого репозитория не предоставляет доступ без авторизации. Для загрузки и автоматического обновления по ссылке нужен отдельно настроенный способ авторизованного получения файла. Вариант с локальным импортом не требует передачи GitHub-токена приложению. Secret Gist не равнозначен закрытому репозиторию: его содержимое доступно любому, у кого есть ссылка.

В Clash Mi откройте:

```text
Core Settings → Overwrite → +
```

Импортируйте скачанный `Personal.yaml`, задайте имя **Personal**, сохраните и выберите его как текущий overwrite. **Add Profile Link** используйте только при наличии ссылки, по которой приложение действительно может получить YAML.

В списке Overwrite должен появиться `Personal`.

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

Для профиля выберите **Core Overwrite → Current Selected (Personal)**. При необходимости задайте интервал обновления, например `1 d`.

После сохранения в **My Profiles** у профиля Default должно отображаться:

```text
Current Selected [Personal]
```

## 3. Как разделены настройки

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

## 4. Обновление

- `Default.yaml` обновляется по Raw GitHub URL.
- `Personal.yaml` редактируется отдельно в закрытом GitHub-репозитории. При локальном импорте после изменения файла скачайте его заново и обновите overwrite **Personal** в Clash Mi.
- Автоматическое обновление персонального overwrite по URL возможно только при настроенном авторизованном доступе к файлу; обычной Raw-ссылки закрытого репозитория недостаточно.
- Clash Mi применяет выбранный Core Overwrite к активному профилю.
- Активным остаётся только один профиль — `Default.yaml`; `Personal.yaml` подключается именно как **Overwrite**, а не как второй профиль.
