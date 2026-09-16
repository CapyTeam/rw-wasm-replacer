# Changelog

## 1.0.1 (Remnawawe 3.4.3–3.4.4)

- Исправлена ошибка `version: unbound variable` при проверке совместимости с новой версией Remnawave.
- Переключатель Xray UI Editor перенесён из Visual settings в «Экспериментальные возможности» для нового интерфейса настроек.
- Патч страницы конфигурационного профиля адаптирован к текущей разметке Remnawave 3.4.3/3.4.4.
- Проверка loading gate WASM поддерживает текущий текст `The WASM module is loading...` и старый вариант.
- Для старых версий Remnawave сохранён fallback с переключателем в Visual settings.

## 1.0.0 (Remnawawe 3.2.1 и Xray UI Editor 1.2.0)

- Добавлено полноценное цветное интерактивное меню.
- Добавлено обнаружение Remnawave и Xray UI Editor по Docker Compose labels.
- Добавлен embedded Xray UI Editor вместо штатного WASM.
- Добавлен локальный переключатель в Visual settings.
- Добавлен stateless SSO без передачи пароля во frontend.
- Добавлена автоматическая CSP-настройка для iframe.
- Скрыты название профиля и кнопка «Профили» во встроенном режиме.
- Верхняя панель Editor адаптирована по ширине и выровнена по центру.
- Добавлена установка на одном или разных хостах.
- Добавлен image-only rollback без удаления БД и volumes.
- Добавлены точные backup-теги Docker image.
- Добавлено самообновление через GitHub Releases с показом release notes.
- Добавлена проверка совместимости новых версий Remnawave реальным dry-run.
- Добавлено чистое официальное обновление Remnawave без патча при несовместимости.
- Добавлен fallback на GitHub semver tags, если `releases/latest` отсутствует.
- Добавлена проверка SHA-256 при самообновлении через release asset `SHA256SUMS`.
- Добавлена публичная диагностика CSP и X-Frame-Options после установки.
- Добавлен полный clean-build откат legacy Editor при отсутствии state-файла.
