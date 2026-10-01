# 🔧 Решение проблем при установке голосового ассистента

## 📋 Оглавление

- [Проблемы с Whisper](#проблемы-с-whisper)
- [Проблемы с Ollama](#проблемы-с-ollama)
- [Проблемы с Extended OpenAI Conversation](#проблемы-с-extended-openai-conversation)
- [Проблемы с CPU и производительностью](#проблемы-с-cpu-и-производительностью)
- [Общие проблемы](#общие-проблемы)

---

## Проблемы с Whisper

### ❌ CPU does not support AVX

**Ошибка:**
```
WARNING: Your CPU does not support Advanced Vector Extensions (AVX)
RuntimeError: NumPy was built with baseline optimizations: (X86_V2)
```

**Причина:** 
- CPU type в Proxmox установлен неправильно
- Флаги AVX не передаются в VM

**Решение:**

```bash
# В Proxmox Shell
qm stop 100  # Замените 100 на ID вашей VM
qm set 100 -cpu host
qm start 100

# Проверьте в HA Terminal
cat /proc/cpuinfo | grep avx
# Должны увидеть: avx avx2
```

**Альтернатива - Vosk (не требует AVX):**

```bash
# В Terminal HA
ha addons repository add https://github.com/rhasspy/hassio-addons
ha addons install rhasspy_vosk
```

---

### ❌ Whisper использует 8 GB RAM

**Причина:**
- Слишком большой beam_size
- Модель не base, а large/medium
- Утечка памяти

**Решение:**

```yaml
# Settings → Add-ons → Whisper → Configuration
language: ru
model: tiny  # Или base
beam_size: 1  # Уменьшите!
```

**Перезапустите add-on**

---

### ❌ Whisper медленно распознает

**Решение 1 - Оптимизируйте настройки:**

```yaml
language: ru
model: tiny  # Самая быстрая модель
beam_size: 1
```

**Решение 2 - Добавьте CPU cores:**

```bash
# В Proxmox Shell
qm stop 100
qm set 100 -cores 4
qm start 100
```

---

## Проблемы с Ollama

### ❌ Скрипт установки выдал ошибку Intel Graphics

**Ошибка:**
```
✖️ exit code 2: gpg --dearmor -o /usr/share/keyrings/intel-graphics.gpg
```

**Решение:**

Контейнер создан, доустановите Ollama вручную:

```bash
# Войдите в контейнер
pct enter 101

# Установите Ollama
curl -fsSL https://ollama.com/install.sh | sh
systemctl start ollama
systemctl enable ollama

# Проверьте
systemctl status ollama
```

✅ **Готово!** Ошибка не критична.

---

### ❌ Ollama не отвечает

**Проверка:**

```bash
# В Proxmox Shell
pct status 101

# Войдите в контейнер
pct enter 101

# Проверьте сервис
systemctl status ollama

# Проверьте API
curl http://localhost:11434/api/tags
```

**Решение:**

```bash
# Перезапустите сервис
systemctl restart ollama

# Проверьте логи
journalctl -u ollama -f

# Если модель не загружена
ollama pull llama3.2:3b-instruct-q4_K_M
```

---

### ❌ Ollama медленно отвечает

**Причина:** Модель слишком большая для CPU

**Решение - Используйте меньшую модель:**

```bash
pct enter 101

# Удалите большую модель
ollama rm qwen2.5:7b-instruct-q4_K_M

# Установите компактную
ollama pull llama3.2:3b-instruct-q4_K_M

# Или совсем маленькую
ollama pull llama3.2:1b-instruct-q4_K_M

exit
```

**Обновите модель в Extended OpenAI Conversation:**
```
Settings → Devices & Services → Extended OpenAI Conversation → Configure
Model: llama3.2:3b-instruct-q4_K_M
```

---

### ❌ Out of Memory в Ollama

**Ошибка:**
```
model requires 2.3 GiB, available 1.4 GiB
```

**Причина:** В LXC контейнерах Ollama может неправильно определять доступную память через cgroup v2.

**Решение 1 - Увеличьте swap (РЕКОМЕНДУЕТСЯ):** ⭐

```bash
# В Proxmox Shell
pct set 101 -swap 4096  # 4 GB swap
pct reboot 101

# Подождите 30 секунд
sleep 30

# Проверьте
pct exec 101 -- free -h
pct exec 101 -- ollama run llama3.2:3b-instruct-q4_K_M "Привет"
```

**Решение 2 - Увеличьте RAM:**

```bash
# В Proxmox Shell
pct stop 101
pct set 101 -memory 6144  # Увеличьте до 6 GB
pct start 101
```

**Решение 3 - Используйте модель 1b:**

```bash
pct exec 101 -- ollama pull llama3.2:1b-instruct-q4_K_M
```

Измените модель в Extended OpenAI Conversation на `llama3.2:1b-instruct-q4_K_M`

---

## Проблемы с Extended OpenAI Conversation

### ❌ Cannot connect to Ollama

**Ошибка:** `Connection refused` или `Timeout`

**Проверка:**

```bash
# Узнайте IP Ollama
pct enter 101
ip addr show eth0 | grep "inet "
exit

# Проверьте доступность из HA
# В Terminal HA:
ping 192.168.1.50  # Замените на ваш IP
curl http://192.168.1.50:11434/api/tags
```

**Решение:**

1. Убедитесь что Ollama запущен:
   ```bash
   pct enter 101
   systemctl status ollama
   ```

2. Проверьте IP адрес в интеграции:
   ```
   Settings → Devices & Services → Extended OpenAI Conversation → Configure
   API Base URL: http://ПРАВИЛЬНЫЙ_IP:11434/v1
   ```

---

### ❌ Ассистент не понимает команды

**Причина:** Неправильный промпт

**Решение - Оптимизируйте промпт:**

```
Settings → Devices & Services → Extended OpenAI Conversation → Configure → Options → Prompt
```

Используйте:

```
Ты голосовой ассистент умного дома на русском языке.

ПРАВИЛА:
1. Отвечай КРАТКО - максимум 1-2 предложения
2. Используй естественный русский язык
3. Всегда подтверждай действия
4. Для управления устройствами используй доступные сервисы

ПРИМЕРЫ:
Пользователь: "Включи свет"
Ты: "Включил свет"

Пользователь: "Какая погода?"
Ты: "Сейчас 15 градусов и ясно"
```

---

### ❌ Слишком длинные ответы

**Решение:**

```
Settings → Devices & Services → Extended OpenAI Conversation → Configure
Max Tokens: 100  # Уменьшите
Temperature: 0.5  # Уменьшите для более точных ответов
```

---

## Проблемы с CPU и производительностью

### ❌ Высокая нагрузка на CPU

**Проверка:**

```bash
# В Terminal HA
top
# Смотрите какой процесс нагружает
```

**Решение:**

1. **Whisper слишком тяжелый:**
   ```yaml
   model: tiny
   beam_size: 1
   ```

2. **Ollama модель большая:**
   ```bash
   # Используйте 1b или 3b модель
   ollama pull llama3.2:1b-instruct-q4_K_M
   ```

3. **Добавьте CPU cores:**
   ```bash
   qm set 100 -cores 4
   ```

---

### ❌ VM медленно работает

**Решение - Оптимизация VM:**

```bash
# В Proxmox Shell

# 1. Включите host CPU
qm stop 100
qm set 100 -cpu host

# 2. Добавьте cores
qm set 100 -cores 4

# 3. Добавьте RAM
qm set 100 -memory 6144

# 4. Включите balloon (динамическая память)
qm set 100 -balloon 4096

# 5. Запустите
qm start 100
```

---

## Общие проблемы

### ❌ Нет раздела Add-ons в Home Assistant

**Причина:** У вас не HAOS, а Core/Container

**Решение:** Используйте другие гайды:
- [QUICK_START.md](QUICK_START.md)
- [INSTALL_GUIDE.md](INSTALL_GUIDE.md)

**Или:** Переустановите на HAOS

---

### ❌ Не хватает памяти на хосте

**Проверка:**

```bash
# В Proxmox Shell
free -h
```

**Решение:**

1. **Оптимизируйте распределение:**
   - HAOS: 4 GB (минимум)
   - Ollama: 4 GB (минимум)
   - Proxmox: 2 GB

2. **Используйте облегченные модели:**
   - Whisper: `tiny`
   - Ollama: `llama3.2:1b`

3. **Используйте облачные сервисы:**
   - Google Cloud Speech-to-Text
   - Yandex SpeechKit

---

### ❌ Ассистент говорит "Готово" вместо ответа

**Причина:** Неправильная настройка conversation

**Решение:**

Убедитесь что в `configuration.yaml`:

```yaml
assist_pipeline:

conversation:
  intents:
    HassTurnOn:
    HassTurnOff:
    HassLightSet:
```

И перезапустите HA.

---

### ❌ HACS не устанавливается (wget not found)

**Решение - Альтернативный метод:**

```bash
# В Terminal HA
cd /config/custom_components
apk add git
git clone https://github.com/hacs/integration.git hacs
```

Перезапустите HA.

---

### ❌ File Editor показывает ошибки в YAML

**Проверка синтаксиса:**

```
Developer Tools → YAML → Check Configuration
```

**Частые ошибки:**
- Неправильные отступы (используйте пробелы, не табы!)
- Незакрытые кавычки
- Лишние пробелы в конце строк

---

## 🆘 Дополнительная помощь

### Логи для диагностики

**Home Assistant:**
```
Settings → System → Logs
```

**Whisper:**
```
Settings → Add-ons → Whisper → Logs
```

**Ollama:**
```bash
pct enter 101
journalctl -u ollama -f
```

**Extended OpenAI Conversation:**
```
Settings → System → Logs
Фильтр: extended_openai_conversation
```

---

### Полезные команды

**Проверка ресурсов в HA:**
```bash
ha host info
free -h
top
```

**Управление Ollama:**
```bash
pct enter 101
ollama list           # Список моделей
ollama ps             # Запущенные модели
systemctl status ollama
journalctl -u ollama
```

**Управление VM:**
```bash
qm list               # Список VM
qm status 100         # Статус VM
qm config 100         # Конфигурация
```

**Управление контейнерами:**
```bash
pct list              # Список LXC
pct status 101        # Статус контейнера
pct config 101        # Конфигурация
```

---

## 📚 Полезные ссылки

- [Home Assistant Voice Docs](https://www.home-assistant.io/voice_control/)
- [Ollama Models](https://ollama.com/library)
- [Proxmox VE Scripts](https://community-scripts.github.io/ProxmoxVE/)
- [HACS Documentation](https://hacs.xyz/)

---

**💡 Не нашли решение?** Откройте issue в репозитории с подробным описанием проблемы и логами!
