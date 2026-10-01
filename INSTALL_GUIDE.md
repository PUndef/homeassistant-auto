# 🎙️ Полное руководство по установке локального голосового ассистента для Home Assistant

> ⚠️ **Внимание:** Этот гайд для Home Assistant Core/Container в LXC.  
> **Если у вас Home Assistant OS (VM в Proxmox)** → используйте [INSTALL_GUIDE_HAOS.md](INSTALL_GUIDE_HAOS.md)

## 📋 Что будет установлено

- ✅ **HACS** - магазин дополнений
- ✅ **Faster Whisper** - локальное распознавание речи (русский язык)
- ✅ **Piper TTS** - локальный синтез речи (русский голос)
- ✅ **Ollama** - локальная LLM модель для умного понимания команд
- ✅ **Extended OpenAI Conversation** - интеграция LLM с Home Assistant
- ✅ **Weather Voice Blueprint** - голосовой прогноз погоды

## 🎯 Требования

- Home Assistant в Proxmox (установлен ✅)
- Минимум 2 ГБ RAM для HA + 4 ГБ для Ollama
- Подключение к интернету для установки (после установки всё работает локально)

---

## Шаг 1: Установка HACS

### 1.1 Подключитесь к консоли Home Assistant

В Proxmox:
1. Выберите контейнер Home Assistant
2. Нажмите **Console**

Или через SSH с хоста Proxmox:

```bash
# Узнайте ID контейнера
pct list

# Войдите в контейнер (замените 100 на ваш ID)
pct enter 100
```

### 1.2 Установите HACS

```bash
wget -O - https://get.hacs.xyz | bash -
```

### 1.3 Перезапустите Home Assistant

В веб-интерфейсе:
```
Settings → System → Restart
```

Подождите 2-3 минуты.

### 1.4 Настройте HACS

После перезапуска:

1. `Settings → Devices & Services → Add Integration`
2. Найдите **HACS**
3. Следуйте инструкциям:
   - Примите условия
   - Авторизуйтесь через GitHub (если нет аккаунта - создайте на github.com)
   - Введите код авторизации

✅ HACS установлен!

---

## Шаг 2: Установка Faster Whisper (распознавание речи)

### 2.1 Установите через HACS

1. Откройте **HACS** (боковое меню)
2. **Integrations** → кнопка меню (три точки) → **Custom repositories**
3. Добавьте: `https://github.com/rhasspy/hassio-addons`
   - Category: **Integration**
4. Нажмите **Explore & Download Repositories**
5. Найдите **Whisper** или **Faster Whisper**
6. **Download**
7. **Restart Home Assistant**

### 2.2 Настройте Whisper

После перезапуска:

1. `Settings → Add-ons → Add-on Store`
2. Найдите **Whisper** 
3. Нажмите **Install** (может занять 5-10 минут)
4. После установки:
   - **Configuration**:
     ```yaml
     model: base
     language: ru
     beam_size: 1
     ```
   - Включите **Start on boot**
   - Нажмите **Start**

### 2.3 Добавьте интеграцию

1. `Settings → Devices & Services → Add Integration`
2. Найдите **Whisper**
3. Выберите установленный аддон

✅ Распознавание речи готово!

---

## Шаг 3: Установка Piper (синтез речи)

### 3.1 Установите Piper

Piper встроен в Home Assistant!

1. `Settings → Add-ons → Add-on Store`
2. Найдите **Piper**
3. **Install**
4. После установки:
   - Включите **Start on boot**
   - **Start**

### 3.2 Добавьте интеграцию

1. `Settings → Devices & Services → Add Integration`
2. Найдите **Piper**
3. Выберите голос: **ru_RU-ruslan-medium** (мужской) или **ru_RU-irina-medium** (женский)

✅ Синтез речи готов!

---

## Шаг 4: Установка Ollama (локальная LLM)

### 4.1 Создайте контейнер для Ollama в Proxmox

В консоли Proxmox:

```bash
# Обновите список шаблонов
pveam update

# Скачайте Ubuntu 22.04
pveam download local ubuntu-22.04-standard_22.04-1_amd64.tar.zst

# Узнайте свободный ID (например, если HA - это 100, то используйте 101)
pct list

# Создайте контейнер (замените 101 на свободный ID)
pct create 101 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname ollama \
  --memory 4096 \
  --cores 2 \
  --storage local-lvm \
  --rootfs 20 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --unprivileged 1 \
  --onboot 1 \
  --start 1

# Подождите 10 секунд для загрузки

# Войдите в контейнер
pct enter 101
```

### 4.2 Установите Ollama в контейнере

```bash
# Обновите систему
apt update && apt upgrade -y

# Установите Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Запустите сервис
systemctl start ollama
systemctl enable ollama

# Проверьте статус
systemctl status ollama
```

### 4.3 Скачайте модель

```bash
# Компактная модель (рекомендуется для начала) - 2GB
ollama pull llama3.2:3b-instruct-q4_K_M

# Или более умная модель - 4GB (требует больше памяти)
# ollama pull qwen2.5:7b-instruct-q4_K_M
```

Скачивание займет 5-15 минут в зависимости от скорости интернета.

### 4.4 Узнайте IP адрес контейнера

```bash
# Узнайте IP адрес
ip addr show eth0 | grep "inet "
```

Вы увидите что-то вроде: `inet 192.168.1.50/24`

**Запомните этот IP!** (например, `192.168.1.50`)

### 4.5 Проверьте доступ

```bash
# Проверьте что Ollama работает
curl http://localhost:11434/api/tags
```

Должен вернуться JSON со списком моделей.

### 4.6 Выйдите из контейнера

```bash
exit
```

✅ Ollama установлен!

---

## Шаг 5: Установка Extended OpenAI Conversation

### 5.1 Установите через HACS

1. Откройте **HACS**
2. **Integrations** → **Explore & Download Repositories**
3. Найдите **Extended OpenAI Conversation**
4. **Download**
5. **Restart Home Assistant**

### 5.2 Настройте интеграцию

После перезапуска:

1. `Settings → Devices & Services → Add Integration`
2. Найдите **Extended OpenAI Conversation**
3. Настройте:
   - **API Base URL**: `http://192.168.1.50:11434/v1` (замените на IP из шага 4.4)
   - **API Key**: `not-needed` (любой текст)
   - **Model**: `llama3.2:3b-instruct-q4_K_M`
   - **Max Tokens**: `150`
   - **Context Threshold**: `0.85`

4. После добавления, нажмите **Configure** на интеграции
5. **Options** → **Prompt**:

```
Ты умный голосовой ассистент для управления умным домом на русском языке.

ПРАВИЛА:
1. Отвечай КРАТКО - максимум 1-2 коротких предложения
2. Используй естественный разговорный русский язык
3. Всегда подтверждай выполненные действия
4. Если не уверен - переспроси
5. Не придумывай информацию - используй только данные из Home Assistant

ПРИМЕРЫ ОТВЕТОВ:
Пользователь: "Включи свет в гостиной"
Ты: "Включил свет в гостиной"

Пользователь: "Какая температура?"
Ты: "Температура 22 градуса"

Пользователь: "Какая погода?"
Ты: "Сейчас 15 градусов и ясно"
```

✅ LLM подключена!

---

## Шаг 6: Настройка конфигурации Home Assistant

### 6.1 Откройте File Editor

1. `Settings → Add-ons → Add-on Store`
2. Найдите **File editor** или **Studio Code Server**
3. Установите и запустите
4. Откройте через боковое меню

### 6.2 Отредактируйте configuration.yaml

Добавьте в конец файла `configuration.yaml`:

```yaml
# ==================== ГОЛОСОВОЙ АССИСТЕНТ ====================

# Пайплайн для голосового управления
assist_pipeline:

# Conversation agent
conversation:
  intents:
    HassTurnOn:
    HassTurnOff:
    HassLightSet:
    HassClimateSetTemperature:
    HassSetPosition:

# Логирование для отладки
logger:
  default: info
  logs:
    homeassistant.components.conversation: debug
    homeassistant.components.assist_pipeline: debug

# Intent Scripts (кастомные команды)
intent_script: !include intent_script.yaml
```

### 6.3 Создайте intent_script.yaml

Создайте новый файл `intent_script.yaml` рядом с `configuration.yaml` и скопируйте содержимое из файла `intent_script.yaml` из этого репозитория.

### 6.4 Проверьте конфигурацию

```
Developer Tools → YAML → Check Configuration
```

Если ошибок нет:

```
Settings → System → Restart
```

---

## Шаг 7: Создание голосового пайплайна

После перезапуска:

### 7.1 Создайте ассистента

1. `Settings → Voice assistants`
2. **Add Assistant**
3. Настройте:
   - **Name**: `Домашний ассистент`
   - **Conversation agent**: `Extended OpenAI Conversation` (или имя вашей интеграции)
   - **Language**: `Russian (ru)`
   - **Speech-to-text**: `faster-whisper` (или Whisper)
   - **Text-to-speech**: `piper` (ru_RU-ruslan-medium)
   - **Wake word**: `Не выбрано` (для начала)

4. **Create**

### 7.2 Установите как основной

Нажмите на созданный ассистент → **Set as preferred**

✅ Голосовой ассистент готов!

---

## Шаг 8: Импорт Weather Voice Blueprint

### 8.1 Импортируйте blueprint

1. `Settings → Automations & Scenes → Blueprints`
2. **Import Blueprint**
3. URL: скопируйте содержимое файла `Weather Voice.yaml` из этого репозитория
   - Или загрузите через File Editor

### 8.2 Создайте автоматизацию

1. **Blueprints** → **Weather Voice**
2. **Create Automation**
3. Настройте:
   - **Weather Entity**: выберите вашу погодную сущность
   - Остальные настройки оставьте по умолчанию
4. **Save**

---

## Шаг 9: Expose entities для управления

### 9.1 Настройте доступные устройства

1. `Settings → Voice assistants → Expose`
2. Включите устройства, которыми хотите управлять голосом:
   - Свет
   - Климат
   - Переключатели
   - Розетки
   и т.д.

---

## 🧪 Тестирование

### Тест 1: Через веб-интерфейс

1. Откройте Home Assistant
2. Нажмите **иконку микрофона** в правом нижнем углу
3. Скажите: **"Какая погода сегодня?"**

### Тест 2: Базовые команды

Попробуйте сказать:
- "Который час?"
- "Какая температура?"
- "Включи свет в гостиной" (если есть такое устройство)
- "Что включено?"

### Тест 3: Умные команды через LLM

- "Мне холодно" (должен предложить включить отопление)
- "Темно" (должен предложить включить свет)
- "Расскажи о погоде"

---

## 🔧 Отладка

### Проблема: Ассистент не отвечает

1. Проверьте логи:
```
Settings → System → Logs
```

2. Проверьте что все сервисы запущены:
   - `Settings → Add-ons` - Whisper и Piper должны быть зелеными
   - В Proxmox контейнер Ollama должен быть запущен

### Проблема: Ollama не отвечает

В консоли контейнера Ollama:

```bash
systemctl status ollama
curl http://localhost:11434/api/tags
```

### Проблема: Плохо распознает речь

1. Улучшите качество микрофона
2. Говорите четко и не слишком быстро
3. В настройках Whisper увеличьте `beam_size` до 5

### Проблема: Медленные ответы

1. Используйте более компактную модель: `llama3.2:3b-instruct-q4_K_M`
2. Уменьшите `Max Tokens` до 100
3. Добавьте больше RAM контейнеру Ollama

---

## 📚 Полезные команды

### Управление контейнером Ollama

```bash
# Запустить
pct start 101

# Остановить
pct stop 101

# Войти в консоль
pct enter 101

# Перезапустить Ollama
systemctl restart ollama

# Список моделей
ollama list

# Удалить модель
ollama rm llama3.2:3b-instruct-q4_K_M

# Скачать другую модель
ollama pull qwen2.5:7b-instruct-q4_K_M
```

---

## 🎉 Готово!

Теперь у вас есть полностью локальный голосовой ассистент:

✅ Распознает русскую речь локально  
✅ Понимает команды через LLM  
✅ Отвечает русским голосом  
✅ Не требует интернета для работы (после установки)  
✅ Управляет устройствами умного дома  
✅ Рассказывает погоду  

---

## 🚀 Следующие шаги

1. **Добавьте кастомные команды** в `intent_script.yaml`
2. **Настройте wake word** (например, через Wyoming Protocol)
3. **Добавьте больше автоматизаций** с conversation triggers
4. **Настройте спутниковые микрофоны** в разных комнатах

---

## 📖 Дополнительные ресурсы

- [Home Assistant Voice Docs](https://www.home-assistant.io/voice_control/)
- [Ollama Models](https://ollama.com/library)
- [Piper Voices](https://rhasspy.github.io/piper-samples/)
- [Weather Voice Blueprint GitHub](https://github.com/TheFes/ha-blueprints)
