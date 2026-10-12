# Эксплуатация и диагностика

## Состояние и health

```bash
systemctl is-active jacred
sudo systemctl status jacred --no-pager
curl -i http://127.0.0.1:9117/health
```

HTTP 200 подтверждает доступность health endpoint, но не завершение синхронизации.

## Журналы

```bash
sudo journalctl -u jacred -n 100 --no-pager
sudo journalctl -u jacred -f --no-pager | grep --line-buffered -Ei 'sync:|syncspidr|tracks|error|exception'
```

Ctrl+C останавливает просмотр, не службу.

## Безопасная проверка конфигурации

```bash
sudo jq '{opensync, syncapi, syncsport, syncspidr, tracks, trackslog, trackscategory, tsuri, timeSync, timeSyncSpidr}' /opt/jacred/init.conf
```

## Перед изменением

```bash
sudo cp -a /opt/jacred/init.conf "/opt/jacred/init.conf.bak-$(date +%Y%m%d-%H%M%S)"
sudo jq empty /opt/jacred/init.conf
sudo systemctl restart jacred
sudo systemctl is-active jacred
sudo journalctl -u jacred -n 80 --no-pager
```

Сначала создайте копию, затем редактируйте нужные поля и проверяйте JSON до рестарта. Не редактировать вручную checkpoint синхронизации без специальной причины.

## Tracks

После включения Tracks проверить доступные endpoints по документации установленной версии. Upstream может предоставлять статистику, например `/stats/tracks?includeTorrentDb=true`; перед автоматизацией подтвердить путь и формат для установленной версии.

## Не делать

- Не включать системный cron парсеров при обновлении.
- Не включать `tracks` до завершения этапа 1 дорожной карты.
- Не заменять рабочий config неполным примером.
- Не коммитить `Data/`, базу, рабочий config, секреты или логи.
