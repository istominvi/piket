# API и URL‑паттерны (черновик)

## Базовые маршруты приложения (SPA)
- `/organizations` — список организаций
- `/organizations/:orgId` — активная организация (редирект на последний открытый раздел)
- `/objects` — список объектов активной организации
- `/objects/:objectId/files` — файлы объекта
- `/plans` — тарифы активной организации

## REST API (пример)
- `POST /api/auth/login`
- `POST /api/organizations` / `GET /api/organizations`
- `PATCH /api/organizations/:id`
- `GET /api/organizations/:id/objects`
- `GET /api/objects/:id/files` / `POST /api/objects/:id/files`
- …
