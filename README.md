# Русский голосовой прогноз погоды для Home Assistant

Интеграция для Home Assistant, позволяющая получать прогноз погоды голосом на русском языке через Local Assist.

## 🎯 Возможности

- Поддержка естественных русских фраз
- Прогноз на сегодня, завтра, послезавтра
- Прогноз на конкретные дни недели (понедельник, вторник и т.д.)
- Прогноз по времени суток (утро, день, вечер, ночь)
- Информация о температуре, ощущаемой температуре
- Вероятность осадков и скорость ветра

## 📋 Примеры команд

- "Какая погода?"
- "Какая погода сегодня?"
- "Погода на завтра"
- "Прогноз погоды на понедельник"
- "Какая погода утром?"
- "Погода вечером"

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

## ⚠️ Почему не Blueprint?

**Важно:** Для голосовых команд нужно использовать Intent Script, а не blueprint с conversation trigger!

Причины:
- Automations с conversation triggers имеют [известные баги](https://github.com/home-assistant/core/issues/109285)
- `set_conversation_response` не работает корректно
- `stop` возвращает только "Готово" вместо текста

Intent Script - это правильный способ для голосовых команд.

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
├── BLUEPRINT_GUIDE.md             # Полное руководство по blueprints
├── weather_voice_intent.yaml      # Intent script для погоды
├── weather_voice_sentences.yaml   # Фразы для распознавания
└── Weather Voice.yaml             # Устаревший blueprint (не используется)
```

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

**Примечание:** Файл `Weather Voice.yaml` является устаревшим blueprint подходом и оставлен только для справки. Используйте Intent Script подход описанный выше.
