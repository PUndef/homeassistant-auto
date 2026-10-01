# ⚡ Быстрый старт - Локальный голосовой ассистент (Home Assistant OS)

> **Для Home Assistant OS в Proxmox (установлен через скрипт)**

## 🎯 Что делаем

Настраиваем полностью локальный голосовой ассистент для Home Assistant OS.

---

## 📝 Чеклист установки

### ✅ Этап 1: HACS (15 минут)

**В Home Assistant:**

1. Settings → Add-ons → Terminal & SSH → Install → Start
2. Открыть Terminal (боковое меню)
3. Выполнить:

```bash
wget -O - https://get.hacs.xyz | bash -
```

**Если wget не работает:**

```bash
cd /config/custom_components
apk add git
git clone https://github.com/hacs/integration.git hacs
```

4. Settings → System → Restart
5. Settings → Devices & Services → Add Integration → HACS
6. Авторизация через GitHub

---

### ✅ Этап 2: Whisper (20 минут)

1. Settings → Add-ons → Add-on Store → Whisper → Install
2. Configuration:
   ```yaml
   language: ru
   model: base
   beam_size: 1
   ```
3. Start on boot → Start
4. Settings → Devices & Services → Add Integration → Whisper

---

### ✅ Этап 3: Piper (10 минут)

1. Settings → Add-ons → Add-on Store → Piper → Install
2. Start on boot → Start
3. Settings → Devices & Services → Add Integration → Piper
4. Выбрать голос: `ru_RU-ruslan-medium`

---

### ✅ Этап 4: Ollama в Proxmox (15 минут)

**Вариант А: Ручная установка (РЕКОМЕНДУЕТСЯ)** ⭐

> **Надежный метод** - проверено работает на всех системах

```bash
# В Shell Proxmox

# 1. Скачать Ubuntu шаблон
pveam update
pveam download local ubuntu-24.04-standard_24.04-2_amd64.tar.zst

# 2. Создать контейнер (замените 101 на свободный ID)
pct create 101 local:vztmpl/ubuntu-24.04-standard_24.04-2_amd64.tar.zst \
  --hostname ollama \
  --memory 4096 \
  --swap 4096 \
  --cores 2 \
  --rootfs local-lvm:20 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --unprivileged 1 \
  --onboot 1 \
  --start 1

# 3. Подождать запуска (10 сек)
sleep 10

# 4. Войти в контейнер
pct enter 101

# 5. Обновить систему
apt update && apt upgrade -y

# 6. Установить Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 7. Запустить сервис
systemctl start ollama
systemctl enable ollama

# 8. Проверить статус
systemctl status ollama
# Должно быть: active (running)

# 9. Настроить для внешнего доступа
mkdir -p /etc/systemd/system/ollama.service.d
cat > /etc/systemd/system/ollama.service.d/override.conf << 'EOF'
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
EOF
systemctl daemon-reload
systemctl restart ollama

# 10. Скачать модель (2GB, займет 5-10 минут)
ollama pull llama3.2:3b-instruct-q4_K_M

# 11. Узнать IP адрес
ip addr show eth0 | grep "inet "
# Запомните IP! (например 192.168.50.140)

# 12. Проверить работу (должен вернуть JSON)
curl http://localhost:11434/api/tags

# 13. Выйти
exit

# ⚠️ Если получили ошибку "out of memory":
# pct set 101 -swap 4096
# pct reboot 101
# (swap уже установлен в команде выше, но если создавали вручную - добавьте)
```

**Вариант Б: Автоматический скрипт**

<details>
<summary>⚠️ Может не работать на некоторых системах (развернуть)</summary>

```bash
# В Shell Proxmox
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/ollama.sh)"

# Если скрипт выдаст ошибку с Intel Graphics:
# - Контейнер уже создан
# - Войдите: pct enter 101
# - Доустановите Ollama:
#   curl -fsSL https://ollama.com/install.sh | sh
#   systemctl start ollama
#   systemctl enable ollama
```

**Известные проблемы:**
- Ошибка `gpg --dearmor` на Intel Graphics
- Не критично - Ollama работает без GPU
- Используйте Вариант А для надежности

</details>

---

### ✅ Этап 5: Extended OpenAI Conversation (10 минут)

1. HACS → Integrations → Extended OpenAI Conversation → Download
2. Settings → System → Restart
3. Settings → Devices & Services → Add Integration → Extended OpenAI Conversation

**Настройки:**
- API Base URL: `http://192.168.1.50:11434/v1` (ваш IP)
- API Key: `not-needed`
- Model: `llama3.2:3b-instruct-q4_K_M`
- Max Tokens: `150`

4. Configure → Options → Prompt:

```
Ты голосовой ассистент умного дома на русском.
Отвечай кратко (1-2 предложения).
Используй естественный язык.
Подтверждай действия.
```

---

### ✅ Этап 6: Конфигурация HA (15 минут)

1. Settings → Add-ons → File editor → Install → Start

2. Открыть File editor

3. Добавить в `configuration.yaml`:

```yaml
assist_pipeline:

conversation:
  intents:
    HassTurnOn:
    HassTurnOff:
    HassLightSet:
    HassClimateSetTemperature:

intent_script: !include intent_script.yaml
```

4. Создать файл `intent_script.yaml`:

```yaml
GetTime:
  speech:
    text: "Сейчас {{ now().strftime('%H:%M') }}"
  action: []

HomeStatus:
  speech:
    text: |
      {% set lights_on = states.light | selectattr('state', 'eq', 'on') | list | count %}
      Включено {{ lights_on }} ламп
  action: []

GoodNight:
  speech:
    text: "Спокойной ночи! Выключаю свет"
  action:
    - service: light.turn_off
      target:
        entity_id: all
```

5. Developer Tools → Check Configuration → Restart

---

### ✅ Этап 7: Создание ассистента (5 минут)

Settings → Voice assistants → Add Assistant

**Настройки:**
- Name: `Домашний ассистент`
- Conversation agent: `extended_openai_conversation`
- Language: `Russian (ru)`
- Speech-to-text: `faster-whisper`
- Text-to-speech: `piper (ru_RU-ruslan-medium)`

→ Create → Prefer

---

### ✅ Этап 8: Expose устройства (5 минут)

Settings → Voice assistants → Expose

Включить устройства для голосового управления

---

### ✅ Этап 9: Weather Blueprint (10 минут)

1. File editor → создать `blueprints/automation/weather/weather_voice_ru.yaml`
2. Скопировать `Weather Voice.yaml` из репозитория
3. Settings → Automations → Blueprints
4. Create Automation → выбрать weather entity → Save

---

## 🧪 Тестирование

1. Нажать **микрофон** (правый нижний угол)
2. Сказать: **"Какая погода сегодня?"**
3. Попробовать: **"Который час?"**, **"Включи свет"**

---

## ⏱️ Общее время

**~2-3 часа** (с учетом загрузок)

---

## 🔥 Частые проблемы

### wget не работает в Terminal

```bash
cd /config/custom_components
apk add git
git clone https://github.com/hacs/integration.git hacs
```

### Ollama не отвечает

```bash
# В Proxmox Shell
pct enter 101
systemctl restart ollama
systemctl status ollama
curl http://localhost:11434/api/tags
exit
```

### Медленные ответы

```bash
# Используйте модель 1b вместо 3b
pct enter 101
ollama pull llama3.2:1b-instruct-q4_K_M
exit
```

Затем измените модель в Extended OpenAI Conversation

### Проверка IP Ollama из HA

В Terminal HA:
```bash
ping 192.168.1.50
curl http://192.168.1.50:11434/api/tags
```

---

## 📚 Команды для HAOS

### Home Assistant

```bash
# В Terminal HA
ha core restart
ha core logs
ha supervisor info
```

### Ollama (в Proxmox Shell)

```bash
# Управление контейнером
pct start 101
pct stop 101
pct reboot 101
pct status 101

# Войти в контейнер
pct enter 101

# Список моделей
ollama list

# Перезапустить Ollama
systemctl restart ollama

# Выйти
exit
```

---

## 📖 Полная документация

См. [INSTALL_GUIDE_HAOS.md](INSTALL_GUIDE_HAOS.md)

---

## 🎉 Готово!

**Локальный голосовой ассистент работает!**

✅ Спрашивать погоду  
✅ Управлять устройствами  
✅ Задавать вопросы  
✅ Всё локально!
