# API и URL-паттерны

Черновик базовых маршрутов SPA и REST API для системы **Piket.pro**.  
Документ служит ориентиром для фронтенд- и бэкенд-разработчиков, чтобы формировать единый стиль адресов.

## SPA (Frontend) URL-паттерны

### Общие страницы
- `/login` — вход в систему.
- `/registration` — регистрация.
- `/organizations` — список организаций.
- `/organizations/:orgId` — активная организация (переход в последний открытый раздел или в «Объекты» по умолчанию).
- `/objects` — список объектов активной организации.
- `/objects/:objectId/files` — файлы объекта.
- `/objects/:objectId/journal` — журнал работ.
- `/objects/:objectId/acts` — акты объекта.
- `/plans` — тарифные планы активной организации.
- `/access` — управление доступом к организации или объекту.
- `/analytics` — аналитика организации.

> Примечание: SPA будет работать на домене `app.piket.pro`, а публичный лендинг — на `piket.pro`.

---

## REST API (Backend)

### Аутентификация
- `POST /api/auth/login` — вход (возвращает access/refresh токены).
- `POST /api/auth/refresh` — обновление access токена.
- `POST /api/auth/logout` — выход (инвалидировать refresh токен).

### Пользователи
- `GET /api/users/me` — профиль текущего пользователя.
- `PATCH /api/users/me` — обновление своего профиля.
- `GET /api/users/:id` — профиль другого пользователя (с учётом приватных полей).

### Организации
- `GET /api/organizations` — список доступных пользователю организаций.
- `POST /api/organizations` — создание новой организации.
- `GET /api/organizations/:id` — данные организации.
- `PATCH /api/organizations/:id` — обновление организации.
- `DELETE /api/organizations/:id` — удаление (перемещение в архив).
- `GET /api/organizations/:id/access` — список прав пользователей.
- `POST /api/organizations/:id/access` — изменение прав.

### Объекты
- `GET /api/organizations/:orgId/objects` — список объектов организации.
- `POST /api/organizations/:orgId/objects` — создание объекта.
- `GET /api/objects/:id` — данные объекта.
- `PATCH /api/objects/:id` — обновление объекта.
- `DELETE /api/objects/:id` — удаление (архив).
- `GET /api/objects/:id/files` — список файлов объекта.

### Файлы
- `POST /api/objects/:id/files` — загрузка файла в объект.
- `GET /api/files/:fileId` — информация о файле.
- `DELETE /api/files/:fileId` — удаление файла (в корзину).
- `PATCH /api/files/:fileId` — переименование или перемещение.
- `GET /api/files/:fileId/download` — скачивание файла.

### Тарифные планы
- `GET /api/plans` — список тарифов.
- `GET /api/organizations/:orgId/plan` — текущий тариф организации.
- `POST /api/organizations/:orgId/plan` — смена тарифа.
- `GET /api/organizations/:orgId/payments` — история оплат.

### Права доступа
- `GET /api/access/:entityType/:entityId` — текущие права пользователей на сущность.
- `POST /api/access/:entityType/:entityId` — изменение прав.

### Аналитика
- `GET /api/organizations/:orgId/analytics` — свод по организации.
- `GET /api/objects/:objectId/analytics` — (зарезервировано, не используется в MVP).

---

## Принципы именования
- Все пути в REST API начинаются с `/api/`.
- В URL всегда используется **множественное число** для коллекций (`/organizations`, `/objects`).
- Для вложенных ресурсов используется иерархия `/parent/:id/child`.
- Идентификаторы (`:id`) всегда в формате UUID (или bigint — уточнить на этапе БД).

---

## Ответы API (JSON)

### Пример списка организаций
```json
[
  {
    "id": "uuid",
    "shortName": "ООО ДорСтрой",
    "inn": "1234567890",
    "plan": "PRO",
    "objectsCount": 3,
    "storageUsed": 1024000000,
    "storageLimit": 2147483648,
    "subscriptionEnd": "2025-10-01"
  }
]
