# n8n-Telegram-Assistant

Готовый workflow для n8n: Личный Telegram Ассистент с интеграциями Todoist, Google Calendar, Notion, AI и PostgreSQL.

## Возможности
- Управление задачами (Todoist): создание с приоритетами/сроками/метками, список актуальных задач
- Календарь (Google Calendar): создание событий, расписание на день/неделю, напоминания
- Заметки (Notion): создание структурированных заметок, категории, поиск (в расширениях)
- AI-обработка (опционально): классификация намерений, извлечение параметров
- Напоминания: хранение в PostgreSQL, проверка каждые 5 минут (в расширениях), отправка в срок
- Автоотчеты (расширяемо): утренний брифинг 8:00, вечерний отчет 20:00, недельная статистика
- Команды бота: /start, /tasks, /today, /add_task, /add_event, /note, /search, /stats, /remind

## Файл workflow
В репозитории добавлен файл: n8n-telegram-assistant-workflow.json — импортируйте его в n8n.

## Импорт workflow в n8n
1) В n8n откройте Workflows → Import from File
2) Выберите n8n-telegram-assistant-workflow.json и импортируйте
3) Откройте импортированный workflow и настройте credentials (см. ниже)
4) Запустите workflow и, при необходимости, активируйте

## Настройка credentials
- Telegram Bot API: токен бота от @BotFather. В узлах Telegram/Telegram Trigger выберите эти креды
- Todoist OAuth2: создайте OAuth App на https://developer.todoist.com/, укажите Redirect URL из n8n, пройдите OAuth
- Google Calendar OAuth2: в Google Cloud Console создайте OAuth Client, включите Calendar API, укажите Redirect URL n8n
- Notion API: создайте интеграцию https://www.notion.so/my-integrations и поделитесь нужной базой с интеграцией
- PostgreSQL: укажите хост/порт/БД/пользователь/пароль. Таблица reminders создается автоматически
- OpenAI API (опционально): укажите ключ, если добавляете AI-ноды для расширенной обработки

## Переменные окружения
- TELEGRAM_CHAT_ID — ваш Telegram ID (для ответов и подтверждений). Можно не задавать: будет использоваться message.chat.id
- NOTION_DATABASE_ID — ID базы в Notion для заметок

## Структура workflow (ключевые узлы)
- Telegram Trigger — принимает сообщения и callback_query
- Классификация намерения (Function) — определяет intent по командам
- IF-блоки по intent: start, task_create, event_create, note_create, reminder_create
- Парсеры параметров (Function) для /add_task, /add_event, /note, /remind
- Todoist (create) — создание задач с приоритетом/дедлайном/метками
- Google Calendar (create) — событие с временем/локацией/описанием и уведомлениями
- Notion (append) — добавление страницы в указанную базу (Content, Category)
- PostgreSQL (init/insert) — создание таблицы reminders и вставка напоминаний
- Telegram (send) — подтверждения пользователю

Связи: Telegram Trigger → Классификация → ветки IF → парсеры → интеграции → подтверждения.

## Примеры команд
- /start — подсказка по возможностям
- /tasks — список задач (реализуйте отдельной веткой: Todoist get → Telegram send)
- /today — план на сегодня (Calendar list events today → Telegram send)
- /add_task Текст задачи | p:4 | due:2025-10-20 | proj:123 | labels:work,urgent
- /add_event Заголовок | 2025-10-20 10:00 | 2025-10-20 11:00 | loc:Москва | desc:Встреча
- /note Заголовок | текст заметки | cat:Идеи
- /search запрос — поиск в заметках (добавьте Notion search и форматирование)
- /stats — недельная статистика (добавьте агрегирующие узлы)
- /remind Позвонить маме в 18:30 или через 2h

## Детализация интеграций
- Todoist
  - Узел: Todoist (create, OAuth2)
  - Параметры: projectId, content; additional: dueDate, priority (1–4), labels, description
  - Рекомендации: хранить соответствие проектов/меток в KV/Static Data; для списка задач используйте operation=list
- Google Calendar
  - Узел: Google Calendar (create, OAuth2), calendarId=primary
  - Параметры: start, end, summary; additional: description, location, sendNotifications=true
  - Рекомендации: добавьте ветку «/today» с operation=getAll и фильтром по дате
- Notion
  - Узел: Notion (append). databaseId = NOTION_DATABASE_ID
  - Свойства: Title — из note_title; Content — rich text; Category — select
  - Рекомендации: заведите предопределенные селекты в базе
- AI (опционально)
  - Добавьте OpenAI или LangChain-ноды для: классификации intent, извлечения дат/приоритетов/проектов
  - Держите fallback на простые регэкспы и Function-ноды (как в текущей версии)
- PostgreSQL
  - Таблица reminders: id, chat_id, text, run_at, done
  - Планировщик: отдельный workflow с Cron (каждые 5 минут) → SELECT due reminders → Telegram send → UPDATE done=true

## Рекомендации по расширению
- Google Sheets: логирование действий / бриффинги
- Spotify: управление музыкой по командам
- GitHub: оповещения о коммитах/PR
- Финансы: запись расходов в таблицу/Notion
- Умный дом: отправка команд через webhook/HTTP
- CRM: создание лидов/сделок по командам
- Аналитика: недельные дайджесты с графиками (через внешние сервисы)

## Примечания по безопасности
- Не логируйте чувствительные токены в Telegram
- Ограничьте доступ бота по chat_id (сравнение с разрешенным списком)
- Храните секреты в credentials n8n или в переменных окружения, а не в Function

---
Готово к импорту: используйте файл n8n-telegram-assistant-workflow.json, настройте credentials и активируйте workflow.
