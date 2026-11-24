---
title: "OpenCode"
description: "OpenCode - harness для работы с моделями"
---

# OpenCode

## OpenCode: установка и настройка

OpenCode — агент для программирования, работающий в терминале. Перед настройкой определи, какая у тебя версия: в OpenCode 1 и 2 разные поля конфигурации.

### 1. Установка

```bash
curl -fsSL https://opencode.ai/v2/install | bash
```

Другие способы для v2:

```bash
brew install opencode
npm install -g @opencode/cli
```

Скрипт v2 ставит один бинарник в `~/.opencode/bin` и не требует Node.js.

### 2. Подключение провайдера

Внутри OpenCode:

```text
/connect
```

Выбери провайдера и введи API-ключ. Вне интерфейса можно использовать:

```bash
opencode auth login
opencode auth list
```

Ключи также можно передавать переменными окружения, например `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`. Локально учётные данные хранятся в `~/.local/share/opencode/auth.json`.

## 3. Первый запуск

```bash
cd ~/my-project
opencode
```

Команда `/init` внутри интерфейса анализирует проект и создаёт файл `AGENTS.md` с инструкциями для агента.

Полезные команды: `/models` — выбор модели, `/undo` и `/redo` — откат и возврат последнего запроса вместе с изменениями файлов.

## 4. Файлы конфигурации

| Уровень | Путь |
|---|---|
| Проект | `opencode.json` или `opencode.jsonc` в корне проекта |
| Пользователь | `~/.config/opencode/opencode.json` |
| Своя | переменная `OPENCODE_CONFIG` |

После изменения конфигурации перезапусти OpenCode. Для диагностики используй `opencode debug config`.

## 5. Свой OpenAI-совместимый провайдер

```json
{
  "$schema": "[https://opencode.ai/config.json](https://opencode.ai/config.json)",
  "providers": {
    "myprovider": {
      "name": "My Provider",
      "env": ["MY_API_KEY"],
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "[https://api.example.com/v1](https://api.example.com/v1)"
      },
      "models": {
        "model-id": { "name": "My Model" }
      }
    }
  }
}
```

## 6. Проверка

1. Запусти `opencode models` — твои модели должны быть в списке.
2. Выбери модель через `/models`.
3. Отправь короткий тестовый запрос.

## 7. Частые проблемы

- Провайдер не виден: перезапусти OpenCode и проверь, что поля соответствуют версии (`provider`/`npm`/`options` для v1, `providers`/`package`/`settings` для v2).
- Модель есть в конфиге, но не работает: проверь, что ключ сохранён под тем же идентификатором провайдера.
- Не храни ключи в репозитории: используй переменные окружения.

## 8. Установка моей фирменной темы

```sh
mkdir -p ~/.config/opencode/themes
curl -fsSL \
  https://raw.githubusercontent.com/Suntechnic/green-papper/main/sub/opencode/green-chrome.json \
  -o ~/.config/opencode/themes/green-chrome.json
```

## 9. Полезное для первого старта

- Перевода на новую строку в консоле не будет работать с `Ctrl+Enter` - используй `Ctrl+j`
- Модели переключаются командой `/models`, но если уже начал печатать сообщение, то нажми `Ctrl+x` и затем, отпустив `k`
- Для переключения агента используй `Ctrl+x` и затем, отпустив `a`
- При выделении, OpenCode сам копирует выделенное в буфер обмена, но это не будет работать если он запущен на удаленном сервере - используйте для копирования из терминала выделение с зажатым `Shift` и копирование как из терминала - `Ctrl+Shift+C`