# Синхронизация

Источник: `syncapi=https://sync.jacred.stream`. Используются FileDB Sync и отдельная синхронизация Spidr. Upstream code: [SyncCron.cs](https://github.com/jacred-fdb/jacred/blob/main/Infrastructure/Background/SyncCron.cs) и [SyncController.cs](https://github.com/jacred-fdb/jacred/blob/main/Controllers/SyncController.cs).

## Зафиксированный старт

12 октября 2026 года журнал показывал `lastsync=-1`, затем партии `[1]` и `[2]` по 2001 запись, обе с `nextread=True`. Это подтверждает получение данных, но не окончание первоначальной синхронизации.

Наблюдать:

```bash
sudo journalctl -u jacred -f --no-pager | grep --line-buffered -Ei 'sync:|syncspidr|error|exception'
```

Ctrl+C завершает только просмотр журнала. Ищите `sync: end`, отсутствие повторяющихся ошибок и последующий успешный инкрементальный цикл. После рестарта проверьте, что состояние и checkpoint сохранены.

## User-Agent и HTTP 403

Во время диагностики Cloudflare-защищённый `sync.jacred.stream` отвечал по-разному для запросов с различными заголовками. Браузерный User-Agent давал HTTP 200; upstream JacRed также задаёт User-Agent. Не делайте вывод о VPN-маршрутизации по одному ответу — воспроизводите запрос с фактическими заголовками клиента.

## Snapshot

`https://sync.jacred.stream/latest.tar.zst.zip` возвращал HTTP 404 на 12 октября 2026 года, но Sync API работал. При новой установке это нужно проверить заново.
