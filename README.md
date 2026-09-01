# Clash Mi — public profile + personal overwrite

В этом репозитории хранится публичный профиль **Clash Mi** `Default.yml`.

Рекомендуемая схема:

```text
Default.yml (public, GitHub)
          +
Personal.yml (personal overwrite, Secret Gist)
          ↓
      Clash Mi
```

Публичная часть содержит общую конфигурацию: DNS, правила, proxy groups и другие настройки. Персональная часть содержит приватные параметры, в первую очередь реальные proxy nodes.

## 1. Добавить персональный overwrite

Создайте `Personal.yml` и разместите его в **Secret Gist**. Используйте **Raw**-ссылку на файл Gist.

> Secret Gist является unlisted, а не полноценным private-хранилищем: любой, кто получит ссылку, сможет прочитать содержимое. Не публикуйте Raw URL в этом репозитории.

В Clash Mi откройте:

```text
Core Settings → Overwrite → +
```

Добавьте Raw URL вашего `Personal.yml`, задайте имя **Personal** и сохраните.

![Personal overwrite](docs/images/overwrite.jpg)

В списке Overwrite должен появиться `Personal`.

## 2. Подключить публичный профиль Default

Откройте:

```text
My Profiles → +
```

В поле **Clash Profile Link** укажите:

```text
https://raw.githubusercontent.com/rsivanov-git/clash-mi/refs/heads/main/Default.yml
```

Тип оставьте `yaml`.

![Add Profile Link](docs/images/add-profile.jpg)

Для профиля выберите **Core Overwrite → Current Selected (Personal)**. При необходимости задайте интервал обновления, например `1 d`.

После сохранения в **My Profiles** у профиля Default должно отображаться:

```text
Current Selected [Personal]
```

![Default profile with Personal overwrite](docs/images/profiles.jpg)

## 3. Как разделены настройки

### Default.yml

Публичный профиль использует только логическое имя proxy provider:

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

### Personal.yml

Персональный overwrite заменяет `Proxy-List` реальными proxy nodes:

```yaml
proxy-providers:
  Proxy-List:
    type: inline
    payload:
      - name: "London - 🦏 🇬🇧"
        type: hysteria2
        server: YOUR_SERVER
        port: 9443
        password: "YOUR_PASSWORD"
        sni: YOUR_SERVER
        down: 1024
        udp: true
        client-fingerprint: chrome

      - name: "London - 🐘 🇬🇧"
        type: anytls
        server: YOUR_SERVER
        port: 8443
        password: "YOUR_PASSWORD"
        sni: YOUR_SERVER
        udp: true
        client-fingerprint: chrome
```

В результате `Default.yml` не содержит IP-адресов, паролей и других персональных данных. При изменении общих правил достаточно обновить публичный `Default.yml`; личные proxy nodes продолжают храниться отдельно в `Personal.yml`.

## 4. Обновление

- `Default.yml` обновляется по Raw GitHub URL.
- `Personal.yml` обновляется отдельно по Raw URL Secret Gist.
- Clash Mi применяет выбранный Core Overwrite к активному профилю.
- Активным остаётся только один профиль — `Default.yml`; `Personal.yml` подключается именно как **Overwrite**, а не как второй профиль.
