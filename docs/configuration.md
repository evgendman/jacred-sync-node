# Конфигурация

Рабочий файл: `/opt/jacred/init.conf`. Не заменять его коротким примером: остальные параметры установленной версии тоже могут быть важны.

| Параметр | Текущее значение | Назначение |
|---|---|---|
| `syncapi` | `https://sync.jacred.stream` | Родительский источник |
| `opensync` | `false` | Не предоставлять наш Sync API другим узлам |
| `syncsport` | `true` | Разрешить синхронизацию спортивных записей |
| `syncspidr` | `true` | Включить синхронизацию Spidr |
| `tracks` | `false` временно | Отложить локальный анализ до конца первичной синхронизации |
| `trackslog` | `true` | Логирование Tracks |
| `trackscategory` | `jacred-mini-9117` | Категория этого узла |
| `tsuri` | `http://127.0.0.1:8097` | TorrServer для Tracks |

`syncapi` и `opensync` задают разные направления: первое — откуда получать данные, второе — отдаёт ли наш экземпляр Sync API другим узлам.

Проверить только выбранные поля, не печатая весь файл:

```bash
sudo jq '{opensync, syncapi, syncsport, syncspidr, tracks, trackslog, trackscategory, tsuri, timeSync, timeSyncSpidr}' /opt/jacred/init.conf
```

Перед изменением создать резервную копию, изменить только необходимые поля, проверить JSON через `sudo jq empty /opt/jacred/init.conf`, затем перезапустить службу. Не добавлять рабочий файл в Git.
