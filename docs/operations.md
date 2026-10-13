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


## Временный pin для диагностики Sync API

13 октября 2026 года один и тот же запрос /sync/fdb/torrents через 104.21.73.234 не завершился за 90 секунд, а через 172.67.193.89 вернул полный ответ за 0,943 секунды. Статические VPN-маршруты Keenetic были удалены; постоянную запись IP в /etc/hosts не считать решением, поскольку Cloudflare использует общие anycast IP и адреса могут изменяться.

В текущем эксперименте pin уже установлен на `172.67.193.89`; запись помечена маркером, а одноразовый systemd unit должен удалить её при следующей загрузке до `jacred.service` и отключить себя. После перезапуска JacRed обработал пять порций примерно за секунду на порцию. Не менять checkpoint. Следом проверить, обновился ли `lastsync.txt`, и дать процессу продолжить catch-up. Не оставлять привязку после перезагрузки.

## Tracks

После включения Tracks проверить доступные endpoints по документации установленной версии. Upstream может предоставлять статистику, например `/stats/tracks?includeTorrentDb=true`; перед автоматизацией подтвердить путь и формат для установленной версии.

## Не делать

- Не включать системный cron парсеров при обновлении.
- Не включать `tracks` до завершения этапа 1 дорожной карты.
- Не заменять рабочий config неполным примером.
- Не коммитить `Data/`, базу, рабочий config, секреты или логи.
