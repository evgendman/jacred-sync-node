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

Если pin применяется для эксперимента, обязательно сохранить резервную копию /etc/hosts и использовать помеченную запись. До перезапуска JacRed подготовить одноразовый systemd unit, который при следующей загрузке выполняется до jacred.service, удаляет только запись с маркером и отключает себя. После добавления проверить getent, затем проверить фактическое продвижение checkpoint самим JacRed. Не менять checkpoint и не оставлять привязку после перезагрузки.

## Tracks

После включения Tracks проверить доступные endpoints по документации установленной версии. Upstream может предоставлять статистику, например `/stats/tracks?includeTorrentDb=true`; перед автоматизацией подтвердить путь и формат для установленной версии.

## Не делать

- Не включать системный cron парсеров при обновлении.
- Не включать `tracks` до завершения этапа 1 дорожной карты.
- Не заменять рабочий config неполным примером.
- Не коммитить `Data/`, базу, рабочий config, секреты или логи.
