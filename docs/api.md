# API и URL-паттерны

Единые правила адресов для фронтенда (SPA) и backend (REST API).  
Цель — простые понятные URL в интерфейсе, стабильные короткие ссылки для «Поделиться», и строгая модель адресации в API.

---

## Базовые домены и URL

- **Лэндинг (публичный сайт):** `https://piket.pro/`
- **SPA-приложение (клиент):** `https://app.piket.pro/`
- **Короткие ссылки (редирект-домен):** `https://go.piket.pro/{token}`
- **REST API (backend):** `https://api.piket.pro/v1/`

> Во всех примерах ниже префикс **/v1** опущен для краткости.

---

## Конвенции

### Что такое SPA
**SPA (Single-Page Application)** — одностраничное веб-приложение: навигация происходит на клиенте без полной перезагрузки страницы; данные запрашиваются у REST API.

### Нотация параметров в путях
- **SPA-маршруты:** `:param` (стиль клиентских роутеров), напр. `/p/:token`.
- **REST API:** `{param}` (фигурные скобки), напр. `/orgs/{orgId}/objects/{objectId}`.

### Идентификаторы
- В REST API все `{id}` — UUID.
- Для коротких ссылок используются непрозрачные **токены** (base62/ULID и пр.), не равные внутренним ID.

### Время и форматы
- Временные значения — ISO-8601 в UTC, напр. `2025-08-15T10:00:00Z`.
- Ответы — JSON; ошибки — стандартизованный объект (см. ниже).

---

## SPA (Frontend) URL-паттерны

### Принцип адресации
1. Пользователь видит **короткие** адреса верхнего уровня — без вложенных идентификаторов.
2. **Контекст** (активная организация и активный объект) хранится на клиенте (`sessionStorage` вкладки).  
   Если контекст отсутствует, выполняется редирект на выбор соответствующей сущности.
3. Для «Поделиться» и закладок используются **короткие одноуровневые ссылки** на домене `go.piket.pro` (см. раздел ниже).

### Короткие (видимые) маршруты
- `/login` — вход
- `/registration` — регистрация
- `/orgs` — список организаций
- `/analytics` — аналитика **активной** организации
- `/warehouses` — склады **активной** организации
- `/objects` — список объектов **активной** организации  
  → если активная организация не выбрана — редирект на `/orgs`
- `/files` — файлы **активного** объекта
- `/works` — журнал работ **активного** объекта
- `/areas` — участки **активного** объекта
- `/acts` — акты **активного** объекта
- `/tomes` — тома **активного** объекта

> Модули **«Планы»**, **«Доступ»**, **«Просмотр»** — всплывающие окна (без отдельных маршрутов). По желанию состояние модалки можно отображать в query:  
> `?modal=plans` · `?modal=access&entity=object&id=...` · `?modal=viewer&source=acts&selection=...`.

### Редиректы при отсутствии контекста
- Страницы уровня **организации** (`/analytics`, `/warehouses`, `/objects`) без активной организации → редирект на `/orgs`.
- Страницы уровня **объекта** (`/files`, `/works`, `/areas`, `/acts`, `/tomes`) без активного объекта → редирект на `/objects` (подсказка «Выберите объект»).

---

## Короткие ссылки (Short Links)

Короткие одноуровневые URL для «Поделиться» и закладок. Не содержат иерархию; расширяются приложением до нужной страницы.

### Формат
- Базовый путь: `https://go.piket.pro/{token}`
- `{token}` — непрозрачный идентификатор с префиксом типа ресурса:
  - `o_{code}` — организация
  - `obj_{code}` — объект
  - `act_{code}` — акт
  - `tm_{code}` — том

Примеры:
- `https://go.piket.pro/o_9fL2bWQeK4`
- `https://go.piket.pro/obj_Pk7a2mDkX`
- `https://go.piket.pro/act_D0x3qZJtN`
- `https://go.piket.pro/tm_E2y7hL0pS`

### Поведение редирект-домена
- `GET https://go.piket.pro/{token}` → **302 Redirect** на SPA-маршрут `/p/{token}` на `https://app.piket.pro`.  
  Аутентификация на домене `go.*` не выполняется.

### Разрешение в SPA
- Маршрут: `/p/:token` (внутренний сервис-роут, не отображается в меню).
- Алгоритм:
  1) SPA вызывает `GET /resolve/{token}`.  
     Ответ: `{ type: "org|object|act|tome", orgId, objectId?, resourceId }`
  2) Устанавливает `activeOrgId`/`activeObjectId` в `sessionStorage` (при смене организации — сбрасывает активный объект).
  3) Переходит на соответствующий **короткий** маршрут:
     - `o_*` → `/orgs` (или `/analytics` — по продуктовой логике)
     - `obj_*` → `/objects`
     - `act_*` → `/acts` (c авто-фокусом/раскрытием акта)
     - `tm_*` → `/tomes` (c авто-фокусом/раскрытием тома)
- Если токен невалиден или нет прав — страница 404/«Нет доступа» (без раскрытия внутренних ID).

### Генерация токенов
- При создании сущности генерируется поле **`shortCode`** (напр., Base62(ULID/UUIDv4), 10–14 символов, непредсказуемый).
- Итоговый токен: `{prefix}_{shortCode}`.
- Токен **стабилен** (не меняется при переименовании/перемещении), **не совпадает** с внутренним ID.

### Безопасность и приватность
- Токены непоследовательные и непредсказуемые (исключаем перебор).
- Ошибки доступа/отсутствия ресурса возвращают **404** (не «подсказываем» существование).

---

## Технические детали SPA

### Хранение контекста
- `sessionStorage`: `activeOrgId`, `activeObjectId`; очищается при закрытии вкладки.
- Переключение организации сбрасывает активный объект.

### Роут-гварды
- Гварды на уровне роутера проверяют наличие `activeOrgId/activeObjectId` и выполняют редиректы (см. выше).

---

## REST API (Backend)

### Общие принципы
- **Контекст организации передаётся только в URL:** `/orgs/{orgId}/…`.  
  Заголовки `X-Org-Id` и query `orgId` **не используются**.
- Вложенные ресурсы адресуются через родителя: `/orgs/{orgId}/objects/{objectId}/…`.
- Версионирование — префикс `/v1`.
- CORS: разрешён origin `https://app.piket.pro`.
- **Нет** эндпоинтов экспорта печатных форм (`/export/pdf|docx` отсутствуют; печать через «Просмотр» в SPA).

### Аутентификация и сессии
- HttpOnly-cookies (`Secure`, `SameSite=Lax`), коротко живущий access-токен и refresh-токен с ротацией.
- Смена пароля/выход инвалидируют все активные сессии.

**Эндпоинты**
- `POST /auth/login` — вход (email/пароль); устанавливает cookies; возвращает `{ user }`
- `POST /auth/refresh` — обновление токенов (по refresh-cookie)
- `POST /auth/logout` — выход (инвалидация refresh, очистка cookies)
- `GET  /auth/me` — текущий пользователь

### Резолвер коротких ссылок
- `GET /resolve/{token}` → `{ type, orgId, objectId?, resourceId }`  
  Коды ошибок: `404` (нет токена/ресурса), `403` (нет доступа), `429`, `500`.

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
- `GET  /orgs/{orgId}/objects/{objectId}/works` — список записей (журнал)
- `POST /orgs/{orgId}/objects/{objectId}/works` — добавить запись
- `GET  /orgs/{orgId}/objects/{objectId}/works/{entryId}` — запись
- `PATCH /orgs/{orgId}/objects/{objectId}/works/{entryId}` — обновить
- `DELETE /orgs/{orgId}/objects/{objectId}/works/{entryId}` — удалить (мягко)

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

- **Пагинация:** `?page=1&pageSize=25` (дефолт `pageSize=25`, максимум `100`).  
  Ответ коллекции включает мета-блок:
  ```json
  { "data": [...], "meta": { "page": 1, "pageSize": 25, "total": 123 } }
  ```
- **Сортировка:** `?sort=field:asc,another:desc`
- **Фильтрация:** зарезервированы `q` (поиск), `status`, `from`, `to` и др. по ресурсу.


## Формат ошибок

Все ошибки возвращаются как JSON:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Некорректные параметры запроса",
    "details": { "field": "pageSize", "reason": "must be <= 100" },
    "requestId": "3a1c8b0d..."
  }
}
```
Коды: `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `VALIDATION_ERROR`, `RATE_LIMITED`, `INTERNAL`.

## Принципы именования

- Коллекции — множественное число: `/orgs`, `/orgs/{orgId}/objects`.
- Вложенность — через `/parent/{id}/child`.
- Идентификаторы — UUID в API; `slug` — только в SPA пермалинках.
- Статусы и бизнес-правила описаны в профильных документах (например, печать — только через «Просмотр»).

