        # redis — Гибридный режим и введение в репликацию

        Homework-шаблон для урока **l3_hybrid_and_replication** (Гибридный режим и введение в репликацию) на платформе Vibe Learn.

        ## Что делать

        Напиши Go-программу, демонстрирующую replication lag и WAIT-семантику.

**Окружение:** docker-compose с master + 2 replicas (async-репликация, гибридный persistence).

**Шаги:**
1. Запиши 1000 `INCR` на master, параллельно семпли `INFO replication` и считай
   lag в байтах (`master_repl_offset` − `slave_repl_offset`) и в секундах для каждой реплики.
2. Сразу после записи читай counter с обеих реплик и зафиксируй случаи
   read-after-write inconsistency (реплика вернула старое значение).
3. Повтори запись с `WAIT 1 100` после каждой и покажи, что доля inconsistent-чтений
   падает (но latency записи растёт).
4. Отчёт: lag-распределение, число inconsistent-чтений без/с WAIT, p99 write latency.

**CI-проверки в template repo:**
- `assert max_lag_ms < 100` — под нормальной нагрузкой реплики не отстают сильно.
- `assert inconsistent_with_wait <= inconsistent_without_wait` — WAIT уменьшает рассинхрон.
- `assert write_p99_with_wait >= write_p99_without_wait` — WAIT стоит latency.

## Контекст (из transfer-задачи урока)

На проде: 1 мастер Redis + 2 реплики. Async-репликация. Read-нагрузка 50k RPS — разделена
между всеми тремя через client-side load balancer. Внезапно: пользователи жалуются «мой
профиль обновлённый не виден сразу — то старое, то новое». Опиши причину и три варианта
решения.

## Recap из урока

- Гибридный режим (RDB + AOF preamble) — default с Redis 5. Загрузка как RDB, durability как AOF.
- Default репликация **async**: мастер не ждёт реплику. Реплика может отстать на десятки мс — секунды под нагрузкой.
- Первое подключение реплики = full sync: BGSAVE → RDB по сети → replay входящих. После — incremental streaming.
- WAIT N TIMEOUT даёт «команда видна на N репликах», но НЕ durability — реплика могла не fsync'нуть.
- Read-only реплики снимают 50-80% read-нагрузки с мастера, но дают **read-after-write inconsistency**.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node Redis 7 на `localhost:6379` (с включёнными keyspace-notifications и AOF). Адрес переопределяется через env `REDIS_ADDR`.

        ## Запуск

        ```bash
        # Поднять локальный Redis
        docker compose up -d

        # Прогнать тесты (интеграционный включается через REDIS_INTEGRATION=1)
        go test ./...
        REDIS_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
