# API и URL-паттерны

Единые правила адресов для фронтенда (SPA) и backend (REST API).  
Цель — простые понятные URL в интерфейсе, стабильные пермалинки для шаринга и строгая модель адресации в API.

---

## Базовые домены и URL

- **Лэндинг (публичный сайт):** `https://piket.pro/`
- **SPA-приложение (клиент):** `https://app.piket.pro/`
- **REST API (backend):** `https://api.piket.pro/v1/`

> Во всех примерах ниже префикс **/v1** опущен для краткости.

---

## Конвенции

### Идентификаторы
- В API все `{id}` — UUID.
- В пермалинках SPA используются **slug** (читаемые идентификаторы), где это возможно.

### Время и форматы
- Временные значения — ISO-8601 в UTC, например `2025-08-15T10:00:00Z`.
- Ответы — JSON; ошибки — стандартизованный объект (см. ниже).

---

## SPA (Frontend) URL-паттерны

### Принцип адресации
1. Пользователь видит **короткие** адреса верхнего уровня — без вложенных идентификаторов.
2. **Контекст** (активная организация и активный объект) хранится на клиенте (`sessionStorage` вкладки).  
   Если контекст отсутствует, выполняется редирект на выбор соответствующей сущности.
3. Для шаринга и прямых ссылок существуют **пермалинки** (отдельные «глубокие» пути) со `slug`. Они **не** используются в основном меню.

### Короткие (видимые) маршруты
- `/login` — вход
- `/registration` — регистрация
- `/orgs` — список организаций
- `/objects` — список объектов **активной** организации  
  → если активная организация не выбрана — редирект на `/orgs`
- `/files` — файлы **активного** объекта
- `/works` — журнал работ **активного** объекта
- `/areas` — участки **активного** объекта
- `/acts` — акты **активного** объекта
- `/tomes` — тома **активного** объекта
- `/analytics` — аналитика **активной** организации
- `/warehouses` — склады **активной** организации

**Всплывающие модули** (отдельного маршрута не имеют):  
«Планы», «Доступ», «Просмотр». Допускается отражать состояние модалки в query:  
`?modal=plans`, `?modal=access&entity=object&id=...`, `?modal=viewer&source=acts&selection=...`.

### Пермалинки (для шаринга)
- `/o/:orgSlug` — дашборд организации
- `/o/:orgSlug/objects/:objSlug` — объект
- `/o/:orgSlug/objects/:objSlug/acts/:actNumber` — акт
- `/o/:orgSlug/objects/:objSlug/tomes/:tomeNumber` — том

Если slug недействителен или нет прав доступа — страница «Нет доступа»/404 без раскрытия внутренних UUID.

### Редиректы при отсутствии контекста
- Страницы уровня **объекта** без активного объекта → `/objects` (подсказка «Выберите объект»).
- Страницы уровня **организации** без активной организации → `/orgs`.

---

## REST API (Backend)

### Общие принципы
- **Контекст организации передаётся только в URL**: `/orgs/{orgId}/…`.  
  Заголовки `X-Org-Id` и query `orgId` **не используются**.
- Вложенные ресурсы адресуются через родителя: `/orgs/{orgId}/objects/{objectId}/…`.
- Версионирование — префикс `/v1`.
- CORS: разрешён `https://app.piket.pro`.
- Отсутствуют эндпоинты экспорта печатных форм (`/export/pdf|docx` не реализуются).

### Аутентификация и сессии
- HttpOnly-cookies (`Secure`, `SameSite=Lax`), короткоживущий access-токен и refresh-токен с ротацией.
- Смена пароля/выход инвалидируют все активные сессии.

**Эндпоинты**
- `POST /auth/login` — вход (email/пароль), выдаёт cookies и `{ user }`
- `POST /auth/refresh` — обновление токенов (по refresh-cookie)
- `POST /auth/logout` — выход (инвалидация refresh, очистка cookies)
- `GET  /auth/me` — текущий пользователь

### Организации
- `GET  /orgs` — список доступных организаций
- `POST /orgs` — создать
- `GET  /orgs/{orgId}` — данные организации
- `PATCH /orgs/{orgId}` — обновить
- `DELETE /orgs/{orgId}` — архивировать/удалить
- `GET  /orgs/{orgId}/access` — права на организацию

### Объекты (внутри организации)
- `GET  /orgs/{orgId}/objects` — список объектов
- `POST /orgs/{orgId}/objects` — создать объект
- `GET  /orgs/{orgId}/objects/{objectId}` — данные объекта
- `PATCH /orgs/{orgId}/objects/{objectId}` — обновить
- `DELETE /orgs/{orgId}/objects/{objectId}` — архив/удаление

### Файлы (объект)
- `GET  /orgs/{orgId}/objects/{objectId}/files` — список файлов объекта
- `POST /orgs/{orgId}/objects/{objectId}/files` — загрузить файл
- `GET  /orgs/{orgId}/files/{fileId}` — метаданные файла
- `PATCH /orgs/{orgId}/files/{fileId}` — переименовать/переместить
- `DELETE /orgs/{orgId}/files/{fileId}` — удалить (в корзину)
- `GET  /orgs/{orgId}/files/{fileId}/download` — скачать

### Журнал работ и участки (объект)
- `GET  /orgs/{orgId}/objects/{objectId}/journal` — список записей
- `POST /orgs/{orgId}/objects/{objectId}/journal` — добавить запись
- `GET  /orgs/{orgId}/objects/{objectId}/journal/{entryId}` — запись
- `PATCH /orgs/{orgId}/objects/{objectId}/journal/{entryId}` — обновить
- `DELETE /orgs/{orgId}/objects/{objectId}/journal/{entryId}` — удалить (мягко)

- `GET  /orgs/{orgId}/objects/{objectId}/areas` — список участков
- `POST /orgs/{orgId}/objects/{objectId}/areas` — создать участок
- `GET  /orgs/{orgId}/objects/{objectId}/areas/{areaId}` — участок
- `PATCH /orgs/{orgId}/objects/{objectId}/areas/{areaId}` — обновить
- `DELETE /orgs/{orgId}/objects/{objectId}/areas/{areaId}` — удалить (мягко)

### Акты и тома (объект)
- `GET  /orgs/{orgId}/objects/{objectId}/acts` — список актов
- `POST /orgs/{orgId}/objects/{objectId}/acts` — создать акт
- `GET  /orgs/{orgId}/objects/{objectId}/acts/{actId}` — акт
- `PATCH /orgs/{orgId}/objects/{objectId}/acts/{actId}` — обновить/сменить статус

- `GET  /orgs/{orgId}/objects/{objectId}/tomes` — список томов
- `POST /orgs/{orgId}/objects/{objectId}/tomes` — создать том
- `GET  /orgs/{orgId}/objects/{objectId}/tomes/{tomeId}` — том
- `PATCH /orgs/{orgId}/objects/{objectId}/tomes/{tomeId}` — обновить/сменить статус

### Тарифные планы (организация)
- `GET  /orgs/{orgId}/plans` — список тарифов
- `GET  /orgs/{orgId}/plan` — текущий тариф
- `POST /orgs/{orgId}/plan` — смена тарифа
- `GET  /orgs/{orgId}/payments` — история оплат

### Аналитика
- `GET  /orgs/{orgId}/analytics` — свод по организации
- `GET  /orgs/{orgId}/objects/{objectId}/analytics` — свод по объекту (зарезервировано)

### Доступы (ACL)
- `GET  /access/{entityType}/{entityId}` — текущие права на сущность
- `POST /access/{entityType}/{entityId}` — изменить права

---

## Пагинация, сортировка, фильтрация

- **Пагинация:** `?page=1&pageSize=25` (дефолт `pageSize=25`, максимум 100).  
  Ответ коллекции включает мета-блок:
  ```json
  { "data": [...], "meta": { "page": 1, "pageSize": 25, "total": 123 } }
