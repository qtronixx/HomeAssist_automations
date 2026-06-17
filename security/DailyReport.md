## 📝 **"Security Daily Summary Professional"**

# 📊 Ежедневный отчет безопасности с AI-анализом

Автоматизация собирает события с камер за последние 24 часа, отправляет их в Google AI для анализа и присылает краткий отчет в Telegram.

## 🔧 Зависимости

Для работы автоматизации необходимы:

| Компонент | Описание | Где находится |
|-----------|----------|---------------|
| `sensor.frigate_daily_report_data` | Сенсор с данными о событиях | `sensor.yaml` |
| `conversation.google_ai_conversation` | Интеграция Google AI | Настройки → Интеграции |
| Telegram bot | Для отправки отчета | Настройки → Интеграции |

## 📁 Файлы

- `security_daily_summary.yaml` — основная автоматизация
- `sensor.yaml` — конфигурация command_line сенсора для сбора данных из Frigate

## 🤖 Как это работает

1. **Каждый час** command_line сенсор запрашивает у Frigate события за последние 24 часа
2. **В 23:55** автоматизация принудительно обновляет сенсор
3. Данные отправляются в Google AI с промптом:
   > "Ты — начальник службы безопасности. Проанализируй логи событий..."
4. AI анализирует активность и составляет краткий отчет
5. Отчет отправляется в Telegram

## 📊 Пример отчета


📊 ИТОГИ ЗА СУТКИ

За прошедшие сутки на участке было спокойно. 
Зафиксировано 3 события:
- [road_gate]: автомобиль проехал мимо калитки в 14:23
- [doorbell]: человек подходил к калитке в 19:45
- [on_vagon]: кошка прогулялась по участку в 03:12

Подозрительной активности не обнаружено.


## ⚙️ Настройка

### 1. Сенсор (уже в `sensor.yaml`)
```yaml
command_line:
  - sensor:
      name: Frigate Daily Report Data
      unique_id: frigate_daily_report_data
      command: >
        curl -s "http://xxx.xxx.xxx.xxx:5000/api/events?after={{ (as_timestamp(now()) - 86400) | int }}&limit=50" | 
        jq -c '{events: [.[] | select(.data != null and .data.description != null) | {camera: .camera, desc: .data.description}]}'
      scan_interval: 3600
      value_template: "{{ value_json.events | length }}"
      json_attributes:
        - events
```

### 2. Интеграция Google AI
- Добавьте через **Настройки → Устройства и службы → Добавить интеграцию → Google AI**
- Получите API ключ в [Google AI Studio](https://aistudio.google.com/)
- Назовите агента `google_ai_conversation` (или измените в автоматизации)

### 3. Telegram
- Убедитесь, что `chat_id: -xxxxxxxxxx` соответствует вашему чату
- При необходимости замените на другой ID

## 🎯 Кастомизация

### Изменить время отчета
В триггере поменяйте `at: "23:55:00"` на нужное время.

### Изменить промпт для AI
Отредактируйте текст в поле `text:` — можно уточнить стиль, язык, формат отчета.

### Фильтровать камеры
Добавьте условие в jq-запрос, например:
```bash
select(.camera | in(["doorbell", "road_gate"]))
```

## 🐛 Отладка

Проверьте работу сенсора вручную:
```bash
# В терминале HA
curl -s "http://xxx.xxx.xxx.xxx:5000/api/events?after=$(date -d '24 hours ago' +%s)&limit=50" | jq
```

В Home Assistant проверьте атрибуты сенсора:
**Инструменты разработчика → Состояния → sensor.frigate_daily_report_data**

## 📌 Примечания

- Автоматизация использует **chat_id** вместо устаревшего target (совместимо с HA 2026.9+)
- Для работы необходим доступ к Frigate API по адресу `xxx.xxx.xxx.xxx:5000`
- События без описания (description) игнорируются
