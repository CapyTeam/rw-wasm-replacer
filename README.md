# Remnawawe WASM Replacer by CapyTeam

`rw-wasm` — интерактивный CLI-инструмент для замены штатного WASM-редактора конфигурационных профилей Remnawave на Remnawave Xray UI Editor.

Скрипт поддерживает установку компонентов на одном или разных хостах, безопасный откат Docker image, проверку совместимости будущих версий Remnawave и обновление самого скрипта через GitHub Releases.

## Что делает патч

После установки в Remnawave появляется локальный переключатель:

```text
Settings → Visual settings → Внешний редактор конфигурационных профилей
```

При включённом переключателе на компьютере:

- Xray UI Editor отображается прямо внутри страницы профиля Remnawave;
- адресная строка браузера остаётся на домене панели;
- открывается профиль с тем же UUID;
- штатный WASM вообще не инициализируется;
- выполняется автоматическая SSO-авторизация;

На мобильных устройствах остаётся штатный WASM.

## Архитектура SSO
Пароль Xray UI Editor хранится только в `.env` backend Remnawave.

Последовательность:

```text
Remnawave frontend
→ авторизованный запрос в backend Remnawave
→ backend получает короткоживущий подписанный токен у Editor
→ iframe открывает SSO URL
→ Editor устанавливает HttpOnly cookie
→ Editor открывает /profiles/<UUID>
```

Пароль не компилируется в frontend и не передаётся браузеру.

Editor разрешает встраивание только с указанного HTTPS-origin панели через CSP `frame-ancestors`. Патч Remnawave добавляет домен Editor в `frame-src`.

## Безопасность базы данных

Скрипт не использует:

```bash
docker compose down -v
docker volume rm
docker system prune --volumes
```

Он не удаляет PostgreSQL, Redis, Docker volumes или пользовательские настройки.

При установке, обновлении и откате меняется только Docker image нужного сервиса, после чего пересоздаётся соответствующий контейнер:

```bash
docker compose up -d --force-recreate <service>
```

Миграции самой Remnawave при обновлении выполняются штатным приложением.

Несмотря на то, что скрипт не затрагивает volumes, перед обновлением production-панели рекомендуется иметь актуальный бэкап БД.

## Требования

- Linux.
- Root-доступ.
- Docker Engine.
- Docker Compose plugin.
- Remnawave и/или Xray UI Editor, установленные через Docker Compose.
- Публичный HTTPS для панели и Editor.
- Доступ к:
  - GitHub API и codeload;
  - Docker Hub/GHCR;
  - `validator.remna.dev`;
  - npm registry во время сборки.

Поддерживаются установки на одном и на разных хостах.


## Установка команды `rw-wasm`

Скопируйте файл на сервер:

```bash
curl -fLO https://github.com/CapyTeam/rw-wasm-replacer/releases/download/1.0.0/rw-wasm
chmod +x rw-wasm
sudo ./rw-wasm --install
```
После этого меню можно открыть из любого каталога:

```bash
sudo rw-wasm
```

Дополнительные команды:

```bash
rw-wasm --version
rw-wasm --help
sudo rw-wasm --offline
```

`--offline` отключает проверку обновлений GitHub, но сохраняет локальную установку и откат.

## Главное меню

Вверху отображаются:

- название `Remnawawe WASM Replacer by CapyTeam`;
- версия скрипта;
- версия установленной Remnawave;
- версия установленного Xray UI Editor;
- статус патча каждого компонента;
- версия установленного патча;
- уведомления об обновлениях.

Пример:

```text
Версия скрипта:                 1.0.0
Версия Remnawawe:               3.2.1
Версия Xray UI Editor:          1.2.0
Статус патча:                   Remnawave: установлен; Editor: установлен
Версия установленного патча:    Remnawave 1.0.0, Editor 1.0.0
```

### Блок установки

```text
1. Пропатчить Xray UI Editor
2. Пропатчить Remnawawe
3. Пропатчить Remnawawe и Xray UI Editor на одном хосте
```

Пункт доступен только тогда, когда соответствующий контейнер найден и патч ещё не установлен.

### Разные хосты

На сервере Editor:

```bash
sudo rw-wasm
```

Выберите пункт 1.

Скрипт запросит:

- HTTPS-origin панели;
- `APP_PASSWORD` Editor.

Затем на сервере Remnawave выберите пункт 2.

Скрипт запросит:

- HTTPS URL Editor;
- `APP_PASSWORD` Editor;
- публичный HTTPS URL панели для проверки.

### Один хост

Если найдены оба контейнера, выберите пункт 3.

URL и пароль будут запрошены один раз, после чего сначала патчится Editor, затем Remnawave.

## Откат

Пункт:

```text
4. Откатить патч
```

Если пропатчены оба компонента, сначала выбирается:

```text
1. Remnawave
2. Xray UI Editor
3. Оба компонента
```

Затем тип отката:

### Полный откат

Для Remnawave устанавливается официальный image той же текущей версии.

Пример:

```text
remnawave/backend:3.2.1
```

Для Editor восстанавливается исходный image, который использовался до патча. Если обнаружен старый legacy-патч без state-файла, чистый Editor собирается из официального тега той же версии.

### Откат до бэкапа

Перед патчем скрипт присваивает текущему image отдельный локальный backup-тег:

```text
local/capyteam/rw-wasm-backup/remnawave:<timestamp>
local/capyteam/rw-wasm-backup/editor:<timestamp>
```

Откат до бэкапа возвращает точную копию image, работавшего непосредственно перед патчем. Для legacy-патчей, установленных до появления `rw-wasm` 1.0.0, точного backup-тега может не быть; в таком случае доступен полный откат, но не восстановление неизвестного прежнего image.

При обоих типах отката:

- `.env` не удаляется;
- Compose-файл не заменяется целиком;
- меняется только строка `image:` нужного сервиса;
- volumes не удаляются;
- БД не сбрасывается.

## Проверка обновлений Remnawave

Скрипт получает последний стабильный GitHub Release `remnawave/backend`. Если `releases/latest` недоступен, используется последний строгий semver-тег `X.Y.Z`. Затем загружаются соответствующие теги:

```text
remnawave/backend
remnawave/frontend
```

После этого выполняется реальный dry-run патчера по исходникам.

Проверяются именно изменяемые файлы и контрольные блоки:

- страница конфигурационного профиля;
- connector инициализации WASM;
- Visual settings;
- backend modules;
- JWT guard dependencies;
- Dockerfile stages;
- Helmet CSP;
- остальные участки, которые модифицирует патч.

### Совместимая новая версия

Если dry-run успешен:

```text
ДОСТУПНО ОБНОВЛЕНИЕ С ПАТЧЕМ! [vX.Y.Z]
```

Меню предлагает:

```text
1. Установить обновление с патчем
2. Установить чистую официальную версию
q. Отмена
```

### Несовместимая новая версия

Если структура изменяемого кода изменилась:

```text
ДОСТУПНО ОБНОВЛЕНИЕ, ТЕКУЩИЙ ПАТЧ НЕ ПОДДЕРЖИВАЕТСЯ! [vX.Y.Z]
```

Скрипт предупреждает, что может установить только чистый официальный image.

После подтверждения выполняется безопасный эквивалент:

```bash
docker compose pull <service>
docker compose up -d --force-recreate <service>
```

`docker compose down -v` не используется.

## Автоматическое определение компонентов

По умолчанию скрипт ищет контейнеры:

```text
remnawave
remnawave-xray-ui-editor
```

Compose working directory, имя сервиса и путь к Compose-файлу определяются по Docker labels:

```text
com.docker.compose.project.working_dir
com.docker.compose.project.config_files
com.docker.compose.service
```

Если labels недоступны, используются стандартные каталоги:

```text
/opt/remnawave
/opt/remnawave-xray-ui-editor
```

## Хранилище состояния

```text
/usr/local/bin/rw-wasm
/var/lib/rw-wasm-replacer/
/var/lib/rw-wasm-replacer/backups/
/var/cache/rw-wasm-replacer/
/opt/rw-wasm-replacer/build/
/var/log/rw-wasm-replacer.log
```

Назначение:

- `/var/lib/rw-wasm-replacer/*.env` — состояние установленного патча;
- `/var/lib/rw-wasm-replacer/backups/` — архив предыдущих state-файлов;
- `/var/cache/rw-wasm-replacer/` — ответы GitHub API и dry-run;
- `/opt/rw-wasm-replacer/build/` — исходники и результаты сборки;
- `/var/log/rw-wasm-replacer.log` — журнал операций.

## Диагностика

Версия:

```bash
rw-wasm --version
```

Контейнеры:

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
```

Логи:

```bash
docker logs --tail=200 remnawave
docker logs --tail=200 remnawave-xray-ui-editor
tail -n 200 /var/log/rw-wasm-replacer.log
```

Проверка CSP Editor:

```bash
curl -skD- -o /dev/null https://editor.example.com/ \
  | grep -iE '^(content-security-policy|x-frame-options):'
```

Проверка CSP панели:

```bash
curl -skD- -o /dev/null https://panel.example.com/ \
  | grep -iE '^(content-security-policy|x-frame-options):'
```

Ожидается, что:

- Editor разрешает `frame-ancestors` для origin панели;
- Remnawave содержит `frame-src` с origin Editor;
- публичный reverse proxy не заменяет эти заголовки более строгими значениями.

После установки Remnawave скрипт автоматически проверяет публичные CSP/X-Frame-Options и выводит предупреждение, если reverse proxy переопределяет заголовки приложения.

## Ограничения

- Успешный dry-run подтверждает структурную совместимость изменяемых участков, но не заменяет полноценное production-тестирование новой версии.
- При радикальной смене архитектуры Remnawave или Editor потребуется новая версия патчера.
- Некоторые браузерные или корпоративные политики могут полностью запрещать сторонние cookies. Наиболее надёжная схема — панель и Editor на поддоменах одного базового домена.
- Автоматическое обновление Xray UI Editor в меню 1.0.0 не реализовано; перед его патчем совместимость выбранной версии всё равно проверяется.

Made by Sakred_ and CapyTeam