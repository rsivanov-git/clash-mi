# Clash Mi — public profile + personal overwrite

В этом репозитории хранится публичный профиль **Clash Mi** `Default.yaml`.

Рекомендуемый способ подключения персональных настроек — **Secret Gist** с файлом `Personal.yaml`.

| Компонент | Источник |
| --- | --- |
| Основной профиль `Default.yaml` | Публичный GitHub-репозиторий |
| Персональный overwrite `Personal.yaml` | Raw-ссылка на Secret Gist |
| Дополнительный вариант с шифрованием | HTTPS-прокси в Yandex Cloud, читающий файл из закрытого GitHub-репозитория по токену |

Публичная часть содержит общую конфигурацию: DNS, правила и proxy groups. Персональная часть содержит реальные proxy nodes с адресами серверов и паролями.

## Приложения Clash Mi

- **iOS / iPadOS:** [Clash Mi в App Store](https://apps.apple.com/us/app/clash-mi/id6744321968)
- **Android:** [Clash Mi — последняя версия на GitHub Releases](https://github.com/KaringX/clashmi/releases/latest)
- **macOS:** [Clash Mi — последняя версия на GitHub Releases](https://github.com/KaringX/clashmi/releases/latest)

Официальная страница загрузки для всех платформ: [clashmi.app/download](https://clashmi.app/download).

## 1. Подключить Personal.yaml через Secret Gist

1. Откройте [GitHub Gist](https://gist.github.com/), войдите в свой аккаунт и создайте файл **Personal.yaml** по шаблону ниже. Замените адреса серверов, пароли и SNI своими значениями.
2. Выберите **Create secret gist**.
3. Нажмите **Raw** у файла и скопируйте ссылку. Для обновления до последней версии используйте URL без идентификатора конкретной ревизии:

   ```text
   https://gist.githubusercontent.com/OWNER/GIST_ID/raw/Personal.yaml
   ```

   Замените `OWNER` и `GIST_ID` значениями вашего Gist. Если после `/raw/` в скопированной ссылке стоит хеш ревизии, удалите его, оставив имя файла.

4. В Clash Mi откройте **Core Settings → Overwrite (Override) → + → Add Profile Link**.
5. Вставьте Raw-ссылку, сохраните overwrite под именем **Personal** и выберите его как текущий.
6. При необходимости настройте интервал обновления.

**Secret Gist доступен любому, у кого есть ссылка**: он не отображается в публичном поиске, но не является закрытым хранилищем и не шифрует содержимое. Не публикуйте Raw-ссылку с личными настройками. Подробнее — [документация GitHub](https://docs.github.com/en/get-started/writing-on-github/editing-and-sharing-content-with-gists/creating-gists).

## 2. Подключить публичный профиль Default

Откройте **My Profiles → +**. В поле **Clash Profile Link** укажите:

```text
https://raw.githubusercontent.com/rsivanov-git/clash-mi/refs/heads/main/Default.yaml
```

Тип оставьте `yaml`. Для профиля выберите **Core Overwrite → Current Selected (Personal)**. При необходимости задайте интервал обновления, например `1 d`.

После сохранения в **My Profiles** у профиля Default должно отображаться:

```text
Current Selected [Personal]
```

## 3. Дополнительно: шифрующий прокси в Yandex Cloud

Для дополнительного шифрования можно разместить HTTPS-прокси в **Yandex Cloud**, а оригинал `Personal.yaml` хранить в закрытом GitHub-репозитории, например `clash-mi-personal`. Это альтернативный источник overwrite; Secret Gist в этой схеме не требуется.

Схема загрузки:

```text
Clash Mi → HTTPS-прокси в Yandex Cloud → закрытый GitHub-репозиторий
          зашифрованный ответ ← Personal.yaml по токену
```

Прокси должен:

1. Читать `Personal.yaml` через [GitHub Contents API](https://docs.github.com/en/rest/repos/contents#get-repository-content), передавая fine-grained personal access token в `Authorization: Bearer …`. Ограничьте токен исходным репозиторием и разрешением **Contents: Read-only**. Токен хранится на стороне прокси и не передаётся в Clash Mi или в URL подписки.
2. Шифровать содержимое в формате, который поддерживает Clash Mi: **AES-128-CBC с PKCS7**, ключ — 16 байт `MD5(UTF-8(password))`, свежий случайный IV — 16 байт для каждого ответа. Тело ответа — `Base64(IV + ciphertext)`.
3. Возвращать зашифрованное тело с HTTP-заголовком:

   ```http
   subscription-encryption: true
   ```

4. Обслуживать запросы по HTTPS; пароль шифрования хранить на стороне прокси и отдельно настроить в Clash Mi.

Одного заголовка недостаточно: тело ответа должно быть зашифровано. Формат и проверка заголовка описаны в исходном коде Clash Mi: [расшифровка содержимого](https://github.com/KaringX/clashmi/blob/main/lib/app/utils/profile_decrypt_utils.dart) и [загрузка профиля](https://github.com/KaringX/clashmi/blob/main/lib/app/modules/profile_manager.dart).

В настройках сетевого overwrite **Personal** укажите HTTPS-адрес прокси вместо Raw-ссылки Gist и задайте пароль расшифровки. Дождитесь успешной загрузки; у `Default.yaml` оставьте **Core Overwrite → Current Selected [Personal]**.

Это схема дополнительного развёртывания: реализация прокси и конфигурация Yandex Cloud в данный репозиторий не входят.

## 4. Как разделены настройки

### Default.yaml

Публичный профиль содержит provider `Proxy-List` с заглушкой. Группа `Proxy` использует его по имени:

```yaml
proxy-providers:
  Proxy-List:
    type: inline
    payload:
      - name: "Private placeholder"
        type: direct
        client-fingerprint: firefox

proxy-groups:
  - name: Proxy
    type: select
    use:
      - Proxy-List
```

Заглушка позволяет загрузить публичный профиль без персонального overwrite; реальные прокси появятся после подключения `Personal.yaml`.

### Personal.yaml

Персональный overwrite заменяет содержимое `Proxy-List`. Шаблон для Secret Gist или закрытого репозитория при использовании шифрующего прокси:

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

```

Все адреса и секреты в примере — заглушки; перед использованием замените их. `sni` должен соответствовать вашему серверному TLS-сертификату.

Сохраните имя provider `Proxy-List`: на него ссылается группа публичного профиля. DNS и правила маршрутизации остаются в `Default.yaml`; копировать их в персональный overwrite не требуется.

В результате `Default.yaml` не содержит адресов ваших прокси-серверов и их паролей. При изменении общих правил достаточно обновить публичный `Default.yaml`; личные proxy nodes хранятся отдельно в `Personal.yaml`.

## 5. Обновление

- `Default.yaml` обновляется по публичному Raw GitHub URL.
- В рекомендуемой схеме редактируйте `Personal.yaml` в Secret Gist. Raw-ссылка без хеша ревизии позволяет получать последнюю версию файла.
- При использовании прокси редактируйте файл в закрытом репозитории; прокси получает его по токену и возвращает зашифрованный ответ.
- Обновляйте сетевой overwrite **Personal** вручную или по настроенному интервалу.
- Активным остаётся профиль `Default.yaml`; Clash Mi применяет к нему выбранный **Core Settings overwrite**.
