# 🏠 Home Assistant - Локальный голосовой ассистент

Полная настройка локального голосового ассистента для Home Assistant с поддержкой русского языка + голосовой прогноз погоды.

## ⚡ Быстрый выбор

### 🤔 Не знаете какой гайд использовать?

➡️ **[КАКОЙ_ГАЙД_ВЫБРАТЬ.md](КАКОЙ_ГАЙД_ВЫБРАТЬ.md)** - определите ваш тип установки HA

### 🎯 Вариант 1: Полный голосовой ассистент (НОВИНКА!)
**Для тех, кто хочет умный дом с голосовым управлением**

**Для HAOS (VM в Proxmox)** ⭐ - большинство пользователей:
➡️ **[QUICK_START_HAOS.md](QUICK_START_HAOS.md)** - начать установку (2-3 часа)  
➡️ **[INSTALL_GUIDE_HAOS.md](INSTALL_GUIDE_HAOS.md)** - подробное руководство

**Для Core/Container (LXC):**
➡️ **[QUICK_START.md](QUICK_START.md)** - начать установку (2 часа)  
➡️ **[INSTALL_GUIDE.md](INSTALL_GUIDE.md)** - подробное руководство

**Что получите:**
- 🎙️ Локальное распознавание речи (Faster Whisper)
- 🔊 Локальный синтез речи (Piper TTS, русский голос)
- 🧠 Умное понимание команд (LLM через Ollama)
- 🌤️ Голосовой прогноз погоды
- 💡 Управление устройствами голосом
- 🔒 Полная конфиденциальность - без облака!

### 🌤️ Вариант 2: Только прогноз погоды
**Для тех, кто хочет добавить только голосовую погоду**

➡️ См. секцию "Установка Weather Voice" ниже

---

## 🎯 Возможности полной системы

### Голосовое управление
- ✅ Локальное распознавание речи на русском
- ✅ Локальный синтез речи (мужской/женский голос)
- ✅ Умное понимание команд через LLM
- ✅ Работает полностью офлайн (после установки)

### Прогноз погоды
- ✅ Естественные русские фразы
- ✅ Прогноз на сегодня, завтра, послезавтра
- ✅ Прогноз на дни недели
- ✅ Прогноз по времени суток
- ✅ Температура, осадки, ветер

### Управление домом
- ✅ Включение/выключение света
- ✅ Управление климатом
- ✅ Управление розетками
- ✅ Сценарии (спокойной ночи, ухожу из дома)

## 📋 Примеры команд

### Погода
- "Какая погода сегодня?"
- "Какая погода завтра утром?"
- "Прогноз на понедельник"
- "Какая погода вечером?"

### Управление
- "Включи свет в гостиной"
- "Выключи всё"
- "Установи температуру 22 градуса"
- "Открой шторы"

### Информация
- "Который час?"
- "Какая дата?"
- "Какая температура в доме?"
- "Что включено?"

### Режимы
- "Спокойной ночи" (выключает свет)
- "Доброе утро" (включает свет)
- "Ухожу из дома" (режим отсутствия)

---

## 🚀 Установка полного голосового ассистента

**Время: ~2-3 часа** | **Сложность: Средняя**

### ⚡ Для Home Assistant OS (Proxmox VM) - РЕКОМЕНДУЕТСЯ

> **Если установили HA через официальный скрипт Proxmox**

📖 **[QUICK_START_HAOS.md](QUICK_START_HAOS.md)** - чеклист (быстро)  
📖 **[INSTALL_GUIDE_HAOS.md](INSTALL_GUIDE_HAOS.md)** - подробно

### 🐧 Для Home Assistant Core/Container (LXC)

> **Если HA установлен в LXC контейнере вручную**

📖 **[QUICK_START.md](QUICK_START.md)** - чеклист (быстро)  
📖 **[INSTALL_GUIDE.md](INSTALL_GUIDE.md)** - подробно

### Что будет установлено
1. HACS - магазин дополнений
2. Faster Whisper - распознавание речи
3. Piper - синтез речи
4. Ollama (в отдельном контейнере Proxmox) - LLM
5. Extended OpenAI Conversation - интеграция
6. Weather Voice Blueprint - прогноз погоды

### Требования
- Home Assistant в Proxmox
- 2 ГБ RAM для HA + 4 ГБ для Ollama
- Интернет для установки (работа - локально)

---

## 🌤️ Установка только Weather Voice

## 🚀 Установка

### Шаг 1: Добавьте Intent Script

Добавьте в ваш `configuration.yaml`:

```yaml
# Если у вас ещё нет секции intent_script
intent_script: !include weather_voice_intent.yaml

# Или если уже есть intent_script, добавьте содержимое weather_voice_intent.yaml
```

### Шаг 2: Создайте файл Custom Sentences

1. Создайте папки (если их нет):
   ```
   config/custom_sentences/ru/
   ```

2. Скопируйте содержимое файла `weather_voice_sentences.yaml` в:
   ```
   config/custom_sentences/ru/weather.yaml
   ```

### Шаг 3: Настройте погодную сущность

Откройте файл `weather_voice_intent.yaml` и замените:

```yaml
entity_id: weather.forecast_home_assistant
```

На вашу погодную сущность. Чтобы найти её:
1. Откройте Home Assistant
2. Настройки → Устройства и службы → Сущности
3. Фильтр по домену: `weather`
4. Скопируйте ID вашей сущности (например, `weather.home`)

Замените в трёх местах в файле:
- Строка 15: `entity_id: weather.forecast_home_assistant`
- Строка 21: `entity_id: weather.forecast_home_assistant`
- Строка 27: `weather_ent: weather.forecast_home_assistant`

### Шаг 4: Перезагрузите Home Assistant

Либо:
- Настройки → Система → Перезапуск

Либо только конфигурацию:
- Инструменты разработчика → YAML → Перезагрузить Intent Scripts

### Шаг 5: Проверьте работу

1. Откройте Инструменты разработчика → Assist
2. Введите или скажите: "Какая погода?"
3. Вы должны услышать голосовой ответ с прогнозом

## 📚 Документация

См. [`BLUEPRINT_GUIDE.md`](BLUEPRINT_GUIDE.md) - полное руководство по созданию blueprints и intent scripts для Home Assistant.

### Два подхода Weather Voice

#### Blueprint (Weather Voice.yaml)
✅ Простая настройка через UI  
✅ Работает с conversation triggers  
❌ Требует точные фразы  

#### Intent Script (weather_voice_intent.yaml)
✅ Максимальная гибкость  
✅ Стандартный подход HA  
❌ Сложнее настройка  

**Рекомендация:** Используйте Blueprint если устанавливаете полный голосовой ассистент (см. выше)

## 🔧 Продвинутая настройка

### Изменение фраз активации

Отредактируйте `config/custom_sentences/ru/weather.yaml`:

```yaml
intents:
  GetWeatherForecast:
    data:
      - sentences:
          - "ваша [новая] фраза"
          - "ещё одна фраза {phrase}"
```

### Добавление новых слотов

Добавьте в секцию `slots`:

```yaml
slots:
  phrase:
    - "сегодня"
    - "ваш_новый_период"
```

Затем обработайте их в `weather_voice_intent.yaml`.

### Изменение формата ответа

Отредактируйте секцию `speech` в `weather_voice_intent.yaml`:

```yaml
speech:
  type: plain
  text: "Ваш кастомный формат: {{ response_text }}"
```

## 🐛 Решение проблем

### Ассистент говорит "Готово" вместо прогноза

- Проверьте, что вы используете Intent Script, а не automation
- Убедитесь, что `custom_sentences/ru/weather.yaml` находится в правильной папке
- Перезагрузите конфигурацию intent scripts

### "Извините, я не понял"

- Проверьте правильность фраз в `custom_sentences/ru/weather.yaml`
- Убедитесь, что язык в Assist настроен на русский
- Проверьте логи: Настройки → Система → Логи

### Ошибка "Entity not found"

- Убедитесь, что погодная сущность существует
- Проверьте правильность ID сущности (должна начинаться с `weather.`)
- Замените во всех трёх местах в `weather_voice_intent.yaml`

### Неправильные данные погоды

- Убедитесь, что ваша погодная сущность обновляется
- Проверьте доступность `weather.get_forecasts` для вашей сущности
- Проверьте, что интеграция погоды настроена правильно

## 📝 Структура проекта

```
homeassistant-auto/
├── README.md                      # Этот файл
│
├── INSTALL_GUIDE_HAOS.md          # 📖 Установка для HAOS (Proxmox VM) ⭐
├── QUICK_START_HAOS.md            # ⚡ Быстрый старт для HAOS ⭐
├── TROUBLESHOOTING.md             # 🔧 Решение проблем
│
├── INSTALL_GUIDE.md               # 📖 Установка для HA Core/Container
├── QUICK_START.md                 # ⚡ Быстрый старт для Core/Container
│
├── КАКОЙ_ГАЙД_ВЫБРАТЬ.md          # 🤔 Определите тип вашей установки
├── BLUEPRINT_GUIDE.md             # Руководство по blueprints
├── ИСПРАВЛЕНИЕ.md                 # История исправлений
│
├── configuration_example.yaml     # Пример конфигурации HA
├── intent_script.yaml             # Кастомные голосовые команды
├── sentences_ru.yaml              # Дополнительные фразы
│
├── Weather Voice.yaml             # Weather blueprint (русский)
├── weather_voice_intent.yaml      # Intent script для погоды
└── weather_voice_sentences.yaml   # Фразы для распознавания
```

⭐ = Рекомендуется для большинства пользователей

## 🎯 С чего начать?

### У вас Home Assistant OS в Proxmox? (установлен через скрипт)

1. **Быстрый старт** → [QUICK_START_HAOS.md](QUICK_START_HAOS.md) ⭐
2. **Подробное руководство** → [INSTALL_GUIDE_HAOS.md](INSTALL_GUIDE_HAOS.md) ⭐

### У вас HA Core или Container (LXC)?

1. **Быстрый старт** → [QUICK_START.md](QUICK_START.md)
2. **Подробное руководство** → [INSTALL_GUIDE.md](INSTALL_GUIDE.md)

### Другие задачи

3. **Только прогноз погоды** → См. секцию выше
4. **Изучить детали** → [BLUEPRINT_GUIDE.md](BLUEPRINT_GUIDE.md)

## 🧪 Тестирование

После установки:

1. Откройте Home Assistant
2. Нажмите **иконку микрофона** (правый нижний угол)
3. Скажите: **"Какая погода сегодня?"**

Или попробуйте:
- "Который час?"
- "Включи свет"
- "Какая температура?"

## 🐛 Решение проблем

### CPU не поддерживает AVX
```bash
# В Proxmox Shell
qm stop 100
qm set 100 -cpu host
qm start 100
```

### Ollama скрипт выдал ошибку Intel Graphics
Контейнер создан, доустановите Ollama:
```bash
pct enter 101
curl -fsSL https://ollama.com/install.sh | sh
systemctl start ollama
```

### Медленные ответы
- Используйте модель `llama3.2:3b` или `1b`
- Уменьшите Max Tokens до 100
- Whisper модель: `tiny` вместо `base`

### Плохо распознает речь
- Проверьте микрофон
- Говорите четко
- Увеличьте beam_size в Whisper

📖 **Полное руководство:** [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

## 🤝 Благодарности

- [TheFes](https://github.com/TheFes/ha-blueprints) - оригинальный Weather Voice Blueprint
- [Home Assistant Community](https://www.home-assistant.io/)
- [Ollama](https://ollama.com/) - простой запуск LLM
- [Piper](https://github.com/rhasspy/piper) - качественный TTS
- [Faster Whisper](https://github.com/SYSTRAN/faster-whisper) - быстрый STT

## 🤝 Вклад

Если вы хотите улучшить этот проект:
1. Создайте issue с описанием проблемы или предложением
2. Отправьте pull request с изменениями

## 📄 Лицензия

Этот проект распространяется свободно для использования в Home Assistant.

## 🔗 Полезные ссылки

- [Home Assistant Documentation](https://www.home-assistant.io/docs/)
- [Intent Script](https://www.home-assistant.io/integrations/intent_script/)
- [Custom Sentences](https://www.home-assistant.io/voice_control/custom_sentences_yaml/)
- [Conversation Integration](https://www.home-assistant.io/integrations/conversation/)

---

## 🌟 Что дальше?

После установки базовой системы:

1. **Wake word** - активация по кодовому слову ("Окей, дом")
2. **Satellite микрофоны** - микрофоны в разных комнатах
3. **Кастомные команды** - добавьте свои команды в `intent_script.yaml`
4. **Автоматизации** - создайте умные сценарии с голосовыми триггерами

---

**⭐ Если проект полезен - поставьте звезду!**

**🎙️ Приятного использования локального голосового ассистента!**
