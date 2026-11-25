---
title: "FreeLLMAPI - короткий гайд"
description: "Объединяем несколько подписок с бесплатными квотами в одну"
---

# FreeLLMAPI: короткий гайд

## 1. Установка и первый запуск

Предварительно установите Git, Node.js и npm совместимых с текущим package.json версий.

```bash
cd ~
git clone https://github.com/tashfeenahmed/freellmapi.git
cd ~/freellmapi
npm install
npm run dev
```

Если репозиторий уже существует, повторно клонировать его не нужно.
По текущему README dev-режим запускает API на порту 3001, панель — на 5173. Ориентируйтесь на адреса в выводе своей версии. 

## 2. Настройка и подключение клиентов

- При первом запуске создайте учётную запись панели: email и пароль. Общего стандартного пароля нет.
- В панели добавьте свои ключи провайдеров и включите нужные модели; для агентных клиентов выбирайте модели с поддержкой инструментов.
- На странице Keys скопируйте Unified API Key вида `freellmapi-…`.
- В клиенте выберите OpenAI-compatible API, Base URL `http://127.0.0.1:3001/v1`, API Key — единый ключ FreeLLMAPI, модель — `auto` либо доступный идентификатор из роутера.

Не путайте три секрета: пароль панели, единый ключ для клиентов и ключи внешних провайдеров. Лицензия Premium — отдельная сущность.

`127.0.0.1` относится к машине, на которой работает клиент. Для Copilot в Remote SSH это удалённый сервер, а не ноутбук. Нужен отдельно настроенный туннель или иной защищённый доступ; сервис на ноутбуке сам его не создаёт. Не публикуйте порты без продуманной защиты.

## 3. Демонизация через systemd

Остановите ручной процесс через Ctrl+C. Следующий блок запускайте в Bash на машине с установленным FreeLLMAPI. Он создаёт/перезаписывает unit и фиксирует текущие пути npm и Node (включая установку через nvm).

```bash
cd ~/freellmapi || exit 1
mkdir -p ~/.config/systemd/user
NPM_BIN="$(command -v npm)"
NODE_BIN="$(command -v node)"

if [ -x "$NPM_BIN" ] && [ -x "$NODE_BIN" ]; then
  cat > ~/.config/systemd/user/freellmapi.service <<EOF
[Unit]
Description=FreeLLMAPI local proxy

[Service]
Type=simple
WorkingDirectory=$PWD
Environment="PATH=$(dirname "$NODE_BIN"):$(dirname "$NPM_BIN"):/usr/local/bin:/usr/bin:/bin"
ExecStart="$NPM_BIN" run dev
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF
  systemctl --user daemon-reload
  systemctl --user enable --now freellmapi.service
  systemctl --user status freellmapi.service --no-pager
else
  echo "Не найден npm или node; сервис не создан."
fi
```

Для запуска при загрузке без входа в учётную запись:

```bash
sudo loginctl enable-linger "$USER"
```

Сервис не получает автоматически переменные из интерактивной оболочки. Если ручной запуск зависит от экспортированных переменных, настройте их отдельно; не публикуйте секреты в unit. Загрузка приложением своего `.env` зависит от реализации установленной версии.

```bash
# Статус и логи
systemctl --user status freellmapi.service --no-pager
journalctl --user -u freellmapi.service -n 80 --no-pager
journalctl --user -u freellmapi.service -f

# Управление
systemctl --user restart freellmapi.service
systemctl --user stop freellmapi.service
systemctl --user disable --now freellmapi.service
```

Ctrl+C при просмотре journalctl закрывает просмотр, не сервис. При смене версии Node через nvm пересоздайте unit с новыми путями. Сон/выключение ноутбука делает сервис недоступным.

## 4. Обновление приложения

Перед обновлением проверьте изменения и сделайте резервную копию `.env`, базы данных и связанных секретов/ключа шифрования в фактических путях вашей версии. Эти пути в переписке не установлены. Согласованную копию базы делайте при остановленном приложении; храните копию вне Git с ограниченным доступом.

```bash
cd ~/freellmapi
# Если есть незакоммиченные изменения — сначала разберите и сохраните их.
git status --short
systemctl --user stop freellmapi.service
# Здесь сделайте резервную копию данных и настроек.
git pull --ff-only && npm install && systemctl --user start freellmapi.service
systemctl --user status freellmapi.service --no-pager
journalctl --user -u freellmapi.service -n 80 --no-pager
```

При ошибке pull/install цепочка не запустит сервис: сначала устраните причину. Не используйте `git reset --hard` или удаление базы как способ обновления. Перед крупным обновлением прочитайте release notes.

Обновление кода и каталога моделей — разные вещи: каталог поступает автоматически; Premium ускоряет получение каталога, но не увеличивает квоты провайдеров.

## 5. Типовые ошибки

| Ошибка | Что проверять |
|---|---|
| `ECONNREFUSED 127.0.0.1:3001` | Процесс, порт, адрес и наличие туннеля для удалённого клиента |
| `Invalid API key` | Единый ключ в клиенте и правильный Base URL; затем логи для различения отказа роутера и внешнего провайдера |
| `All models exhausted` | Доступные маршруты, квоты и cooldown; перезапуск не пополняет квоты |
| Панель не принимает пароль | Email/пароль панели, не единый API-ключ |

Для забытого пароля серверной установки используйте Forgot password / Send reset code. По документации одноразовый код выводится в серверные логи, действует 15 минут. Не отправляйте эти логи без удаления кодов и секретов.

