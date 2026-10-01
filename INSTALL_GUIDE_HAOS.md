# 🎙️ Установка локального голосового ассистента для Home Assistant OS (Proxmox)

> **Для Home Assistant OS установленного через скрипт Proxmox**

## 📋 Что будет установлено

- ✅ **HACS** - магазин дополнений
- ✅ **Whisper** - локальное распознавание речи (русский)
- ✅ **Piper TTS** - локальный синтез речи (русский голос)
- ✅ **Ollama** - локальная LLM (в отдельном LXC контейнере)
- ✅ **Extended OpenAI Conversation** - интеграция LLM
- ✅ **Weather Voice Blueprint** - голосовой прогноз погоды

## 🎯 Требования

- ✅ Home Assistant OS в Proxmox (VM)
- Минимум 2 ГБ RAM для HAOS
- Дополнительно 4 ГБ RAM для Ollama (отдельный контейнер)
- Интернет для установки

---

## Шаг 1: Установка HACS

### 1.1 Включите Advanced Mode

В Home Assistant:
1. Нажмите на ваш профиль (левый нижний угол)
2. Включите **Advanced Mode**

### 1.2 Установите Terminal & SSH Add-on

1. `Settings → Add-ons → Add-on Store`
2. Найдите **Terminal & SSH**
3. Нажмите **Install**
4. После установки:
   - **Configuration**: оставьте по умолчанию
   - Включите **Show in sidebar**
   - Включите **Start on boot**
   - Нажмите **Start**

### 1.3 Откройте Terminal

В боковом меню появится **Terminal**. Откройте его.

### 1.4 Установите HACS

В терминале выполните:

```bash
wget -O - https://get.hacs.xyz | bash -
```

Если команда `wget` не найдена, используйте альтернативный метод:

```bash
# Скачайте HACS вручную
cd /config
mkdir -p custom_components
cd custom_components
apk add --no-cache git
git clone https://github.com/hacs/integration.git hacs
```

### 1.5 Перезапустите Home Assistant

```
Settings → System → Restart
```

Подождите 2-3 минуты.

### 1.6 Настройте HACS

После перезапуска:

1. `Settings → Devices & Services → Add Integration`
2. Найдите **HACS**
3. Следуйте инструкциям:
   - Примите все условия
   - Авторизуйтесь через GitHub
   - Введите код авторизации с сайта GitHub

✅ HACS установлен!

---

## Шаг 2: Установка Whisper (распознавание речи)

### 2.1 Установите Whisper Add-on

1. `Settings → Add-ons → Add-on Store`
2. Найдите **Whisper** в разделе "Official add-ons"
3. Нажмите **Install** (может занять 5-10 минут)

### 2.2 Настройте Whisper

После установки:

1. Перейдите на страницу add-on **Whisper**
2. **Configuration**:

```yaml
language: ru
model: base
beam_size: 1
```

3. Включите **Start on boot**
4. Включите **Watchdog**
5. Нажмите **Start**

### 2.3 Добавьте интеграцию Whisper

1. `Settings → Devices & Services → Add Integration`
2. Найдите **Whisper**
3. Выберите установленный add-on

✅ Распознавание речи готово!

---

## Шаг 3: Установка Piper (синтез речи)

### 3.1 Установите Piper Add-on

1. `Settings → Add-ons → Add-on Store`
2. Найдите **Piper** в разделе "Official add-ons"
3. Нажмите **Install** (5-10 минут)

### 3.2 Настройте Piper

После установки:

1. Перейдите на страницу add-on **Piper**
2. Включите **Start on boot**
3. Включите **Watchdog**
4. Нажмите **Start**

### 3.3 Добавьте интеграцию Piper

1. `Settings → Devices & Services → Add Integration`
2. Найдите **Piper**
3. Выберите голос: **ru_RU-ruslan-medium** (мужской) или **ru_RU-irina-medium** (женский)

✅ Синтез речи готов!

---

## Шаг 4: Установка Ollama (локальная LLM)

> Ollama устанавливается в отдельный LXC контейнер в Proxmox

### 4.1 Откройте Shell Proxmox

В веб-интерфейсе Proxmox:
1. Выберите ваш хост (узел)
2. Нажмите **Shell**

---

### Вариант А: Ручная установка (РЕКОМЕНДУЕТСЯ) ⭐

**Надежный метод, работает на всех системах**

### 4.2 Скачайте шаблон Ubuntu

```bash
# Обновите список шаблонов
pveam update

# Скачайте Ubuntu 24.04 (новая версия, более стабильная)
pveam download local ubuntu-24.04-standard_24.04-2_amd64.tar.zst
```

### 4.3 Создайте контейнер для Ollama

```bash
# Узнайте свободный ID
pct list

# Создайте контейнер (замените 101 на свободный ID)
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
```

**Важно:** `--swap 4096` необходим для работы модели 3b в LXC контейнерах!

Подождите 10-15 секунд для запуска контейнера.

### 4.4 Войдите в контейнер

```bash
pct enter 101
```

### 4.5 Установите Ollama

В контейнере выполните команды последовательно:

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

Должно быть: `active (running)` ✅

---

### Вариант Б: Автоматический скрипт

<details>
<summary>⚠️ Может давать ошибки на некоторых системах</summary>

**Используем готовый скрипт от сообщества:**

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/ollama.sh)"
```

**Известные проблемы:**
- ❌ Ошибка `gpg --dearmor` при установке Intel Graphics драйверов
- ❌ `exit code 2` на некоторых CPU
- ⚠️ Скрипт может не завершиться полностью

**Если скрипт не сработал:**

1. Контейнер уже частично создан
2. Войдите в него: `pct enter 101`
3. Доустановите Ollama вручную:
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   systemctl start ollama
   systemctl enable ollama
   ```

**Рекомендация:** Используйте Вариант А для гарантированного результата.

</details>

---

### 4.6 Настройте Ollama для внешнего доступа

**Важно!** По умолчанию Ollama слушает только localhost. Настройте для доступа из Home Assistant:

```bash
# Создайте конфигурацию для systemd
mkdir -p /etc/systemd/system/ollama.service.d

cat > /etc/systemd/system/ollama.service.d/override.conf << 'EOF'
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_MAX_LOADED_MODELS=1"
Environment="OLLAMA_NUM_PARALLEL=1"
EOF

# Перезагрузите конфигурацию
systemctl daemon-reload
systemctl restart ollama

# Проверьте что слушает на всех интерфейсах
ss -tulpn | grep 11434
# Должно быть: *:11434
```

### 4.7 Скачайте модель

```bash
# Модель 3b (рекомендуется) - 2GB
ollama pull llama3.2:3b-instruct-q4_K_M

# Проверьте что модель работает
ollama run llama3.2:3b-instruct-q4_K_M "Привет"
```

Это займет 5-15 минут в зависимости от скорости интернета.

**Если получаете ошибку "out of memory":**
```bash
exit  # Выйдите из контейнера

# В Proxmox Shell увеличьте swap
pct set 101 -swap 4096
pct reboot 101
```

**Альтернативные модели:**
```bash
# Более умная модель - 4GB (нужно больше RAM)
# ollama pull qwen2.5:7b-instruct-q4_K_M

# Совсем маленькая модель - 1GB (если мало памяти)
# ollama pull llama3.2:1b-instruct-q4_K_M
```

### 4.8 Узнайте IP адрес контейнера

```bash
ip addr show eth0 | grep "inet "
```

Вы увидите: `inet 192.168.X.Y/24`

**Запишите этот IP!** (например, `192.168.50.140`)

### 4.9 Проверьте доступность API

```bash
# Локально в контейнере
curl http://localhost:11434/api/tags

# По IP (замените на ваш IP)
curl http://192.168.50.140:11434/api/tags

# Оба должны вернуть JSON со списком моделей
```

### 4.10 Выйдите из контейнера

```bash
exit
```

### 4.11 Проверьте доступность из Proxmox

```bash
# В Proxmox Shell (замените IP)
curl http://192.168.50.140:11434/api/tags
```

Если возвращает JSON - всё работает! ✅

✅ Ollama установлен и работает!

---

## Шаг 5: Установка Extended OpenAI Conversation

### 5.1 Установите через HACS

1. Откройте **HACS** (боковое меню)
2. **Integrations** → **Explore & Download Repositories**
3. В поиске найдите: **Extended OpenAI Conversation**
4. Нажмите на репозиторий
5. **Download**
6. Выберите последнюю версию

### 5.2 Перезапустите Home Assistant

```
Settings → System → Restart
```

### 5.3 Настройте интеграцию

После перезапуска:

1. `Settings → Devices & Services → Add Integration`
2. Найдите **Extended OpenAI Conversation**
3. Заполните поля:

**Важные настройки:**
- **API Base URL**: `http://192.168.1.50:11434/v1` (замените на IP из шага 4.7)
- **API Key**: `not-needed` (любой текст)
- **Model**: `llama3.2:3b-instruct-q4_K_M`
- **Max Tokens**: `150`
- **Temperature**: `0.7`
- **Context Threshold**: `0.85`

4. Нажмите **Submit**

### 5.4 Настройте промпт

1. Перейдите в `Settings → Devices & Services`
2. Найдите **Extended OpenAI Conversation**
3. Нажмите **Configure**
4. **Options** → **Prompt**

Вставьте:

```
Ты умный голосовой ассистент для управления умным домом на русском языке.

ПРАВИЛА:
1. Отвечай КРАТКО - максимум 1-2 коротких предложения
2. Используй естественный разговорный русский язык
3. Всегда подтверждай выполненные действия
4. Если не уверен - переспроси
5. Не придумывай информацию - используй только данные из Home Assistant

ПРИМЕРЫ:
Пользователь: "Включи свет в гостиной"
Ты: "Включил свет в гостиной"

Пользователь: "Какая температура?"
Ты: "Температура 22 градуса"

Пользователь: "Какая погода?"
Ты: "Сейчас 15 градусов и ясно"
```

5. **Submit**

✅ LLM подключена!

---

## Шаг 6: Настройка конфигурации Home Assistant

### 6.1 Установите File Editor

1. `Settings → Add-ons → Add-on Store`
2. Найдите **File editor** (Official add-on)
3. **Install**
4. Включите **Show in sidebar**
5. Включите **Start on boot**
6. **Start**

### 6.2 Откройте File Editor

В боковом меню появится **File editor**

### 6.3 Отредактируйте configuration.yaml

Откройте файл `configuration.yaml` и добавьте в конец:

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

# Логирование для отладки (опционально)
logger:
  default: info
  logs:
    homeassistant.components.conversation: debug
    homeassistant.components.assist_pipeline: debug
    custom_components.extended_openai_conversation: debug

# Intent Scripts (кастомные команды)
intent_script: !include intent_script.yaml
```

### 6.4 Создайте intent_script.yaml

Создайте новый файл `intent_script.yaml` рядом с `configuration.yaml`:

```yaml
# Intent Scripts - кастомные голосовые команды

# Справка
HelpIntent:
  speech:
    text: "Я могу включать свет, управлять климатом, рассказывать о погоде и времени"
  action: []

# Время
GetTime:
  speech:
    text: "Сейчас {{ now().strftime('%H:%M') }}"
  action: []

# Дата
GetDate:
  speech:
    text: |
      {% set days = ['понедельник', 'вторник', 'среда', 'четверг', 'пятница', 'суббота', 'воскресенье'] %}
      {% set months = ['января', 'февраля', 'марта', 'апреля', 'мая', 'июня', 'июля', 'августа', 'сентября', 'октября', 'ноября', 'декабря'] %}
      Сегодня {{ days[now().weekday()] }}, {{ now().day }} {{ months[now().month - 1] }}
  action: []

# Статус дома
HomeStatus:
  speech:
    text: |
      {% set lights_on = states.light | selectattr('state', 'eq', 'on') | list | count %}
      {% if lights_on > 0 %}
        Включено {{ lights_on }} {{ 'лампа' if lights_on == 1 else ('лампы' if lights_on < 5 else 'ламп') }}
      {% else %}
        Весь свет выключен
      {% endif %}
  action: []

# Температура
HomeTemperature:
  speech:
    text: |
      {% set temp_sensors = states.sensor | selectattr('attributes.device_class', 'eq', 'temperature') | list %}
      {% if temp_sensors | count > 0 %}
        {% set avg_temp = temp_sensors | map(attribute='state') | map('float') | select('number') | average | round(1) %}
        Средняя температура {{ avg_temp }} градусов
      {% else %}
        Датчики температуры не найдены
      {% endif %}
  action: []

# Спокойной ночи
GoodNight:
  speech:
    text: "Спокойной ночи! Выключаю свет"
  action:
    - service: light.turn_off
      target:
        entity_id: all
```

### 6.5 Проверьте конфигурацию

```
Developer Tools → YAML → Check Configuration
```

Если ошибок нет → **Restart**

```
Settings → System → Restart
```

✅ Конфигурация применена!

---

## Шаг 7: Создание голосового пайплайна

### 7.1 Создайте ассистента

После перезапуска:

1. `Settings → Voice assistants`
2. **Add Assistant**
3. Настройте:

**Name**: `Домашний ассистент`

**Conversation agent**: Выберите `extended_openai_conversation` (или имя вашей интеграции)

**Language**: `Russian (ru)`

**Speech-to-text**: `faster-whisper`

**Text-to-speech**: `piper` → выберите `ru_RU-ruslan-medium`

**Wake word**: Оставьте пустым (для начала)

4. **Create**

### 7.2 Установите как основной

Нажмите на созданный ассистент → **Prefer**

✅ Голосовой ассистент готов!

---

## Шаг 8: Expose entities для управления

### 8.1 Откройте настройки Expose

```
Settings → Voice assistants → Expose
```

### 8.2 Включите устройства

Включите переключатели для устройств, которыми хотите управлять голосом:
- 💡 Свет (light)
- 🌡️ Климат (climate)
- 🔌 Розетки (switch)
- 🪟 Шторы (cover)
- и т.д.

✅ Устройства доступны для голосового управления!

---

## Шаг 9: Импорт Weather Voice Blueprint

### 9.1 Установите File Editor (если ещё не установлен)

См. Шаг 6.1

### 9.2 Загрузите Weather Voice.yaml

1. Откройте **File editor**
2. Найдите папку `blueprints/automation/`
3. Создайте папку `weather/` (если нет)
4. Создайте файл `weather_voice_ru.yaml`
5. Скопируйте содержимое из `Weather Voice.yaml` из репозитория

Или используйте Terminal:

```bash
cd /config/blueprints/automation
mkdir -p weather
cd weather
# Скопируйте содержимое Weather Voice.yaml в файл
```

### 9.3 Импортируйте blueprint

1. `Settings → Automations & Scenes → Blueprints`
2. Blueprint должен появиться автоматически
3. Если нет - нажмите кнопку обновления

### 9.4 Создайте автоматизацию

1. Нажмите на blueprint **Голосовой прогноз погоды (RU)**
2. **Create Automation**
3. Настройте:
   - **Weather Entity**: выберите вашу погодную сущность (например, `weather.home`)
   - Остальное можно оставить по умолчанию
4. **Save**

✅ Голосовой прогноз погоды работает!

---

## 🧪 Тестирование

### Тест 1: Через веб-интерфейс

1. Откройте Home Assistant
2. Нажмите **иконку микрофона** в правом нижнем углу
3. Скажите: **"Какая погода сегодня?"**

### Тест 2: Базовые команды

Попробуйте:
- "Который час?"
- "Какая дата?"
- "Какая температура?"
- "Что включено?"

### Тест 3: Управление устройствами

- "Включи свет в [название комнаты]"
- "Выключи всё"
- "Установи температуру 22 градуса"

### Тест 4: Умные команды через LLM

- "Мне холодно"
- "Темно"
- "Расскажи о погоде"

---

## 🔧 Отладка

### Проблема: Ассистент не отвечает

1. **Проверьте логи:**
   ```
   Settings → System → Logs
   ```

2. **Проверьте статус add-ons:**
   ```
   Settings → Add-ons
   ```
   - Whisper должен быть зеленым
   - Piper должен быть зеленым

3. **Проверьте интеграции:**
   ```
   Settings → Devices & Services
   ```
   - Extended OpenAI Conversation должна быть активна

### Проблема: Ollama не отвечает

В Shell Proxmox:

```bash
# Проверьте статус контейнера
pct status 101

# Войдите в контейнер
pct enter 101

# Проверьте Ollama
systemctl status ollama

# Проверьте доступность API
curl http://localhost:11434/api/tags

# Выйдите
exit
```

Если Ollama не отвечает:

```bash
pct enter 101
systemctl restart ollama
systemctl status ollama
exit
```

### Проблема: Скрипт установки выдал ошибку Intel Graphics

**Ошибка:**
```
exit code 2: gpg --dearmor -o /usr/share/keyrings/intel-graphics.gpg
```

**Решение:**

Контейнер уже создан, просто доустановите Ollama:

```bash
# Войдите в контейнер
pct enter 101

# Установите Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Запустите сервис
systemctl start ollama
systemctl enable ollama

# Проверьте
systemctl status ollama

# Готово!
```

**Примечание:** Ошибка не критична. Скрипт пытался установить Intel GPU драйверы, которые не нужны для CPU режима Ollama.

### Проблема: Extended OpenAI Conversation не работает

1. Проверьте IP адрес Ollama:
   ```bash
   pct enter 101
   ip addr show eth0 | grep inet
   exit
   ```

2. Проверьте доступность с HA:
   - Terminal в HA:
   ```bash
   ping 192.168.1.50
   curl http://192.168.1.50:11434/api/tags
   ```

3. Пересоздайте интеграцию с правильным IP

### Проблема: Плохо распознает речь

1. **Улучшите микрофон**
2. **Говорите четко и не слишком быстро**
3. **Увеличьте beam_size:**
   - Settings → Add-ons → Whisper → Configuration
   - `beam_size: 5`
   - Restart add-on

### Проблема: Медленные ответы

1. **Используйте более легкую модель:**
   ```bash
   pct enter 101
   ollama pull llama3.2:1b-instruct-q4_K_M
   exit
   ```
   Затем измените модель в Extended OpenAI Conversation

2. **Уменьшите Max Tokens:**
   - Settings → Devices & Services → Extended OpenAI Conversation
   - Configure → Max Tokens: `100`

3. **Добавьте RAM контейнеру:**
   ```bash
   pct set 101 -memory 6144
   pct reboot 101
   ```

---

## 📚 Полезные команды для Home Assistant OS

### Управление через SSH

```bash
# Перезапуск HA
ha core restart

# Обновление HA
ha core update

# Статус supervisor
ha supervisor info

# Список add-ons
ha addons

# Логи HA
ha core logs

# Проверка конфигурации
ha core check
```

### Управление Ollama контейнером (из Proxmox Shell)

```bash
# Запустить контейнер
pct start 101

# Остановить
pct stop 101

# Перезапустить
pct reboot 101

# Войти в консоль
pct enter 101

# Статус
pct status 101

# Информация
pct config 101

# Изменить память
pct set 101 -memory 6144

# Изменить CPU
pct set 101 -cores 4
```

### Управление Ollama (внутри контейнера)

```bash
# Войти в контейнер
pct enter 101

# Список моделей
ollama list

# Удалить модель
ollama rm llama3.2:3b-instruct-q4_K_M

# Скачать модель
ollama pull qwen2.5:7b-instruct-q4_K_M

# Запустить модель вручную (тест)
ollama run llama3.2:3b-instruct-q4_K_M "Привет, как дела?"

# Перезапустить сервис
systemctl restart ollama

# Проверить логи
journalctl -u ollama -f

# Выйти
exit
```

---

## 🎉 Готово!

Теперь у вас есть полностью локальный голосовой ассистент для Home Assistant OS:

✅ Распознает русскую речь локально  
✅ Понимает команды через LLM  
✅ Отвечает русским голосом  
✅ Не требует интернета (после установки)  
✅ Управляет устройствами умного дома  
✅ Рассказывает погоду  

---

## 🚀 Следующие шаги

1. **Добавьте кастомные команды** в `intent_script.yaml`
2. **Настройте wake word** (Wyoming Protocol)
3. **Создайте автоматизации** с conversation triggers
4. **Добавьте спутниковые микрофоны** (ESP32, Raspberry Pi)

---

## 📖 Дополнительные ресурсы

- [Home Assistant Voice Docs](https://www.home-assistant.io/voice_control/)
- [HAOS Documentation](https://www.home-assistant.io/installation/)
- [Ollama Models](https://ollama.com/library)
- [Piper Voices](https://rhasspy.github.io/piper-samples/)
- [Extended OpenAI Conversation](https://github.com/jekalmin/extended_openai_conversation)

---

**⏱️ Общее время установки: ~2-3 часа**

**💬 Нужна помощь?** Проверьте раздел "Отладка" или откройте issue в репозитории.
