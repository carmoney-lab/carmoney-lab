# kilo_hello

готов

1) По README.md и корневым файлам это учебный сервис предварительной оценки заявки на заём под ПТС: принимает заявку (VIN, год, пробег, оценочная стоимость, сумма, срок), считает LTV и возвращает решение `approve` / `review` / `reject`.
2) В Makefile: `make up` (поднять сервис и базу), `make down`, `make ps`, `make logs`, `make install`, `make test` (PHPUnit), `make lint` (php -l), `make seed`, `make help`; в docker-compose.yml команд запуска/проверки не нашёл — только сервисы backend (PHP, порт `${APP_PORT:-8080}:8080`) и db (mysql:8.0, порт `${DB_PORT:-3307}:3306`, healthcheck).
3) Решение approve / review / reject считается в `backend/src/Domain/` — `DecisionEngine.php` (пороги берёт из `backend/config/rules.php`), связку LTV + решение собирает `AssessmentService.php`.

модель: training-2026-09-glm-5.3
