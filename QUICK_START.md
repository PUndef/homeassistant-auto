# ⚡ Быстрый старт - Локальный голосовой ассистент

> ⚠️ **Внимание:** Этот гайд для Home Assistant Core/Container в LXC.  
> **Если у вас Home Assistant OS (VM в Proxmox)** → используйте [QUICK_START_HAOS.md](QUICK_START_HAOS.md)

## 🎯 Что делаем

Настраиваем полностью локальный голосовой ассистент для Home Assistant в Proxmox.

---

## 📝 Чеклист установки

### ✅ Этап 1: HACS (15 минут)

```bash
# В консоли HA контейнера
wget -O - https://get.hacs.xyz | bash -
```

→ Перезапуск HA  
→ Settings → Devices & Services → Add Integration → HACS  
→ Авторизация через GitHub

---

### ✅ Этап 2: Whisper (20 минут)

→ Settings → Add-ons → Add-on Store → Whisper → Install  
→ Configuration: `model: base`, `language: ru`  
→ Start on boot → Start  
→ Settings → Devices & Services → Add Integration → Whisper

---

### ✅ Этап 3: Piper (10 минут)

→ Settings → Add-ons → Add-on Store → Piper → Install  
→ Start on boot → Start  
→ Settings → Devices & Services → Add Integration → Piper  
→ Выбрать голос: `ru_RU-ruslan-medium`

---

### ✅ Этап 4: Ollama в Proxmox (30 минут)

```bash
# В консоли Proxmox хоста

# Создать контейнер Ubuntu
pct create 101 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname ollama \
  --memory 4096 \
  --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --unprivileged 1 \
  --onboot 1 \
  --start 1

# Войти в контейнер
pct enter 101

# Установить Ollama
curl -fsSL https://ollama.com/install.sh | sh
systemctl start ollama
systemctl enable ollama

# Скачать модель (2GB)
ollama pull llama3.2:3b-instruct-q4_K_M

# Узнать IP
ip addr show eth0 | grep "inet "
# Запомните IP! (например 192.168.1.50)

# Выйти
exit
```

---

### ✅ Этап 5: Extended OpenAI Conversation (10 минут)

→ HACS → Integrations → Extended OpenAI Conversation → Download  
→ Перезапуск HA  
→ Settings → Devices & Services → Add Integration → Extended OpenAI Conversation

**Настройки:**
- API Base URL: `http://192.168.1.50:11434/v1` (ваш IP из этапа 4)
- API Key: `not-needed`
- Model: `llama3.2:3b-instruct-q4_K_M`
- Max Tokens: `150`

**Промпт (Options → Prompt):**
```
Ты голосовой ассистент умного дома на русском.
Отвечай кратко (1-2 предложения).
Используй естественный язык.
Подтверждай действия.
```

---

### ✅ Этап 6: Конфигурация HA (15 минут)

1. Установить File Editor:
   → Settings → Add-ons → File editor → Install → Start

2. Добавить в `configuration.yaml`:
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

3. Создать `intent_script.yaml` (скопировать из репозитория)

4. Check Configuration → Restart

---

### ✅ Этап 7: Создание ассистента (5 минут)

→ Settings → Voice assistants → Add Assistant

**Настройки:**
- Name: `Домашний ассистент`
- Conversation agent: `Extended OpenAI Conversation`
- Language: `Russian (ru)`
- Speech-to-text: `faster-whisper`
- Text-to-speech: `piper (ru_RU-ruslan-medium)`

→ Set as preferred

---

### ✅ Этап 8: Expose устройства (5 минут)

→ Settings → Voice assistants → Expose  
→ Включить устройства для голосового управления

---

### ✅ Этап 9: Weather Blueprint (5 минут)

→ Settings → Automations → Blueprints → Import Blueprint  
→ Загрузить `Weather Voice.yaml`  
→ Create Automation → Выбрать weather entity → Save

---

## 🧪 Тестирование

1. Нажать **микрофон** в правом нижнем углу
2. Сказать: **"Какая погода сегодня?"**
3. Попробовать: **"Который час?"**, **"Включи свет"**

---

## ⏱️ Общее время установки

**~2 часа** (с учетом загрузок)

---

## 🔥 Частые проблемы

### Ollama не отвечает
```bash
pct enter 101
systemctl status ollama
curl http://localhost:11434/api/tags
```

### Whisper не распознает
- Проверить микрофон
- Говорить четко
- Увеличить beam_size в настройках

### Медленные ответы
- Использовать модель 3b вместо 7b
- Уменьшить Max Tokens
- Добавить RAM контейнеру

---

## 📚 Полная документация

См. [INSTALL_GUIDE.md](INSTALL_GUIDE.md) для детального руководства.

---

## 🎉 Готово!

**Локальный голосовой ассистент работает!**

Теперь можно:
✅ Спрашивать погоду  
✅ Управлять устройствами  
✅ Задавать вопросы  
✅ Всё локально, без интернета
