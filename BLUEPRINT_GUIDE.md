# Полное руководство по созданию Blueprints для Home Assistant

## Содержание
1. [Что такое Blueprint](#что-такое-blueprint)
2. [Структура Blueprint](#структура-blueprint)
3. [Conversation triggers и голосовые ответы](#conversation-triggers-и-голосовые-ответы)
4. [Intent Script - правильный подход](#intent-script---правильный-подход)
5. [Селекторы (Selectors)](#селекторы-selectors)
6. [Примеры](#примеры)

---

## Что такое Blueprint

Blueprint - это переиспользуемая конфигурация автоматизации, скрипта или шаблона, которую можно легко импортировать и настроить через UI.

### Основные преимущества:
- Пользователи могут импортировать blueprint по URL
- Настройка через UI вместо редактирования YAML
- Автоматические обновления при изменении исходного blueprint
- Обратная совместимость при обновлении

---

## Структура Blueprint

### Минимальная структура

```yaml
blueprint:
  name: Название вашего blueprint
  description: Подробное описание (поддерживает Markdown)
  domain: automation  # или script, или template
  input:
    # Здесь определяются настраиваемые параметры
```

### Полная структура с метаданными

```yaml
blueprint:
  name: Название
  description: |
    Подробное описание с поддержкой Markdown.
    Описание должно объяснять:
    - Что делает blueprint
    - Какие входные параметры требуются
    - Примеры использования
  domain: automation
  author: Ваше имя
  homeassistant:
    min_version: 2024.4.0  # Минимальная версия HA
  input:
    my_input:
      name: Название параметра
      description: Описание параметра
      selector:
        entity:
          domain: light
      default: light.living_room

# После секции blueprint идёт обычная конфигурация automation/script
trigger:
  # ваши триггеры

action:
  # ваши действия
```

### Использование входных параметров

```yaml
blueprint:
  input:
    weather_entity:
      name: Погодная сущность
      selector:
        entity:
          domain: weather

action:
  - service: weather.get_forecasts
    target:
      entity_id: !input weather_entity  # Используем !input для доступа
    data:
      type: daily
    response_variable: forecasts
```

---

## Conversation triggers и голосовые ответы

### ⚠️ ВАЖНО: Известная проблема с Conversation Triggers в Automations

**Автоматизации с conversation triggers имеют баги с голосовыми ответами!**

Проблемы:
- `set_conversation_response` не работает корректно
- `stop` возвращает только "Done"/"Готово"
- `conversation.process` не предназначен для ответа пользователю

Это [известные баги](https://github.com/home-assistant/core/issues/109285) в Home Assistant.

### ❌ НЕ РАБОТАЕТ (автоматизация с conversation trigger):

```yaml
blueprint:
  domain: automation

trigger:
  - platform: conversation
    command:
      - "какая погода"

action:
  - service: weather.get_forecasts
    # ...
  - set_conversation_response: "{{ response }}"  # НЕ РАБОТАЕТ!
```

---

## Intent Script - правильный подход

### ✅ ПРАВИЛЬНО: Используйте Intent Script

Для голосовых команд используйте **Intent Script** + **Custom Sentences** вместо blueprint!

### Структура Intent Script

```yaml
# configuration.yaml или custom_sentences/ru/weather.yaml
conversation:
  intents:
    GetWeather:
      - "какая погода [сегодня]"
      - "какая погода {time_period}"
      - "погода на {time_period}"

intent_script:
  GetWeather:
    description: Возвращает прогноз погоды
    action:
      - service: weather.get_forecasts
        target:
          entity_id: weather.home
        data:
          type: daily
        response_variable: forecasts
      - variables:
          response: >-
            {% set forecast = forecasts['weather.home'].forecast[0] %}
            Сегодня {{ forecast.temperature }}°, {{ forecast.condition }}
    speech:
      type: plain
      text: "{{ response }}"
```

### Использование action_response

Если нужны данные из выполненного действия:

```yaml
intent_script:
  GetWeather:
    action:
      - service: weather.get_forecasts
        target:
          entity_id: weather.home
        data:
          type: daily
        response_variable: result
    speech:
      text: >-
        {% set forecast = action_response['weather.home'].forecast[0] %}
        Температура {{ forecast.temperature }} градусов
```

### Custom Sentences синтаксис

```yaml
# config/custom_sentences/ru/weather.yaml
language: "ru"
intents:
  GetWeather:
    data:
      - sentences:
          - "какая погода [сегодня]"
          - "какая погода {time_period}"
          - "погода на {time_period}"
        slots:
          time_period:
            - "завтра"
            - "послезавтра"
            - "понедельник"

# Затем в intent_script можно использовать slots
intent_script:
  GetWeather:
    action:
      - variables:
          period: "{{ trigger.slots.time_period }}"
    speech:
      text: "Погода на {{ period }}: ..."
```

---

## Селекторы (Selectors)

Селекторы определяют как входные параметры отображаются в UI.

### Основные типы селекторов

#### Entity Selector - выбор сущности

```yaml
input:
  weather_entity:
    name: Погодная сущность
    selector:
      entity:
        domain: weather  # Показывать только weather сущности
```

#### Multiple entities - несколько сущностей

```yaml
input:
  lights:
    name: Светильники
    selector:
      entity:
        domain: light
        multiple: true
```

#### Target Selector - выбор устройств/сущностей/областей

```yaml
input:
  target_lights:
    name: Целевые светильники
    selector:
      target:
        entity:
          domain: light
```

#### Number Selector - числовой ввод

```yaml
input:
  delay_seconds:
    name: Задержка (секунды)
    default: 30
    selector:
      number:
        min: 0
        max: 300
        step: 10
        unit_of_measurement: "s"
```

#### Duration Selector - продолжительность

```yaml
input:
  wait_time:
    name: Время ожидания
    default:
      minutes: 5
    selector:
      duration:
```

#### Time Selector - время

```yaml
input:
  notify_time:
    name: Время уведомления
    default: "07:00:00"
    selector:
      time: {}
```

#### Text Selector - текстовое поле

```yaml
input:
  message:
    name: Сообщение
    selector:
      text:
        multiline: true  # Многострочное поле
```

#### Boolean Selector - чекбокс

```yaml
input:
  enable_notifications:
    name: Включить уведомления
    default: true
    selector:
      boolean: {}
```

#### Select Selector - выпадающий список

```yaml
input:
  weather_type:
    name: Тип прогноза
    default: daily
    selector:
      select:
        options:
          - label: Ежедневный
            value: daily
          - label: Почасовой
            value: hourly
```

#### Conversation Agent Selector

```yaml
input:
  conversation_agent:
    name: Conversation Agent
    selector:
      conversation_agent: {}
```

### Полный список селекторов

- `action` - выбор действия
- `addon` - дополнение
- `area` - область
- `attribute` - атрибут сущности  
- `backup_location` - расположение бэкапа
- `boolean` - да/нет
- `color_rgb` - RGB цвет
- `color_temp` - цветовая температура
- `condition` - условие
- `conversation_agent` - conversation agent
- `country` - страна
- `date` - дата
- `datetime` - дата и время
- `device` - устройство
- `duration` - продолжительность
- `entity` - сущность
- `file` - файл
- `floor` - этаж
- `icon` - иконка
- `language` - язык
- `location` - местоположение
- `media` - медиа
- `number` - число
- `object` - объект
- `select` - выпадающий список
- `state` - состояние
- `target` - цель (entity/device/area)
- `template` - шаблон
- `text` - текст
- `theme` - тема
- `time` - время
- `trigger` - триггер

[Полная документация по селекторам](https://www.home-assistant.io/docs/blueprint/selectors/)

---

## Примеры

### Пример 1: Blueprint для уведомления по расписанию

```yaml
blueprint:
  name: Ежедневное уведомление о погоде
  description: Отправляет уведомление с прогнозом погоды в заданное время
  domain: automation
  homeassistant:
    min_version: 2024.4.0
  input:
    weather_entity:
      name: Погодная сущность
      selector:
        entity:
          domain: weather
    notify_time:
      name: Время уведомления
      default: "07:00:00"
      selector:
        time: {}
    notify_service:
      name: Notify service
      default: notify.notify
      selector:
        text: {}

trigger:
  - platform: time
    at: !input notify_time

action:
  - service: weather.get_forecasts
    target:
      entity_id: !input weather_entity
    data:
      type: daily
    response_variable: forecasts
  - variables:
      weather_entity: !input weather_entity
      forecast: "{{ forecasts[weather_entity].forecast[0] }}"
      message: >-
        Прогноз погоды на сегодня:
        Температура: {{ forecast.temperature }}°
        Условия: {{ forecast.condition }}
  - service: !input notify_service
    data:
      message: "{{ message }}"
```

### Пример 2: Intent Script для голосовых команд (НЕ blueprint!)

```yaml
# custom_sentences/ru/weather.yaml
language: "ru"
intents:
  GetWeatherForecast:
    data:
      - sentences:
          - "какая погода [сегодня]"
          - "какая погода [на] {day}"
          - "прогноз погоды [на] {day}"

# configuration.yaml
intent_script:
  GetWeatherForecast:
    description: Голосовой прогноз погоды на русском
    action:
      - service: weather.get_forecasts
        target:
          entity_id: weather.home
        data:
          type: daily
        response_variable: forecasts
      - variables:
          day: "{{ trigger.slots.day | default('сегодня') }}"
          forecast: "{{ forecasts['weather.home'].forecast[0] }}"
          temp: "{{ forecast.temperature | round(0) }}"
          condition_map:
            clear: ясно
            cloudy: облачно
            rainy: дождь
            sunny: солнечно
          condition: "{{ condition_map.get(forecast.condition, forecast.condition) }}"
    speech:
      type: plain
      text: "{{ day | capitalize }}: {{ temp }}°, {{ condition }}"
```

### Пример 3: Blueprint со секциями (HA 2024.6+)

```yaml
blueprint:
  name: Продвинутые уведомления
  description: Blueprint с группировкой параметров
  domain: automation
  homeassistant:
    min_version: 2024.6.0
  input:
    notify_time:
      name: Время уведомления
      selector:
        time: {}
    
    weather_section:
      name: Настройки погоды
      icon: mdi:weather-partly-cloudy
      collapsed: false
      input:
        weather_entity:
          name: Погодная сущность
          selector:
            entity:
              domain: weather
        include_forecast:
          name: Включить прогноз
          default: true
          selector:
            boolean: {}
    
    notification_section:
      name: Настройки уведомлений
      icon: mdi:bell
      collapsed: true
      input:
        notify_service:
          name: Notify service
          default: notify.notify
          selector:
            text: {}

trigger:
  - platform: time
    at: !input notify_time

action:
  - choose:
      - conditions:
          - "{{ include_forecast }}"
        sequence:
          # действия с прогнозом
      default:
        # действия без прогноза
```

---

## Рекомендации

### Обратная совместимость

При обновлении blueprint:
- ❌ **НЕ** меняйте имена существующих входных параметров
- ❌ **НЕ** удаляйте существующие параметры
- ✅ Новые параметры **ДОЛЖНЫ** иметь значения по умолчанию
- ✅ Используйте `min_version` для новых функций

### Лучшие практики

1. **Описание**: Пишите подробные описания с примерами
2. **Значения по умолчанию**: Указывайте разумные значения по умолчанию
3. **Селекторы**: Используйте правильные селекторы для каждого типа данных
4. **Фильтры**: Используйте фильтры в entity селекторах (domain, device_class)
5. **Версия**: Указывайте `min_version` если используете новые функции
6. **Тестирование**: Тестируйте blueprint перед публикацией

### Публикация

Для публикации в Home Assistant Community:
1. Создайте топик с blueprint на форуме
2. Добавьте import badge
3. Включите полный YAML код в код-блок
4. Добавьте релевантные теги
5. Проверьте импорт по URL

---

## Заключение

**Для голосовых команд:**
- Используйте **Intent Script** + **Custom Sentences**
- НЕ используйте automations с conversation triggers (есть баги)

**Для blueprint:**
- Используйте для автоматизаций по времени, событиям, состояниям
- Добавляйте подробные описания
- Используйте правильные селекторы
- Обеспечьте обратную совместимость

---

## Полезные ссылки

- [Blueprint Schema](https://www.home-assistant.io/docs/blueprint/schema/)
- [Blueprint Selectors](https://www.home-assistant.io/docs/blueprint/selectors/)
- [Intent Script](https://www.home-assistant.io/integrations/intent_script/)
- [Custom Sentences](https://www.home-assistant.io/voice_control/custom_sentences_yaml/)
- [Conversation API](https://developers.home-assistant.io/docs/intent_conversation_api/)
- [Blueprint Community](https://community.home-assistant.io/c/blueprints-exchange)
